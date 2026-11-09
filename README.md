# Sistem Perpustakaan

![CI](https://github.com/HQID/modul3/actions/workflows/ci_pipeline.yml/badge.svg)

## Deskripsi Aplikasi

Sistem Perpustakaan adalah aplikasi desktop berbasis Python yang membantu petugas perpustakaan mengelola data buku dan proses autentikasi pengguna. Aplikasi dibangun dengan pola arsitektur **MVC (Model-View-Controller)**, antarmuka grafis menggunakan **CustomTkinter**, dan penyimpanan data pada **MySQL**.

Fitur utama:

- Login petugas dengan password yang disimpan dalam bentuk hash SHA-256.
- Pengelolaan data buku.
- Pengujian otomatis (unit test) dengan pytest.
- Integrasi berkelanjutan (CI) menggunakan GitHub Actions: setiap push dan pull request ke branch `main` diperiksa oleh linter flake8 dan dijalankan pengujiannya.
- Dapat dikemas menjadi berkas `.exe` mandiri dengan PyInstaller.

## Struktur Direktori MVC

```text
modul3/
├── .github/
│   └── workflows/
│       └── ci_pipeline.yml      # Konfigurasi CI (flake8 + pytest)
├── config/                      # Konfigurasi koneksi database
├── models/                      # Model: akses data & logika bisnis (mis. user_model.py)
├── views/                       # View: tampilan antarmuka (CustomTkinter)
├── controllers/                 # Controller: penghubung View dan Model
├── tests/
│   ├── test_auth.py             # Unit test autentikasi & hashing SHA-256
│   └── test_buku.py             # Unit test fitur buku
├── main.py                      # Titik masuk aplikasi
├── requirements.txt             # Daftar dependensi Python
├── .gitignore
└── README.md
```

| Lapisan | Folder | Tanggung jawab |
|---|---|---|
| Model | `models/` | Berinteraksi dengan database MySQL dan menyimpan logika data. |
| View | `views/` | Menampilkan antarmuka dan menerima input pengguna. |
| Controller | `controllers/` | Menerima aksi dari View, memanggil Model, lalu memperbarui View. |

## Panduan Instalasi & Konfigurasi Database MySQL

### 1. Prasyarat

- Python 3.11 atau lebih baru
- Git
- [Laragon](https://laragon.org/) (menyediakan MySQL)
- Visual Studio Code (opsional)

### 2. Clone repositori dan siapkan lingkungan Python

```bash
git clone https://github.com/HQID/modul3.git
cd modul3

python -m venv env
env\Scripts\activate

pip install -r requirements.txt
```

### 3. Konfigurasi database MySQL (Laragon)

1. Buka Laragon, lalu klik **Start All** hingga MySQL berstatus berjalan.
2. Buka HeidiSQL atau phpMyAdmin dari Laragon, lalu buat database baru:

   ```sql
   CREATE DATABASE db_perpustakaan;
   ```

3. Buat tabel yang dibutuhkan. Contoh untuk tabel pengguna (sesuaikan dengan skema proyek Anda):

   ```sql
   USE db_perpustakaan;

   CREATE TABLE users (
       id_user  INT AUTO_INCREMENT PRIMARY KEY,
       username VARCHAR(50)  NOT NULL UNIQUE,
       password CHAR(64)     NOT NULL,   -- hash SHA-256 (64 karakter heksadesimal)
       role     VARCHAR(20)  NOT NULL
   );

   -- Contoh akun petugas (password: admin123)
   INSERT INTO users (username, password, role)
   VALUES ('petugas', SHA2('admin123', 256), 'admin');
   ```

   Jika proyek memiliki berkas `.sql`, impor berkas tersebut melalui HeidiSQL atau phpMyAdmin.

4. Sesuaikan pengaturan koneksi di folder `config/`. Nilai bawaan Laragon:

   | Parameter | Nilai |
   |---|---|
   | Host | `localhost` |
   | Port | `3306` |
   | User | `root` |
   | Password | *(kosong)* |
   | Database | `db_perpustakaan` |

## Cara Menjalankan Aplikasi

### Dari kode sumber

1. Pastikan MySQL pada Laragon sudah menyala.
2. Aktifkan virtual environment:

   ```bash
   env\Scripts\activate
   ```

3. Jalankan aplikasi:

   ```bash
   python main.py
   ```

4. Login menggunakan akun petugas yang terdaftar pada tabel `users`.

### Menjalankan pengujian

```bash
pytest -v
```

### Dari berkas `.exe` (pengguna akhir)

1. Unduh `AplikasiPerpustakaan.exe` dari halaman **Releases** repositori ini.
2. Pastikan MySQL pada Laragon sudah menyala.
3. Klik dua kali `AplikasiPerpustakaan.exe`.

### Membangun ulang berkas `.exe`

```bash
pip install pyinstaller
pyinstaller --noconfirm --onefile --windowed --name "AplikasiPerpustakaan" main.py
```

Hasilnya berada di folder `dist/`.

## Pipeline CI

Berkas `.github/workflows/ci_pipeline.yml` menjalankan langkah berikut pada setiap push atau pull request ke `main`:

1. Checkout kode dari repositori.
2. Menyiapkan Python 3.11.
3. Menginstal dependensi dari `requirements.txt`.
4. Memeriksa kualitas kode dengan **flake8** pada folder `models/` dan `controllers/`.
5. Menjalankan unit test dengan **pytest**.
