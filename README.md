# K-Pedia 6.0

Pusat informasi perkeretaapian (dulu TARSUS Finder). Menu utama:

| Menu | Isi |
|---|---|
| **Beranda** | Pencarian utama (kereta, stasiun, istilah, menu), 3 menu, dan iklan otomatis tiap 3 detik |
| **Tarif Khusus** | Cari per rute atau nama kereta (tetap seperti sebelumnya), simulasi hemat, cek tiket Access by KAI, cara menggunakan, syarat dan ketentuan |
| **Peta & Stasiun** | Peta jalur kereta api dan daftar nama stasiun beserta detailnya |
| **Ensiklopedia KAI** | Istilah, jenis rangkaian, cara pemesanan dan pembatalan tiket |

Koneksi Google Sheets (`data.js`), PWA, login, favorit, dan riwayat tidak diubah.

## 1. Menjalankan dan memasang

- **GitHub Pages**: ganti seluruh isi repo `primaadis/tarsus-backup` dengan isi ZIP ini, lalu commit. Alamatnya tetap `https://primaadis.github.io/tarsus-backup/`.
- **Lokal**: buka folder ini di terminal, jalankan `python3 -m http.server 8000`, lalu buka `http://localhost:8000`.
- Setelah memperbarui file, buka aplikasi dua kali (atau hapus data situs) agar service worker memuat versi baru. Jika perubahan tidak muncul, naikkan angka `CACHE_NAME` di `sw.js`.

## 2. Struktur file

| File | Fungsi | Sering diedit? |
|---|---|---|
| `content.js` | Iklan, tautan Access by KAI, syarat dan cara pakai Tarif Khusus | Ya |
| `stations-info.js` | Informasi stasiun (peta + detail stasiun) | Ya |
| `wiki-data.js` | Data ensiklopedia bawaan (dibuat dari spreadsheet) | Jarang, lebih mudah lewat spreadsheet |
| `K-Pedia-Ensiklopedia.xlsx` | Database ensiklopedia yang mudah diedit | Ya |
| `data.js` | Koneksi dan aturan data tarif | Jangan diubah |
| `app.js`, `style.css`, `index.html` | Logika dan tampilan | Hanya bila mengubah fitur |
| `assets/` | Logo, latar, poster iklan, foto stasiun | Ya |

## 3. Tutorial pengisian

### 3.1 Mengubah iklan di beranda
1. Siapkan poster (disarankan rasio portrait 3:4, lebar minimal 800 px) dan simpan di `assets/`, misalnya `iklan-6.jpg`.
2. Buka `content.js`, cari `ads: [` lalu salin satu blok `{ image: ..., title: ..., desc: ..., link: ..., linkLabel: ... }`.
3. Isi `title` (judul singkat), `desc` (deskripsi 1 sampai 2 kalimat), dan `link` (kosongkan `""` jika tidak ada tombol).
4. Urutan di daftar = urutan tampil. Kecepatan ganti diatur `adIntervalMs` (3000 = 3 detik).
5. Tambahkan nama file gambar ke daftar `OPTIONAL_ASSETS` di `sw.js` agar tersedia offline.

### 3.2 Mengubah teks Tarif Khusus
Di `content.js`, bagian `tarsus`: `terms` untuk **Syarat dan ketentuan**, `steps` untuk **Cara menggunakan**. Tautan Access by KAI dan kontak ada di `links`. **Teks bawaan bersifat umum, cocokkan dengan ketentuan resmi sebelum dipublikasikan.**

### 3.3 Mengisi informasi stasiun (detail, foto, peta)
Buka `stations-info.js`. Satu stasiun = satu blok. Untuk **menambah stasiun**, salin satu blok lalu ubah:

| Kolom | Keterangan |
|---|---|
| `id` | kode unik huruf kecil, mis. `jat` |
| `nama` | nama stasiun, sama dengan `NAMA_STASIUN` di Google Sheets |
| `alias` | nama lain untuk pencocokan, mis. `["Jatinegara","Meester Cornelis"]` |
| `kategori` | `antarkota`, `commuter`, `bandara`, `whoosh`, atau `heritage` |
| `status` | `active` atau `inactive` |
| `region`, `provinsi`, `kota` | wilayah (samakan penulisan Daop/Divre dengan sheet agar filter rapi) |
| `koordinat` | `[lintang, bujur]`, ambil dari Google Maps (klik kanan pada lokasi). Kosongkan `[]` bila belum ada, stasiun tetap tampil di daftar namun tidak muncul di peta |
| `ulasan`, `deskripsi`, `sejarah`, `fasilitas`, `denah` | isi sesukamu. **Bagian yang kosong otomatis disembunyikan** |

