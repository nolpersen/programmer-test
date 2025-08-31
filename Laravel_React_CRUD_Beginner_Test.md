# CRUD Beginner Test – Laravel & React (PostgreSQL)

> **Tujuan:** Menguji dasar-dasar pembuatan aplikasi CRUD sederhana end-to-end (Laravel backend + React frontend). Fokus pada pemahaman konsep dasar: routing, controller, model, migration, validasi, CRUD API, dan konsumsi API di React.
>
> **Durasi rekomendasi:** 2–3 jam.  
> **Level:** Beginner.

---

## 1) Studi Kasus: **Book Tracker**

Bangun aplikasi sederhana untuk mencatat koleksi **Buku** dengan relasi sederhana ke **Kategori**.

### Entitas
- **Category**
  - `id`, `name` (unik), `created_at`, `updated_at`
- **Book**
  - `id`, `title`, `author`, `published_year` (integer), `category_id` (FK), `price` (decimal, >=0), `created_at`, `updated_at`

**Aturan sederhana:**
- `name` kategori harus unik.
- `title` dan `author` wajib diisi.
- `published_year` antara 1900–tahun sekarang.
- `price` tidak boleh negatif.
- Saat kategori dihapus, buku dengan kategori tersebut tidak boleh ikut terhapus (gunakan **restrict** atau validasi di controller/service).

---

## 2) Backend (Laravel) – Requirements

1. **Versi & Setup**
   - Laravel 10/11, PHP 8.1+.
   - **Database:** gunakan **PostgreSQL 14+**.
   - Gunakan **migrations** dan **seeders** (3 kategori contoh, 5–10 buku).

   **Contoh `.env` (PostgreSQL)**
   ```env
   DB_CONNECTION=pgsql
   DB_HOST=127.0.0.1
   DB_PORT=5432
   DB_DATABASE=crud_beginner
   DB_USERNAME=postgres
   DB_PASSWORD=secret
   ```

   **Contoh `docker-compose.yml` (opsional)**
   ```yaml
   services:
     postgres:
       image: postgres:14
       environment:
         POSTGRES_DB: crud_beginner
         POSTGRES_USER: postgres
         POSTGRES_PASSWORD: secret
       ports: ["5432:5432"]
       volumes: ["pg_data:/var/lib/postgresql/data"]
   volumes:
     pg_data:
   ```

2. **API Endpoints (RESTful minimal)**
   - `GET /api/categories` → list kategori
   - `POST /api/categories` → buat kategori (validasi unik `name`)
   - `PUT /api/categories/{id}` → update
   - `DELETE /api/categories/{id}` → hapus (tolak jika masih dipakai `books`)

   - `GET /api/books` → list buku + query opsional: `q` (cari di `title`/`author`), `category_id`
   - `POST /api/books` → buat buku
   - `GET /api/books/{id}` → detail buku
   - `PUT /api/books/{id}` → update buku
   - `DELETE /api/books/{id}` → hapus buku

   **Catatan teknis:**
   - Gunakan **Route::apiResource** untuk Book & Category.
   - Validasi menggunakan **Form Request** (mis. `StoreBookRequest`, `UpdateBookRequest`).
   - Gunakan **Eloquent Relationship** (`Book` belongsTo `Category`, `Category` hasMany `Book`).
   - Pencarian `q` boleh menggunakan `ILIKE` (PostgreSQL) untuk case-insensitive.

3. **Kualitas Kode**
   - Controller ringkas, gunakan Model + Form Request.
   - Response JSON konsisten (mis. `{ data: ..., message: ... }`).
   - Tangani error 422 (validasi) dan 404 (data tidak ditemukan).

4. **Dokumentasi Singkat**
   - Di README tuliskan cara run, sample `.env`, dan contoh request/response pendek untuk create/update Book.

---

## 3) Frontend (React) – Requirements

1. **Stack Sederhana**
   - React (Vite), React Router.
   - Data fetching: fetch/Axios (boleh langsung, tidak perlu React Query).
   - Form: bebas (React Hook Form **disarankan**, tapi tidak wajib).

