# Panduan Pemasangan Sadhu Health

Sadhu Health berjalan di HP seperti aplikasi biasa, tetap bisa dipakai tanpa internet, dan menyimpan data di Google Drive milik keluarga. Setiap anggota keluarga mendaftar dan masuk memakai email dan password sendiri.

Pemasangan dilakukan **sekali saja oleh satu orang**, yaitu pemilik data keluarga. Waktu yang dibutuhkan sekitar 45–60 menit. Semua layanan yang dipakai gratis.

## Isi paket

| File | Fungsi |
|---|---|
| `index.html` | Aplikasinya |
| `config.js` | Tempat mengisi kunci Google dan reCAPTCHA milikmu (Langkah 3) |
| `sw.js` | Membuat aplikasi tetap terbuka tanpa internet |
| `manifest.webmanifest`, `logo.png`, `icon-*.png` | Nama, logo, dan ikon saat dipasang di layar utama HP |
| `PANDUAN.md` | Panduan ini |

## Gambaran cara kerjanya

Ada dua lapis di aplikasi ini, dan keduanya dibutuhkan bersama:

1. **Penyambungan perangkat** (sekali per HP, lewat akun Google) — inilah yang benar-benar memberi aplikasi izin membaca dan menulis ke Google Drive. Ini aturan dari Google sendiri: hanya akun Google yang secara langsung memberi izin yang boleh mengakses Drive-nya. Dilakukan sekali saat HP pertama kali dipasang.
2. **Masuk sehari-hari** (email + password, per orang) — inilah yang dipakai setiap kali membuka aplikasi. Setiap anggota keluarga punya akun sendiri, mendaftar sekali, dan aktivasinya disetujui pemilik lewat WhatsApp.

```
Drive Saya (pemilik)/
└── Sadhu Health/                    ← dibagikan ke HP anggota keluarga
    ├── data-catatan-sehat.json      ← seluruh data: catatan kesehatan, akun, log aktivitas
    └── Laporan/                     ← laporan PDF, Excel, CSV
```

Catatan jujur soal aktivasi WhatsApp: aplikasi ini **tidak mengirim pesan WhatsApp secara otomatis**, karena itu perlu layanan WhatsApp Business berbayar dengan server sendiri. Sebagai gantinya, saat mendaftar, aplikasi membuka WhatsApp dengan pesan berisi kode aktivasi yang sudah terisi, dikirim ke nomor WhatsApp pemilik. Pemilik lalu menyetujui pendaftaran itu langsung di aplikasi. Tetap memakai WhatsApp sungguhan, hanya saja persetujuannya manual oleh pemilik.

Catatan jujur soal reCAPTCHA: tanpa server sendiri, reCAPTCHA di sini hanya bisa menampilkan tantangan dari Google (jika dikonfigurasi di Langkah 3) untuk mempersulit bot dan skrip otomatis, tanpa verifikasi tersembunyi di server. Ini tetap menghalangi spam biasa secara nyata, hanya bukan jaminan tingkat perusahaan besar. Jika dilewati, aplikasi memakai centang "Saya bukan robot" sebagai gantinya.

---

## Langkah 1: Taruh aplikasi di internet lewat GitHub Pages

1. Buka **github.com** lalu daftar atau login.
2. Klik tombol **+** di kanan atas, pilih **New repository**.
3. Isi *Repository name* dengan `sadhu-health`. Pilih **Public**, lalu klik **Create repository**.
4. Di halaman berikutnya, klik tautan **uploading an existing file**.
5. Ekstrak file zip, lalu seret semua isinya ke halaman itu. Klik **Commit changes**.
6. Buka tab **Settings**, lalu menu **Pages** di sebelah kiri.
7. Pada *Build and deployment*, pilih *Source*: **Deploy from a branch**. Pilih *Branch*: **main** dan folder **/ (root)**, lalu klik **Save**.
8. Tunggu 1–2 menit, lalu muat ulang halaman. Alamat aplikasimu akan muncul, misalnya:
   `https://namakamu.github.io/sadhu-health/`

Catat dua hal ini untuk langkah berikutnya:

- **Alamat aplikasi**: `https://namakamu.github.io/sadhu-health/`
- **Alamat asal (origin)**: `https://namakamu.github.io`, yaitu tanpa nama folder dan tanpa garis miring di akhir.

## Langkah 2: Siapkan akses Google Drive di Google Cloud

Gunakan akun Google milik pemilik data keluarga. Tampilan Google Cloud kadang berubah sedikit, jadi carilah nama menu yang mirip.

### 2a. Buat project

