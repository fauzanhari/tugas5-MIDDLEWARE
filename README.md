# Tugas 5: Middleware dan Penanganan Error API

## Profil Pengerjaan

| Keterangan   | Isi                     |
| ------------ | ----------------------- |
| Nama         | **FAUZAN HARI RAMDANI** |
| NIM          | **F1D02410047**         |
| Varian tugas | **A: Perpustakaan**     |
| Resource API | `/buku`                 |

## Gambaran Singkat

Aplikasi ini merupakan REST API sederhana untuk mengelola koleksi buku. Fitur utamanya meliputi melihat daftar buku, mencari buku berdasarkan kategori atau ID, menambahkan buku, menghapus buku, dan mengubah data buku sebagai fitur tambahan.

Proyek dibuat menggunakan Node.js dan Express. Joi dipakai untuk memeriksa data yang dikirim, sedangkan middleware digunakan untuk mencatat aktivitas request, memeriksa API Key, dan menangani error secara terpusat.

Data buku pada proyek ini disimpan di dalam array, belum memakai database. Artinya, data yang ditambahkan atau diubah akan kembali ke data awal ketika server dijalankan ulang.

## Menyiapkan dan Menjalankan Proyek

Sebelum menjalankan aplikasi, pastikan Node.js dan npm sudah tersedia di komputer.

Buka terminal pada folder proyek, kemudian jalankan perintah berikut:

```bash
npm install
```

Buat file `.env` di folder utama proyek dan isi konfigurasi berikut:

```env
API_KEY=perpus_unram_keren
PORT=3000
```

Setelah itu, nyalakan server:

```bash
npm run dev
```

Jika tidak ada masalah, terminal akan menampilkan informasi bahwa server berjalan di:

```text
http://localhost:3000
```

Request API bisa dicoba melalui file `requests.http` dengan ekstensi REST Client di VS Code. Untuk request POST, PUT, dan DELETE, tambahkan header `x-api-key` dengan nilai `perpus_unram_keren`.

**Catatan API Key:** aplikasi membaca kunci akses dari `process.env.API_KEY`. Jika variabel tersebut belum diatur atau nilainya tidak cocok, request POST, PUT, dan DELETE akan ditolak dengan status 401. Request GET tetap bisa digunakan. File `.env` tidak ikut diunggah ke GitHub karena sudah dimasukkan ke `.gitignore`; pengguna lain bisa membuat file tersebut dengan mengikuti `.env.example`.

## Bagian-Bagian di Dalam Proyek

File proyek dibagi berdasarkan tugas masing-masing supaya alur program lebih mudah diikuti.

```text
T5-MIDDLEWARE-ERROR-NIM9/
├── controllers/
│   └── bukuController.js
├── data/
│   └── buku.js
├── middlewares/
│   ├── cekApiKey.js
│   ├── errorHandler.js
│   ├── logger.js
│   └── notFound.js
├── routes/
│   └── bukuRoutes.js
├── services/
│   └── bukuService.js
├── validators/
│   └── bukuValidator.js
├── utils/
│   └── AppError.js
├── screenshot/
├── .env.example
├── .gitignore
├── index.js
├── package.json
├── README.md
└── requests.http
```

Penjelasan singkatnya:

- `index.js` menjalankan aplikasi dan memasang router utama.
- `routes/` berisi daftar alamat API serta urutan middleware yang digunakan.
- `controllers/` mengatur request yang masuk dan response yang dikirim.
- `services/` menjalankan proses utama, termasuk aturan penambahan, pengubahan, dan penghapusan buku.
- `data/` menyimpan data awal buku.
- `middlewares/` berisi logger, pemeriksaan API Key, penanganan URL yang tidak tersedia, dan error handler.
- `validators/` menyimpan aturan pemeriksaan data menggunakan Joi.
- `utils/` berisi `AppError` untuk membantu membuat error aplikasi.
- `requests.http` berisi kumpulan request untuk mencoba endpoint.
- `screenshot/` menjadi tempat menyimpan bukti hasil pengujian.

Router dipasang melalui `app.use('/buku', bukuRoutes)` di `index.js`. Karena awalan `/buku` sudah dipasang di sana, router cukup menggunakan path relatif seperti `/` dan `/:id`.

## Fitur API yang Tersedia

