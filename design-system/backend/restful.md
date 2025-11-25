# **RESTful API**
##### Write by: Adrian Milano

Representational State Transfer (REST) adalah gaya arsitektur yang memanfaatkan protokol HTTP secara maksimal untuk komunikasi tanpa status (stateless) antar layanan.

**1. Proper Routing (Penentuan Jalur yang Tepat)**
- Resource-Based: URL harus berfokus pada sumber daya (resource), bukan tindakan (action). Gunakan kata benda jamak (plural nouns) untuk koleksi sumber daya.
- Protokol HTTP: Manfaatkan metode HTTP untuk menyatakan tindakan (verb):

| Metode HTTP   | Tindakan (Implikasi)                              | Contoh URL             |
|---------------|--------------------------------------------------|------------------------|
| **GET**       | Mengambil data (Read)                            | /api/v1/users/123      |
| **POST**      | Membuat sumber daya baru (Create)               | /api/v1/orders         |
| **PUT/PATCH** | Mengganti/Memperbarui seluruh/sebagian sumber daya (Update) | /api/v1/users/123      |
| **DELETE**    | Menghapus sumber daya (Delete)                  | /api/v1/orders/456     |

**2. Pagination, Filtering, & Sorting (Manajemen Data)**

Saat data koleksi (list) menjadi besar, kita harus menyediakan cara untuk membatasi, menyaring, dan mengurutkan hasilnya di sisi klien, bukan mengambil semua data dari database.

| Fitur       | Contoh Parameter                        | Keterangan                                                                 |
|------------|----------------------------------------|---------------------------------------------------------------------------|
| **Pagination** | ?page=2&limit=50 atau Cursor-based (?after=base64_cursor) | Membatasi jumlah item dan menentukan halaman. Cursor-based lebih efisien untuk scrolling tanpa batas. |
| **Filtering**  | ?status=pending&user_id=123           | Menyaring hasil berdasarkan kriteria tertentu.                             |
| **Sorting**    | ?sort=price,-created_at               | Mengurutkan hasil. Tanda minus (-) biasanya berarti urutan menurun (descending). |


**3. Versioning (Kontrol Versi)**

Penting untuk mengelola perubahan API yang tidak kompatibel (breaking changes) agar klien lama tetap berfungsi. Ada tiga metode utama:

| Metode              | Contoh                  | Keterangan                                                         |
|--------------------|------------------------|-------------------------------------------------------------------|
| **URL Path (Paling Umum)** | /api/**v1**/users       | Paling eksplisit, mudah di-cache, dan direkomendasikan.           |
| **Query Parameter** | /api/users?version=2    | Kurang disukai, tetapi mudah diimplementasikan.                   |
| **Custom Header**   | Accept-version: v3      | Paling bersih, tetapi klien perlu manajemen header yang lebih kompleks. |

**4. Error Response Standard (RFC 7807)**

Daripada hanya mengembalikan error 500 dengan string, API modern harus menggunakan respons error yang terstruktur (berbasis JSON) sesuai standar RFC 7807 (Problem Details for HTTP APIs).

Ini memberikan klien informasi yang kaya untuk menangani masalah secara terprogram.

```json
{
  "type": "https://example.com/probs/out-of-credit",
  "title": "You do not have enough credit.",
  "status": 403,
  "detail": "Your current balance is 30, but the price is 50.",
  "instance": "/account/12345/payments/2024" 
}
```
| Field     | Kegunaan                                                       |
|-----------|----------------------------------------------------------------|
| **status**   | Kode status HTTP (400, 403, 500).                             |
| **title**    | Deskripsi singkat, umum, dan mudah dibaca manusia.            |
| **detail**   | Penjelasan spesifik tentang masalah.                          |
| **type**     | URI yang mengidentifikasi jenis masalah (dapat diklik).       |
| **instance** | URI yang mengidentifikasi kejadian masalah tertentu.          |

**5. OpenAPI / Swagger (Dokumentasi)**

OpenAPI Specification (OAS), yang sebelumnya dikenal sebagai Swagger, adalah standar untuk mendefinisikan API REST secara machine-readable (dapat dibaca mesin).
- Peran: Bertindak sebagai Service Contract (yang kita bahas sebelumnya). Kontrak ini memungkinkan klien, server stubs, dan dokumentasi dihasilkan secara otomatis.
- Mendefinisikan kontrak sebelum menulis kode (Design First). Ini sangat penting untuk tim yang berbeda yang mengembangkan producer dan consumer secara independen.

**6. Authentication & Authorization**

A. JWT (JSON Web Token)
- Apa itu: Standar terbuka untuk membuat token akses yang compact (ringkas) dan self-contained (mandiri).
- Cara Kerja: Token ditandatangani secara kriptografis (misalnya, menggunakan HMAC SHA256) oleh Auth Service. Token ini berisi claims (informasi user dan peran) dan dikirim pada setiap request melalui Header `Authorization: Bearer <token>`.
- Keunggulan: API Gateway dapat memvalidasi token tanpa perlu menghubungi Auth Service pada setiap request (stateless dan cepat).

B. OAuth 2.0 (Delegasi Akses)
- Apa itu: Protokol Authorization (izin) yang memungkinkan user memberikan akses ke sumber daya mereka kepada aplikasi pihak ketiga tanpa membagikan kata sandi.
- Peran: OAuth 2.0 adalah framework yang mengatur bagaimana token (seperti JWT) diterbitkan dan digunakan.
- Flows Umum: Authorization Code Grant (untuk aplikasi web/mobile), Client Credentials (untuk komunikasi service-to-service).
