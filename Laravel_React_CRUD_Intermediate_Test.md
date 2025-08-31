# CRUD Intermediate Test – Laravel & React

> **Tujuan:** Menguji kemampuan kandidat membangun _CRUD app_ tingkat menengah end‑to‑end (backend Laravel + frontend React), meliputi desain skema, relasi, API RESTful, autentikasi, otorisasi berbasis peran, validasi, upload file, pencarian & filter, pagination, serta UX dasar di React.
>
> **Durasi rekomendasi:** 3–4 jam (maks 1 hari).  
> **Level:** Intermediate (bukan boilerplate semata).

---

## 1) Studi Kasus Singkat: **Inventory & Orders**

Bangun aplikasi manajemen **Produk** dan **Pesanan** dengan entitas berikut:

- **Product**
  - `id`, `name`, `sku` (unik), `price` (decimal), `stock` (integer), `status` (`active|inactive`), `image_path` (nullable), `description` (text, nullable), `created_at`, `updated_at`, `deleted_at` (soft delete)
- **Category**
  - `id`, `name` (unik), `created_at`, `updated_at`
- **product_category** (pivot many‑to‑many)
  - `product_id`, `category_id`
- **Order**
  - `id`, `order_number` (unik, format: `ORD-YYYYMMDD-####`), `customer_name`, `customer_email`, `total_amount` (decimal), `status` (`pending|paid|cancelled`), `created_at`, `updated_at`
- **OrderItem**
  - `id`, `order_id`, `product_id`, `qty` (integer, >0), `price` (decimal, harga saat transaksi, bukan relasi), `subtotal` (computed = `qty * price`)

**Aturan bisnis ringkas:**
- `Order.total_amount` adalah **penjumlahan** `OrderItem.subtotal`. Tidak boleh diedit manual via API.
- Saat membuat Order, `stock` Product **berkurang** sesuai `qty`. Saat status Order diubah ke `cancelled`, stok **dikembalikan**.
- Tidak boleh menambah `OrderItem` pada Order yang sudah `paid` atau `cancelled`.
- Produk `inactive` **tidak bisa** ditambahkan ke Order.

---

## 2) Backend (Laravel) – Requirements

1. **Versi & Setup**
   - Laravel 10/11, PHP 8.1+.
   - **Database:** gunakan **PostgreSQL 14+** sebagai DBMS utama.
   - Gunakan **migrations**, **seeders** (contoh 10 produk, 3 kategori, 3 user role).  
   - Aktifkan **soft deletes** untuk Product.

   **Contoh `.env` PostgreSQL**
   ```env
   DB_CONNECTION=pgsql
   DB_HOST=127.0.0.1
   DB_PORT=5432
   DB_DATABASE=crud_test
   DB_USERNAME=postgres
   DB_PASSWORD=secret
   ```

   **Contoh `docker-compose.yml` (opsional)**
   ```yaml
   services:
     postgres:
       image: postgres:14
       environment:
         POSTGRES_DB: crud_test
         POSTGRES_USER: postgres
         POSTGRES_PASSWORD: secret
       ports: ["5432:5432"]
       volumes: ["pg_data:/var/lib/postgresql/data"]
   volumes:
     pg_data:
   ```

2. **Autentikasi & Otorisasi**
   - Implement **JWT** atau **Laravel Sanctum** (pilih salah satu).  
   - Role minimal: `admin`, `staff`, `viewer`.
     - `admin`: CRUD semua resource.
     - `staff`: CRUD Product & Order, **tanpa** hard delete (boleh soft delete).
     - `viewer`: read‑only semua endpoint.
   - Gunakan **Policies/Gates** untuk pembatasan aksi.