| Method | Endpoint               | Fungsi                          | Hasil                   |
| ------ | ---------------------- | ------------------------------- | ----------------------- |
| GET    | `/buku`                | Menampilkan semua buku          | 200                     |
| GET    | `/buku?kategori=sains` | Menampilkan buku kategori sains | 200                     |
| GET    | `/buku/:id`            | Melihat detail buku             | 200 atau 404            |
| POST   | `/buku`                | Menambahkan buku baru           | 201, 401, 409, 422      |
| DELETE | `/buku/:id`            | Menghapus buku                  | 204, 401, 404           |
| PUT    | `/buku/:id`            | Mengubah data buku, fitur bonus | 200, 401, 404, 409, 422 |

## Ketentuan Data Buku

Setiap data buku menggunakan field berikut:

| Field      | Ketentuan                                                                      |
| ---------- | ------------------------------------------------------------------------------ |
| `id`       | Dibuat oleh aplikasi                                                           |
| `judul`    | Wajib diisi, berupa teks 3 sampai 100 karakter                                 |
| `penulis`  | Wajib diisi, berupa teks minimal 3 karakter                                    |
| `tahun`    | Wajib berupa bilangan bulat dari 1900 sampai 2026                              |
| `stok`     | Bilangan bulat dari 0 sampai 100, nilai awal 1 jika tidak dikirim              |
| `kategori` | Wajib diisi dengan salah satu nilai: `novel`, `sains`, `sejarah`, atau `agama` |

Pemeriksaan data dilakukan menggunakan Joi dengan pengaturan `abortEarly: false`. Dengan pengaturan ini, kesalahan pada beberapa field bisa dikumpulkan dan ditampilkan sekaligus, jadi pengguna tidak perlu memperbaiki input satu per satu.

Selain validasi tersebut, aplikasi juga memeriksa judul yang sudah pernah digunakan. Pemeriksaan ini dilakukan di service karena perlu membandingkan input dengan data buku yang sudah tersimpan. Perbandingan judul tidak membedakan huruf besar dan kecil. Jika judul sudah ada, API mengirim status 409 dengan kode `JUDUL_DUPLIKAT`.

## Cara Aplikasi Menangani Error

Middleware membantu supaya proses yang digunakan berulang tidak perlu ditulis ulang di setiap endpoint.

- **Logger** mencatat method, URL, status respons, dan lama proses setiap request.
- **cekApiKey** memeriksa header `x-api-key` pada endpoint yang membutuhkan akses khusus.
- **Validasi Joi** memeriksa data sebelum controller menjalankan proses berikutnya.
- **notFound** menangani URL yang tidak memiliki route.
- **AppError** digunakan untuk error aplikasi yang sudah diketahui, misalnya buku tidak ditemukan atau judul sudah terdaftar.
- **errorHandler** menerima error dari bagian aplikasi dan mengirim respons dengan format yang konsisten.

Daftar kode error yang digunakan:

| Status | Kode                    | Keterangan                                             |
| ------ | ----------------------- | ------------------------------------------------------ |
| 401    | `UNAUTHORIZED`          | API Key tidak dikirim, salah, atau belum dikonfigurasi |
| 404    | `NOT_FOUND`             | Buku dengan ID yang diminta tidak ditemukan            |
| 404    | `ROUTE_NOT_FOUND`       | URL yang diminta tidak tersedia                        |
| 409    | `JUDUL_DUPLIKAT`        | Judul buku sudah terdaftar                             |
| 422    | `VALIDATION_ERROR`      | Data request tidak sesuai aturan validasi              |
| 500    | `INTERNAL_SERVER_ERROR` | Terjadi kesalahan di dalam server                      |

Bentuk respons error dibuat seragam, contohnya:

```json
{
  "error": {
    "pesan": "Buku tidak ditemukan",
    "kode": "NOT_FOUND"
  }
}
```

Untuk error 500, client hanya menerima pesan umum. Detail kesalahan dan stack trace dicatat di terminal agar informasi internal server tidak ikut dikirim ke pengguna API.

## Catatan Pengujian

Semua skenario pengujian disiapkan di `requests.http` dan dijalankan melalui REST Client di VS Code. Hasilnya diperiksa dari panel Response, sementara aktivitas request juga dilihat melalui terminal.

Skenario yang dicoba mencakup daftar buku, filter kategori, detail buku yang tersedia, detail buku yang tidak ditemukan, URL yang tidak dikenal, POST tanpa API Key, data tidak valid, judul duplikat, POST berhasil, DELETE berhasil, serta PUT sebagai fitur tambahan.

### Bukti Pengujian

**1. Log request di terminal**

Menampilkan catatan logger dengan status 404, 401, 422, 201, dan 204.

![Log request pada terminal](screenshot/01-log.png)

**2. Detail buku tidak ditemukan**