2. **Fitur Minimal**
   - **Books Page**
     - Tabel daftar buku (kolom: Title, Author, Category, Year, Price).
     - Input pencarian (`q`) untuk `title/author`.
     - Filter kategori (dropdown).
     - Tombol **Create**, **Edit**, **Delete**.
   - **Book Form Page**
     - Field: `title`, `author`, `published_year`, `category_id`, `price`.
     - Validasi dasar di sisi client (wajib isi, numeric untuk year & price).
   - **Categories Page (sederhana)**
     - List + Create/Edit/Delete kategori.
   - Tampilkan loading state & error sederhana (alert/toast).

3. **UX Dasar**
   - Navigasi jelas: Home → Books (default), Categories.
   - Setelah create/update, kembali ke list & tampilkan notifikasi singkat.
   - Hindari page reload penuh; gunakan SPA flow.

4. **Struktur Direktori (saran)**
   ```text
   src/
     api/ (axios client)
     pages/
       Books/ (List.jsx, Form.jsx)
       Categories/ (List.jsx, Form.jsx)
     components/ (Input, Select, Button, Table, Alert)
     router/ (routes.jsx)
     utils/ (formatCurrency.js)
   ```

---

## 4) Acceptance Criteria (Checklist)

### Backend
- [ ] Migration & Seeder berjalan.
- [ ] Endpoint CRUD untuk Book & Category berfungsi dan tervalidasi.
- [ ] Pencarian `q` di `/api/books` bekerja.
- [ ] Proteksi hapus kategori yang masih dipakai (return 422 dengan pesan yang jelas).
- [ ] Response JSON konsisten & error ditangani (422/404).

### Frontend
- [ ] Daftar buku tampil dari API.
- [ ] Pencarian dan filter kategori di daftar buku bekerja.
- [ ] Form create/edit buku dengan validasi dasar.
- [ ] CRUD kategori sederhana.
- [ ] Menampilkan loading & error states.

---

## 5) Penilaian (Rubrik, total 100)

- **Fungsional Dasar (40):** CRUD berjalan sesuai spesifikasi.
- **Kualitas Kode (20):** Struktur rapi, komponen terpisah, naming jelas.
- **Validasi & Error Handling (15):** Form Request & client validation, pesan error informatif.
- **UX Sederhana (15):** Navigasi, notifikasi, loading/error states yang wajar.
- **Dokumentasi (10):** README jelas, cara run & contoh request/response.

> **Bonus (opsional, +10):** Pagination sederhana di `/api/books`, format mata uang IDR di UI, dan deploy lokal dengan Docker Compose.

---

## 6) Seeder (Contoh Data)

- Kategori: `Fiksi`, `Teknologi`, `Bisnis`
- Buku contoh 5–10 data (campuran kategori), contoh:
  - "Clean Code" – Robert C. Martin – 2008 – Teknologi – 250000
  - "The Pragmatic Programmer" – Andrew Hunt – 1999 – Teknologi – 220000
  - "Laskar Pelangi" – Andrea Hirata – 2005 – Fiksi – 75000

---

## 7) Contoh Kontrak API (Ringkas)

### List Books dengan Pencarian & Filter
```http
GET /api/books?q=clean&category_id=2
→ 200 {
  "data": [
    { "id": 1, "title": "Clean Code", "author": "Robert C. Martin", "published_year": 2008, "price": 250000, "category": { "id": 2, "name": "Teknologi" } }
  ]
}
```

### Create Book
```http
POST /api/books
Content-Type: application/json
{
  "title": "Atomic Habits",
  "author": "James Clear",
  "published_year": 2018,
  "category_id": 3,
  "price": 120000
}
→ 201 { "data": { "id": 11, "title": "Atomic Habits", ... }, "message": "Book created" }
→ 422 { "errors": { "title": ["The title field is required."] } }
```

---

## 8) Instruksi Pengumpulan

- Kirim sebagai repo atau `.zip` berisi backend & frontend (boleh terpisah).
- Sertakan **README** dengan:
  - Cara menjalankan (include contoh `.env` PostgreSQL).
  - Endpoints ringkas & contoh request/response.
  - Catatan keputusan kecil (mis. memakai React Hook Form atau tidak).
- Sertakan 1–2 **screenshot** UI (List Books, Form Book).

Selamat mengerjakan dan semoga lancar! 🚀