3. **API Endpoints (RESTful)**
   - `POST /auth/login` → login, return access token (+ refresh token jika pakai JWT).
   - `GET /me` → info user & role.
   - `api/products`
     - `GET /api/products` (list + query: `q`, `status`, `category_id[]`, `min_price`, `max_price`, `sort`, `page`, `per_page`)
     - `POST /api/products` (create)
     - `GET /api/products/{id}` (detail)
     - `PUT /api/products/{id}` (update)
     - `DELETE /api/products/{id}` (soft delete)
     - `POST /api/products/{id}/image` (upload gambar; simpan ke storage, simpan path)
     - **Validasi**: unique `sku`, price ≥ 0, stock ≥ 0; `image` max 2MB, `jpeg|png`.
   - `api/categories` (CRUD sederhana + unique name)
   - `api/orders`
     - `GET /api/orders` (list + filter: `status`, rentang tanggal `from`–`to`, `q` untuk `order_number`/`customer_name`, `page`)
     - `POST /api/orders` (create header + item sekaligus; hitung total_amount server‑side; kurangi stok)
     - `GET /api/orders/{id}` (detail termasuk items)
     - `PUT /api/orders/{id}` (update header **kecuali** `total_amount`)
     - `PATCH /api/orders/{id}/status` (ubah status → jika `cancelled` kembalikan stok)
     - **Validasi**: Tidak boleh qty > stock; produk inactive ditolak.
   - Gunakan **API Resource** (transformer) untuk response yang konsisten.

4. **Query, Pagination, & Caching**
   - Pagination standar (`page`, `per_page`).
   - Pencarian `q` untuk name/sku (Products) dan order_number/customer_name (Orders).
   - Tambahkan **query scopes** di Eloquent (mis. `scopeFilter`, `scopeSearch`).
   - **(Bonus)** Cache list Products (mis. 60 detik) yang otomatis _bust_ saat create/update/delete.

5. **Arsitektur & Kualitas Kode**
   - Gunakan **Form Request** untuk validasi.
   - Gunakan **Service layer** untuk logika Order (perhitungan & stok) agar controller tetap tipis.
   - Tambahkan **tests** minimal:
     - Feature: autentikasi, create Product, create Order (stok berkurang), cancel Order (stok kembali).
     - Unit: service perhitungan `subtotal` & `total_amount`.

6. **Dokumentasi API**
   - Tambahkan **OpenAPI/Swagger** **atau** `api-docs` Markdown sederhana dengan contoh request/response, termasuk error (422, 401, 403, 404).

---

## 3) Frontend (React) – Requirements

1. **Stack**
   - React (Vite atau CRA), React Router.
   - Data fetching: **React Query** atau **SWR**.
   - HTTP: Axios/fetch.
   - Form: React Hook Form (+ Yup/Zod) untuk validasi.
   - State global: optional (Context/Zustand) seperlunya.

2. **Fitur Minimal**
   - **Auth Flow:** halaman Login; simpan token aman; _auto-logout_ saat 401; protected routes.
   - **Products:** list + search + filter (status, category, price range), pagination, sort; create/edit; upload gambar; soft delete; detail view.
   - **Categories:** list + create/edit/delete sederhana.
   - **Orders:** list + filter status & tanggal; create Order (pilih Product, set qty, validasi stok, tampilkan subtotal & total otomatis); detail Order beserta items; ubah status (pending→paid/cancelled). 
   - **UX:** loading state, error state, konfirmasi delete, toast notifikasi.
   - **(Bonus)**
     - Optimistic update untuk create/update Product.
     - Reusable Table component (kolom dinamis, sort).
     - Export CSV pada list Products.

3. **Standar UI/UX**
   - Responsif.
   - Handling error 422 (tampilkan pesan per field), 401/403 (redirect/notice), 500 (toast umum).
   - Format harga (IDR) di UI.

4. **Struktur Direktori (saran)**
   ```text
   src/
     api/ (axios client, hooks query)
     components/ (Table, Form, Input, Select, Modal, Toast)
     pages/
       Login/
       Products/ (List, Form, Detail)
       Categories/
       Orders/ (List, Form, Detail)
     utils/ (formatCurrency, guards, constants)
     router/ (ProtectedRoute, routes)
   ```

---

## 4) Acceptance Criteria (Checklist)

### Backend
- [ ] Semua migration & seeder berjalan tanpa error.
- [ ] Auth berfungsi; role membatasi akses sesuai definisi.
- [ ] Endpoint CRUD Product, Category, Order lengkap & _validated_.
- [ ] Upload gambar Product berjalan; file tersimpan & path direturkan di response.
- [ ] Pencarian, filter, sort, dan pagination bekerja.
- [ ] Logika stok benar saat create/cancel Order.
- [ ] Dokumentasi API tersedia & akurat.
- [ ] Test minimal lulus.