Request GET dengan ID yang tidak tersedia menghasilkan status 404 `NOT_FOUND`.

![GET detail buku 404](screenshot/02-detail.png)

**3. URL tidak dikenal**

Request ke alamat yang tidak terdaftar menghasilkan status 404 `ROUTE_NOT_FOUND`.

![URL tidak dikenal](screenshot/03-url.png)

**4. POST tanpa API Key**

Request ditolak karena header API Key tidak dikirim, dengan status 401.

![POST tanpa API Key](screenshot/04-key.png)

**5. Data tidak lolos validasi**

Joi menemukan input yang tidak sesuai dan API mengembalikan status 422 `VALIDATION_ERROR`.

![POST validasi 422](screenshot/05-validasi.png)

**6. Judul buku sudah terdaftar**

Request dengan judul yang sama ditolak menggunakan status 409 `JUDUL_DUPLIKAT`.

![POST judul duplikat 409](screenshot/06-duplikat.png)

**7. Berhasil menambahkan buku**

Data yang sesuai aturan berhasil disimpan dan API memberikan status 201.

![POST berhasil 201](screenshot/07-tambah.png)

**8. Berhasil menghapus buku**

Buku yang tersedia berhasil dihapus. Respons yang diterima adalah 204 No Content.

![DELETE berhasil 204](screenshot/08-hapus.png)

**9. Percobaan error 500**

Untuk menguji error handler, pemanggilan `koleksiBuku.push(bukuBaru)` di `services/bukuService.js` diubah sementara menjadi `koleksiBuku.dorong(bukuBaru)`. Request POST yang valid kemudian menghasilkan error internal. Client menerima pesan umum, sedangkan detail kesalahan terlihat di terminal. Setelah screenshot diambil, kode dikembalikan lagi menjadi `push()`.

![Pengujian error 500 pada client dan terminal](screenshot/09-error.png)

## Peran Saya dan Bantuan AI

Dalam pengerjaan tugas ini, ChatGPT digunakan sebagai alat bantu belajar dan diskusi teknis. Pembagian perannya dibuat seimbang. AI membantu memberi arahan, contoh, dan penjelasan saat ada bagian yang belum dipahami. Saya mengerjakan proses di editor, menjalankan aplikasi, mencoba request, memeriksa hasil, dan menentukan penyesuaian berdasarkan hasil yang muncul.

| Bagian                  | Bantuan ChatGPT                                                                 | Pekerjaan saya                                                                     |
| ----------------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Struktur aplikasi       | Memberi saran pembagian folder dan menjelaskan hubungan antar-layer             | Menyiapkan dan memeriksa folder serta file proyek                                  |
| Route dan controller    | Memberi contoh alur request dari route ke controller                            | Memeriksa endpoint dan mencoba request yang tersedia                               |
| Service dan aturan buku | Membantu menjelaskan logika tambah, ubah, hapus, dan pemeriksaan judul duplikat | Menjalankan fitur dan memastikan hasilnya sesuai kebutuhan                         |
| Middleware dan validasi | Menjelaskan logger, API Key, Joi, dan error handler                             | Menguji request dengan kondisi berhasil maupun gagal                               |
| Pengujian               | Membantu menyusun daftar skenario dan menjelaskan status HTTP                   | Mengirim request melalui REST Client, memeriksa terminal, dan mengambil screenshot |
| Dokumentasi             | Membantu memberi masukan dan merapikan penjelasan README                        | Memeriksa isi dokumentasi dan menyesuaikannya dengan proyek yang dikerjakan        |

**AI yang digunakan:** ChatGPT.

**Contoh prompt:** “Bantu saya memahami alur middleware pada Express dan cara menguji error handler terpusat. Jelaskan langkahnya supaya saya bisa mencoba sendiri melalui REST Client.”

ChatGPT tidak menjalankan aplikasi atau mengambil screenshot dari komputer saya. Pengujian dilakukan sendiri melalui VS Code, lalu hasilnya saya periksa dari response dan terminal.

## Penutup

Melalui tugas ini, saya belajar mengatur alur request API menggunakan beberapa middleware dan memisahkan bagian aplikasi ke dalam route, controller, service, serta data. Saya juga jadi lebih memahami perbedaan error seperti 401, 404, 409, 422, dan 500 dari hasil pengujian langsung.

Penggunaan error handler terpusat membuat penanganan kesalahan lebih rapi karena response error dikirim dari satu tempat. Selain itu, logger membantu melihat request yang masuk dan status akhirnya melalui terminal.