1. Buka **console.cloud.google.com**.
2. Klik pemilih project di bagian atas, lalu **New Project**.
3. Beri nama `Sadhu Health`, lalu klik **Create**.
4. Pastikan project baru itu yang sedang terpilih di bagian atas.

### 2b. Aktifkan dua API

1. Buka menu **APIs & Services**, lalu **Library**.
2. Cari **Google Drive API**, buka, lalu klik **Enable**.
3. Kembali ke Library, cari **Google Picker API**, lalu klik **Enable**.

### 2c. Atur layar izin login

1. Buka menu **APIs & Services**, lalu **OAuth consent screen** (di tampilan baru bernama **Google Auth Platform**). Klik **Get started**.
2. Isi *App name* dengan `Sadhu Health` dan pilih email kamu sebagai *User support email*.
3. Pada *Audience*, pilih **External**. Isi email kontak, centang persetujuan, lalu klik **Create**.
4. Buka **Data access**, klik **Add or remove scopes**. Cari dan centang izin berakhiran `auth/drive.file`. Klik **Update**, lalu **Save**.
5. Buka **Audience**, klik **Publish app**, lalu **Confirm** sampai statusnya **In production**.

Izin `drive.file` hanya memberi akses ke file yang dibuat aplikasi ini sendiri, bukan ke seluruh isi Drive, sehingga tidak perlu proses verifikasi Google yang panjang.

### 2d. Buat Client ID

1. Buka **Google Auth Platform**, lalu **Clients**, lalu **Create client**.
2. Pilih *Application type*: **Web application**.
3. Pada **Authorized JavaScript origins**, klik **Add URI** dan isi alamat asal dari Langkah 1, misalnya `https://namakamu.github.io`.
4. Klik **Create**, lalu salin **Client ID** yang berakhiran `.apps.googleusercontent.com`.

### 2e. Buat API key untuk pemilih file

1. Buka **APIs & Services**, lalu **Credentials**, lalu **Create credentials**, lalu **API key**. Salin kunci yang muncul (diawali `AIza`).
2. Klik nama kunci itu untuk mengeditnya.
3. Pada *Application restrictions*, pilih **Websites**, lalu tambahkan `https://namakamu.github.io/*`.
4. Pada *API restrictions*, pilih **Restrict key** dan centang **Google Picker API**. Klik **Save**.

### 2f. Salin nomor project

Klik ikon titik tiga di kanan atas, pilih **Project settings** (atau menu **IAM & Admin** → **Settings**). Salin **Project number**, yang berisi angka saja. Jangan tertukar dengan *Project ID*.

## Langkah 3: Siapkan reCAPTCHA (opsional, tapi disarankan)

Langkah ini boleh dilewati — aplikasi tetap berjalan dengan centang "Saya bukan robot" sebagai gantinya. Untuk memakai reCAPTCHA sungguhan dari Google:

1. Buka **google.com/recaptcha/admin/create**.
2. Isi *Label* dengan `Sadhu Health`.
3. Pilih jenis **reCAPTCHA v2**, lalu pilihan **"I'm not a robot" Checkbox**.
4. Pada *Domains*, tambahkan domain dari alamat asalmu tanpa `https://`, misalnya `namakamu.github.io`.
5. Centang persetujuan, klik **Submit**.
6. Salin **Site key** yang muncul. Kunci **Secret key** di halaman itu tidak dipakai di sini karena butuh server untuk memverifikasinya — boleh diabaikan.

## Langkah 4: Isi config.js

1. Di GitHub, buka repository `sadhu-health`, lalu klik file `config.js`.
2. Klik ikon pensil untuk mengedit.
3. Ganti nilai `ISI_...` dengan milikmu. Tanda kutipnya tetap dipakai. Kosongkan `recaptchaSiteKey` (biarkan `ISI_RECAPTCHA_SITE_KEY`) jika melewati Langkah 3.

```js
window.CSK_CONFIG = {
  clientId: '1234567890-abc123.apps.googleusercontent.com',
  apiKey:   'AIzaSyA1b2C3d4E5f6G7h8I9j0KlMnOpQrStUv',
  appId:    '1234567890',
  recaptchaSiteKey: '6Lc...' // atau biarkan ISI_RECAPTCHA_SITE_KEY untuk pakai centang cadangan
};
```

4. Klik **Commit changes**, lalu tunggu sekitar 1 menit.

## Langkah 5: Pasang di HP

- **Android (Chrome):** buka alamat aplikasi, ketuk menu titik tiga, lalu pilih **Instal aplikasi** atau **Tambahkan ke layar utama**.
- **iPhone (Safari):** buka alamat aplikasi, ketuk tombol **Bagikan**, lalu pilih **Tambah ke Layar Utama**.

