# 04 — Upload File ke cPanel

## Tujuan

File project masuk ke hosting, website tampil di domain.

## Tiga Cara Upload

### A. File Manager (paling mudah, file kecil)

1. cPanel → **File Manager** → masuk `public_html`.
2. **Upload** → pilih file / ZIP.
3. Kalau upload ZIP:
   - Upload file `.zip` ke `public_html`.
   - Klik kanan → **Extract**.
   - Pastikan `index.html` langsung di `public_html`, bukan dalam subfolder.
   - Hapus file `.zip` setelah extract.

Batasan: File Manager lambat untuk banyak file. Ukuran upload terbatas tergantung
hosting (biasanya 256MB–1GB per file).

### B. FTP / SFTP (banyak file, lebih cepat)

1. cPanel → **FTP Accounts** → buat akun FTP kalau belum ada.
2. Catat: host, username, password, port.
3. Install FTP client: **FileZilla** (gratis).
4. FileZilla → Site Manager:

```
Host: domainmu.com (atau IP hosting)
Protocol: SFTP - SSH File Transfer Protocol (kalau tersedia)
           FTP - File Transfer Protocol (fallback)
Port: 21 (FTP) atau 22 (SFTP)
Logon Type: Normal
User: username_cpanel atau FTP user
Password: password
```

5. Connect → navigasi ke `/public_html` di panel kanan.
6. Drag & drop file dari kiri (lokal) ke kanan (server).

**Tips:** SFTP lebih aman dari FTP biasa. Pakai SFTP kalau hosting mendukung.

### C. Git Deploy (paling rapi)

Beberapa hosting mendukung Git:

1. cPanel → **Git Version Control** → Create.
2. Clone URL: `https://github.com/user/repo.git`.
3. Repository Path: `/home/user/repositories/myproject`.
4. Deploy ke `public_html` via `.cpanel.yml`:

```yaml
---
deployment:
  tasks:
    - export DEPLOYPATH=/home/user/public_html
    - /bin/cp -R build/* $DEPLOYPATH/
```

5. Setiap push ke GitHub → pull di cPanel → deploy otomatis.

**Catatan:** Jangan taruh seluruh repo di `public_html`. Buat build di tempat
lain, copy hasil build saja.

## Upload Situs Statis (HTML/CSS/JS)

Setelah `npm run build` lokal menghasilkan folder output:

```bash
# Contoh Next.js static export
# output: folder out/
```

1. Zip isi folder hasil build.
2. Upload ke `public_html` via File Manager.
3. Extract.
4. Pastikan `index.html` ada di root `public_html`.

## Upload Next.js SSR / Node.js

Lihat bab terpisah (05-nodejs-di-cpanel.md).

## Struktur yang Benar

```
public_html/
├── index.html          ← harus di sini
├── _next/              ← Next.js static assets
├── css/
├── js/
├── images/
├── favicon.ico
└── .htaccess
```

**JANGAN** taruh di `public_html`:
- `.env` — rahasia
- `node_modules/` — tidak perlu untuk static
- `.git/` — riwayat kode
- backup database
- file credential

## SPA Routing (.htaccess)

Kalau pakai React/Vue SPA, halaman refresh selain `/` akan 404.
Tambah `.htaccess`:

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

## Checklist

- [ ] File berhasil di-upload
- [ ] `index.html` di root `public_html`
- [ ] Website tampil di browser
- [ ] Tidak ada file rahasia di `public_html`
- [ ] `.htaccess` SPA routing aktif (kalau SPA)