**Foto**: simpan sebagai `assets/stations/<kode>.jpg` (kode = nilai `foto` di blok, mis. `assets/stations/jat.jpg`). Rasio 16:9, lebar 1200 px, di bawah 300 KB. Jika file belum ada, tampil gambar placeholder.

Stasiun yang ada di Google Sheets namun belum punya blok di sini tetap muncul di **Nama Stasiun** dengan data dasar dan daftar kereta tarif khusus yang melintas. Stasiun berbadge **Info** sudah punya detail.

> Data awal diambil dari ZIP railmap (27 stasiun) dan hanya dirapikan. Ulasan, tahun, dan fasilitas di dalamnya belum saya verifikasi, mohon cek sebelum dipublikasikan.

### 3.4 Peta
- Peta memakai Leaflet (dimuat dari internet saat menu Peta dibuka), dasar peta Esri/OSM, dan jalur rel dari OpenRailwayMap. **Peta butuh koneksi internet.**
- Pin hanya muncul untuk stasiun yang punya `koordinat`. Warna pin mengikuti `kategori`.
- Mengubah warna atau nama kategori: objek `CAT` di `app.js`.

### 3.5 Ensiklopedia (lewat spreadsheet)
1. Buka `K-Pedia-Ensiklopedia.xlsx` (bisa diunggah ke Google Sheets: Drive > Baru > Unggah file > Buka dengan Google Sheets). Baca sheet **PETUNJUK**.
2. Edit sheet **WIKI** (satu baris = satu topik) dan **WIKI_KATEGORI**. Teks biru boleh diubah. Format kolom `ISI`: baris kosong = paragraf baru, `- ` = poin, `1. ` = langkah.
3. **Sementara** (sebelum tersambung): salin perubahan ke `wiki-data.js`, atau minta saya membuatkan ulang `wiki-data.js` dari spreadsheet yang sudah kamu edit.
4. **Saat sudah tersambung ke Google Sheets**: bagikan sheet sebagai "Siapa saja yang memiliki link: Pelihat", isi `wiki.sheetId` di `content.js` dengan ID sheet (bagian antara `/d/` dan `/edit` pada URL). Kode penyambung sudah disiapkan; selama `sheetId` kosong, data bawaan dipakai. Nama sheet dan kolom harus persis seperti file ini.

Semua isi ensiklopedia bawaan adalah ringkasan umum dan sebagian bersumber dari pemberitaan (kolom `SUMBER`). **Baris bertanda "Perlu verifikasi" harus dicek ke sumber resmi KAI**. Ini mencakup ketentuan pembatalan (bea 25%, batas 2 jam dan 30 menit), nomor WhatsApp KAI, dan tarif KRL.

### 3.6 Latar belakang
Mode gelap memakai `assets/background.jpg` (senja), mode terang memakai `assets/background-boot.jpg` (siang). Ganti file dengan nama sama untuk mengubahnya (portrait, sekitar 900 x 1200 px, di bawah 400 KB). Tingkat kekaburan: `--blur` di `style.css`.

### 3.7 Logo boot screen
Animasi memakai bentuk vektor huruf K (`TARGET` di fungsi `runBoot`, `app.js`) dan `assets/logo-wordmark.png` untuk tulisan "-PEDIA by KAI". Jika logo diganti, bentuk vektor K perlu disesuaikan.

## 4. Catatan penting

- **Login** (`admin`) hanya penghalang tampilan di sisi browser (tidak aman). Jangan menyimpan data rahasia di aplikasi ini.
- Tidak ada klaim sebagai aplikasi resmi KAI. Tautan hanya mengarah ke Access by KAI.
- Tarif, relasi, dan ketentuan dapat berubah. Selalu pastikan ke sumber resmi.