## Langkah 6: Pemilik menyambungkan perangkat dan mendaftar

1. Buka aplikasi. Karena belum ada penyimpanan keluarga, layar **Penyambungan perangkat** akan muncul lebih dulu.
2. Ketuk **Buat penyimpanan keluarga baru**, lalu pilih akun Google kamu dan izinkan akses ke Google Drive. Folder **Sadhu Health** akan dibuat di Drive-mu.
3. Setelah tersambung, layar **Daftar** muncul otomatis untuk pendaftar pertama. Isi nama lengkap, username, email, nomor WhatsApp, tanggal lahir, alamat, password, dan reCAPTCHA, lalu ketuk **Daftar**.
4. Pendaftar pertama ini otomatis menjadi **pemilik data keluarga** dan langsung aktif tanpa perlu aktivasi WhatsApp — wajar, karena belum ada orang lain yang bisa menyetujuinya.
5. Setelah masuk, buka menu **Profil**, gulir ke bagian **Persetujuan akun baru**, lalu isi **Nomor WhatsApp untuk menerima kode aktivasi** dengan nomor WhatsApp aktif milikmu (format `62812xxxxxxx`, tanpa tanda `+`), lalu ketuk **Simpan nomor**.
6. Tambahkan profil kesehatan anggota keluarga (Saya, Ibu, Ayah, dan seterusnya) lewat tombol **Tambah anggota keluarga** di beranda.

Kalau sebelumnya kamu sudah mencatat di versi percobaan (Catatan Sehat Keluarga di Claude), buka versi itu, ketuk **Backup**, lalu **Unduh backup (.json)**, dan simpan filenya. Setelah mendaftar di Sadhu Health, buka menu **Drive**, pilih **Pulihkan dari file backup**, lalu pilih file tersebut untuk menggabungkan catatan kesehatan lama.

## Langkah 7: Anggota keluarga bergabung

Setiap anggota melakukan langkah ini di HP masing-masing:

1. Pasang aplikasi dari alamat yang sama seperti di Langkah 5.
2. Di layar Penyambungan perangkat, ketuk **Buka data keluarga yang sudah ada**.
3. Login dengan akun Google sendiri dan izinkan akses. Di pemilih file, pilih **data-catatan-sehat.json**.
4. Setelah tersambung, ketuk tab **Daftar**, lalu isi formulirnya, termasuk password sendiri.
5. Setelah mendaftar, layar **Menunggu aktivasi** akan muncul berisi kode 6 digit. Ketuk **Kirim kode lewat WhatsApp** — ini akan membuka WhatsApp dengan pesan berisi nama, username, dan kode, terkirim ke nomor pemilik.
6. Tunggu pemilik menyetujui (Langkah 8), lalu ketuk **Periksa status aktivasi**. Begitu aktif, akan diarahkan ke layar Masuk untuk login dengan email/username dan password yang baru dibuat.

## Langkah 8: Pemilik menyetujui pendaftaran baru

1. Buka aplikasi Sadhu Health, ketuk profil di kanan atas.
2. Kalau perlu, ketuk tulisan status sinkron di bawah judul aplikasi untuk memuat data terbaru.
3. Di bagian **Persetujuan akun baru**, cocokkan nama dan kode yang diterima lewat WhatsApp dengan yang tampil di aplikasi, lalu ketuk **Aktifkan**.

---

## Pemakaian sehari-hari

- **Status sinkron** tampil di bawah judul aplikasi. Ketuk tulisan itu kapan saja untuk sinkron manual. Login Google (untuk akses Drive) berlaku sekitar satu jam; aplikasi akan meminta ulang secara diam-diam saat dibutuhkan.
- Selama aplikasi terbuka dan ada internet, aplikasi mengecek data baru setiap 30 detik. Perubahan milikmu dikirim dalam beberapa detik.
- Saat **tidak ada sinyal**, pencatatan tetap jalan dan tersimpan di HP. Perubahan dikirim otomatis begitu kembali online.
- **Log aktivitas** di menu Profil mencatat setiap kali seseorang masuk, gagal masuk, keluar, membuka menu, mengisi data, atau disetujui aktivasinya — gunakan ini untuk memeriksa apakah ada akses yang mencurigakan.
- **Tombol SOS** (lingkaran merah) memanggil 112 dengan satu ketukan, dan menampilkan kontak favorit yang diatur di menu Profil.
- Saat mengekspor laporan, pilih **Google Drive** untuk menyimpan ke folder Laporan, atau **Unduh ke HP**.