### Frontend
- [ ] Login + protected routes berjalan.
- [ ] Products list: search, filter, sort, pagination.
- [ ] Form Product: validasi, upload gambar, error handling 422.
- [ ] Orders: create dengan kalkulasi subtotal/total otomatis; validasi stok; ubah status.
- [ ] UX: loading/error states, konfirmasi destructive action, toast.
- [ ] Bonus (opsional) dicatat jelas jika dikerjakan.

---

## 5) Penilaian (Rubrik, total 100)

- **Kebenaran Fungsional (35):** Endpoint & UI sesuai spesifikasi, stok & total akurat.
- **Kualitas Kode (20):** Struktur rapi, separation of concerns (Controller–Service–Model), naming jelas.
- **Validasi & Keamanan (10):** Form Request, policy/guard, penanganan token, aturan role.
- **Query & Performa (10):** Pagination, filter, search, (bonus) caching.
- **DX/UX (15):** Error/empty/loading states, konsistensi UI, aksesibilitas dasar.
- **Testing (5):** Unit/feature minimal.
- **Dokumentasi (5):** Cara run, .env contoh, endpoint & contoh request/response.

> **Bonus poin (maks +10):** Docker compose (app + db + pgsql/mysql), CI minimal (lint/test), optimistic UI, CSV export, OpenAPI UI, atau demo online.

---

## 6) Data Seeder (Contoh Minimal)

- 3 user:
  - `admin@example.com` / password: `password` (role: admin)
  - `staff@example.com` / password: `password` (role: staff)
  - `viewer@example.com` / password: `password` (role: viewer)
- 3 kategori: `Elektronik`, `Fashion`, `Grocery`
- 10 produk acak dengan kombinasi kategori; beberapa `inactive`.

---

## 7) Contoh Kontrak API (Ringkas)

### Auth
```http
POST /auth/login
Content-Type: application/json
{ "email": "admin@example.com", "password": "password" }
→ 200 { "token": "...", "token_type": "Bearer", "user": { "id": 1, "role": "admin" } }
→ 401 { "message": "Invalid credentials" }
```

### Products (List dengan filter)
```http
GET /api/products?q=keyboard&status=active&min_price=100000&max_price=500000&category_id[]=1&sort=price:desc&page=1&per_page=10
→ 200 { "data": [ ... ], "meta": { "page": 1, "per_page": 10, "total": 57 } }
```

### Create Order
```http
POST /api/orders
Content-Type: application/json
{
  "customer_name": "Rizal",
  "customer_email": "rizal@example.com",
  "items": [
    { "product_id": 3, "qty": 2 },
    { "product_id": 5, "qty": 1 }
  ]
}
→ 201 {
  "id": 10,
  "order_number": "ORD-20250831-0001",
  "status": "pending",
  "total_amount": 350000,
  "items": [
    { "product_id": 3, "qty": 2, "price": 100000, "subtotal": 200000 },
    { "product_id": 5, "qty": 1, "price": 150000, "subtotal": 150000 }
  ]
}
→ 422 { "errors": { "items.0.qty": ["Quantity exceeds stock"] } }
```

---

## 8) Instruksi Pengumpulan

- Sertakan **README** berisi:
  - Cara menjalankan backend & frontend (termasuk contoh `.env`).
  - Kredensial akun seeder.
  - Catatan keputusan teknis (mis. kenapa pilih Sanctum vs JWT).
  - Daftar fitur bonus (jika ada).
- Kirimkan sebagai repo (monorepo atau dua repo) **atau** arsip `.zip`.
- Wajib sertakan **screenshot** UI utama (Products List & Create Order).

---

## 9) Kriteria Diskualifikasi Cepat

- Password hard‑coded di repo (**kecuali seeder demo**).
- Token disimpan di `localStorage` tanpa pertimbangan keamanan (lebih baik HTTP‑only cookie bila feasible; atau jelaskan mitigasinya).
- Manipulasi `total_amount` dari client.
- Tidak ada validasi dasar & penanganan error.

---

## 10) Tambahan (Opsional)

- Endpoint **Export CSV**: `GET /api/products/export.csv` (filter mengikuti query list).
- **Rate limiting**: 60 req/menit per user.
- **E2E Test** (Cypress/Playwright) minimal untuk login + create Product.

Selamat mengerjakan! Semoga sukses. 🚀
