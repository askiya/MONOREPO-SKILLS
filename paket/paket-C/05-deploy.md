# 05 — Deploy ke cPanel (Paket C)

## Tujuan

Website online di cPanel shared hosting. File statis di `public_html/`,
PHP API berjalan, MySQL database aktif.

---

## Sebelum Mulai

- [ ] Build berhasil, folder `out/` ada ([03-build.md](03-build.md))
- [ ] Preview localhost OK ([04-preview.md](04-preview.md))
- [ ] Punya akses ke cPanel hosting (URL, username, password)
- [ ] Domain sudah diarahkan ke server hosting

---

## Referensi Utama

> **Semua langkah detail ada di:**
> [`../../docs/11-hosting-cpanel/`](../../docs/11-hosting-cpanel/) — 6 bab lengkap
>
> Panduan ini adalah **ringkasan alur** yang merujuk ke sana.

---

## Langkah

### 1. Login ke cPanel

Buka URL cPanel:
- Biasanya `https://domainmu.com:2083` atau `https://domainmu.com/cpanel`
- Login dengan credential dari provider hosting

### 2. Buat MySQL Database

Detail: [`../../docs/11-hosting-cpanel/06-database-dan-credential.md`](../../docs/11-hosting-cpanel/06-database-dan-credential.md)

1. cPanel → **MySQL Databases**
2. Create New Database → isi nama → Create
3. Create New User → isi username + password (simpan di tempat aman, **bukan di kode**)
4. Add User to Database → pilih user dan database → All Privileges → Make Changes
5. Buka **phpMyAdmin** → pilih database → tab **Import** → upload `database/schema.sql`

**Berhasil kalau:** tabel muncul di phpMyAdmin.

### 3. Upload File Statis ke public_html

Detail: [`../../docs/11-hosting-cpanel/04-upload-file.md`](../../docs/11-hosting-cpanel/04-upload-file.md)

Panduan config: [`../../deployment-examples/cpanel-static/`](../../deployment-examples/cpanel-static/)

#### Cara 1: File Manager (Disarankan untuk Pemula)

1. Di lokal, zip **isi** folder `out/`:

```bash
cd out
zip -r ../site.zip .
cd ..
```

> ⚠️ Zip **isi** folder `out/`, bukan folder `out/` itu sendiri.
> Hasilnya: `site.zip` berisi `index.html`, `_next/`, dll langsung di root zip.

2. cPanel → **File Manager** → masuk ke `public_html/`
3. Hapus file bawaan hosting (kalau ada `index.html` default)
4. Klik **Upload** → upload `site.zip`
5. Klik kanan `site.zip` → **Extract** → extract ke `public_html/`
6. Hapus `site.zip` setelah extract

#### Cara 2: FTP (Untuk File Banyak)

```bash
# Pakai lftp atau FileZilla
# Host, user, password dari cPanel → FTP Accounts
lftp -u USER,PASS HOST -e "mirror -R out/ public_html/; quit"
```

### 4. Upload .htaccess

Copy `.htaccess` dari `deployment-examples/cpanel-static/`:

```bash
# Di cPanel File Manager, buat file .htaccess di public_html/
# Atau upload file yang sudah ada
```

Isi `.htaccess` yang dibutuhkan:

```apache
# Force HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# SPA fallback (kalau pakai client-side routing)
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ /index.html [L]
```

> File `.htaccess` mungkin tersembunyi di File Manager.
> Klik **Settings** (kanan atas) → centang **Show Hidden Files**.

### 5. Upload PHP API (Kalau Ada)

1. Upload folder `api/` ke `public_html/api/`
2. Buat `api/config.php` **langsung di server** (jangan upload dari lokal):
   - cPanel → File Manager → `public_html/api/` → New File → `config.php`
   - Isi credential database dari langkah 2
3. Set permission: folder `755`, file `644`

```bash
# Verifikasi PHP berjalan
curl https://domainmu.com/api/contact.php
```

**Output yang benar:** response JSON (meski error method karena GET).

### 6. Setup Node.js App (Opsional — Kalau Hosting Mendukung)

Detail: [`../../docs/11-hosting-cpanel/05-nodejs-di-cpanel.md`](../../docs/11-hosting-cpanel/05-nodejs-di-cpanel.md)

Panduan config: [`../../deployment-examples/cpanel-node/`](../../deployment-examples/cpanel-node/)

> ⚠️ Tidak semua cPanel shared hosting punya menu **Setup Node.js App**.
> Kalau tidak ada, jangan paksa. Pakai PHP atau pindah ke Paket A (Vercel).

Kalau ada:

1. Upload `server.js` ke folder di luar `public_html/`, misal `/home/USER/app/`
2. cPanel → **Setup Node.js App** → Create Application
3. Isi:
   - Node version: LTS
   - Application mode: Production
   - Application root: `app`
   - Application URL: domain/subdomain target
   - Startup file: `server.js`
4. **Jangan** isi PORT manual — cPanel/Passenger memberi port otomatis
5. Save → Restart Application

### 7. Verifikasi

```bash
# Cek HTTPS
curl -I https://domainmu.com

# Cek redirect HTTP → HTTPS
curl -I http://domainmu.com

# Cek halaman utama
curl -s https://domainmu.com | head -20
```

**Output yang benar:**
- HTTPS: status `200 OK`
- HTTP: status `301 Moved Permanently`, Location ke HTTPS
- Halaman: HTML dari Next.js

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| 403 Forbidden | `index.html` tidak di root `public_html/` | Pindahkan semua file ke `public_html/` langsung |
| 404 Not Found | File belum diextract atau path salah | Cek `public_html/index.html` ada |
| Halaman blank | Asset `_next/` tidak ikut terupload | Pastikan zip berisi `_next/` folder |
| PHP error 500 | Credential database salah | Cek `api/config.php`, test koneksi |
| Node.js app tidak jalan | Menu Setup Node.js App tidak ada | Hosting tidak support, pakai PHP saja |
| CSS/JS 404 | basePath salah di `next.config.js` | Sesuaikan basePath atau hapus kalau deploy di root domain |
| "Too many redirects" | `.htaccess` redirect loop | Cek apakah hosting sudah force HTTPS otomatis — hapus rule HTTPS di `.htaccess` |

---

## Checklist

- [ ] MySQL database dibuat di cPanel
- [ ] Tabel berhasil diimport via phpMyAdmin
- [ ] File statis terupload ke `public_html/`
- [ ] `index.html` ada langsung di `public_html/index.html`
- [ ] `.htaccess` sudah di-upload
- [ ] `https://domainmu.com` menampilkan website
- [ ] `http://domainmu.com` redirect ke HTTPS
- [ ] PHP API berfungsi (kalau ada)
- [ ] `api/config.php` ada di server tapi TIDAK di Git
- [ ] Node.js app berjalan (kalau pakai)