## Keamanan dan privasi — batasannya, secara jujur

- **Password** disimpan dalam bentuk hash (diacak dengan PBKDF2, bukan teks biasa) di file data keluarga. Ini lebih aman daripada teks polos, tetapi tetap hanya seaman siapa saja yang kamu beri akses ke folder Google Drive-nya. Jangan beri akses folder ke orang di luar keluarga.
- **reCAPTCHA** di sini hanya berupa tantangan visual dari Google tanpa verifikasi server, jadi ia menghalangi bot dan skrip sederhana, bukan penyerang yang benar-benar menargetkan aplikasi ini secara khusus.
- **Aktivasi WhatsApp** bergantung pada kejelian pemilik saat menyetujui — pastikan kode yang diterima lewat WhatsApp cocok dengan yang tampil di aplikasi sebelum menekan Aktifkan.
- Hanya orang yang kamu undang lewat menu **Drive** yang bisa membuka folder data keluarga. Jangan ubah akses folder menjadi **Siapa saja yang memiliki link**.
- File `data-catatan-sehat.json` boleh dipindah ke folder lain, tetapi **jangan diedit isinya secara manual atau dihapus**. Kalau terhapus, pulihkan dari **Sampah** di Google Drive.
- Sesekali unduh **backup manual** (menu Drive → Backup manual) sebagai cadangan tambahan.
- Kode aplikasi di GitHub bersifat publik, tetapi tidak berisi data siapa pun. Client ID, API key, dan reCAPTCHA site key memang dirancang untuk berada di halaman web, dan sudah dibatasi hanya untuk alamat situsmu.

## Nama aplikasi: satu hal yang perlu kamu lakukan sendiri

Nama **Sadhu Health** belum ditemukan dipakai di sumber-sumber umum saat dicek, tetapi ini bukan pengecekan resmi. Sebelum benar-benar dipakai secara luas atau didaftarkan sebagai merek, periksa langsung status ketersediaannya di **pdki-indonesia.dgip.go.id** (Pangkalan Data Kekayaan Intelektual, Ditjen KI Kemenkumham).

## Kalau ada masalah

| Yang terjadi | Yang perlu dicek |
|---|---|
| Muncul **Error 400: origin_mismatch** saat login Google | *Authorized JavaScript origins* di Langkah 2d harus persis sama dengan alamat asal, tanpa garis miring di akhir. |
| Muncul layar **Google belum memverifikasi aplikasi ini** | Ketuk **Lanjutan**, lalu **Buka Sadhu Health**. Ini aplikasimu sendiri. |
| Jendela login Google tidak muncul | Izinkan pop-up untuk situs aplikasi di pengaturan browser. |
| reCAPTCHA tidak muncul, hanya centang biasa | Site key belum diisi di Langkah 4, atau domain di Langkah 3 tidak cocok dengan alamat aplikasi. Aplikasi tetap bisa dipakai dengan centang cadangan. |
| Kode aktivasi WhatsApp tidak terkirim otomatis | Ini memang manual — anggota keluarga sendiri yang mengetuk tombol kirim di WhatsApp mereka. Pastikan nomor pemilik sudah diisi di menu Profil. |
| Anggota sudah disetujui tapi masih tertahan di layar Menunggu aktivasi | Ketuk **Periksa status aktivasi** di layar itu untuk memuat status terbaru dari Google Drive. |
| Username ditolak padahal terasa belum dipakai | Pastikan HP sudah tersambung ke data keluarga (Langkah 7, poin 2–3) sebelum mendaftar, supaya daftar username yang dibandingkan sudah yang terbaru. |
| Pemilih file kosong atau gagal dimuat | Pastikan Google Picker API sudah aktif, pembatasan API key sudah benar, dan nomor project di `config.js` berupa angka *Project number*. Ketik `data-catatan-sehat` di kotak pencarian pemilih file. |
| Muncul **Akses ke data keluarga terputus** | Akses mungkin dicabut atau file terhapus. Buka menu Drive, ketuk **Putuskan HP ini dari Google Drive**, lalu ulangi Langkah 7. |

## Memperbarui aplikasi di kemudian hari

1. Unggah `index.html` versi baru ke GitHub, menggantikan yang lama.
2. Edit `sw.js` dan naikkan angka versinya, misalnya dari `sadhu-v1` menjadi `sadhu-v2`.
3. Di HP, tutup lalu buka aplikasi dua kali agar versi baru terpasang.

Data tidak hilang saat aplikasi diperbarui, karena data tersimpan di HP dan di Google Drive, bukan di dalam kode.
