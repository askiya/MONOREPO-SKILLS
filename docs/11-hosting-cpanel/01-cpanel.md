# 01 — Hosting cPanel

## Tujuan

Memakai shared hosting cPanel ketika VPS belum diperlukan, dengan ekspektasi
yang benar soal batasannya.

## Kapan cPanel Masuk Akal

Cocok:
- situs statis, landing page, company profile,
- WordPress/PHP,
- app Node kecil bila host mendukung Node App.

Kurang cocok:
- Next.js SSR berat,
- worker/background job,
- kontrol Docker penuh,
- trafik tinggi dengan kebutuhan skala.

Kalau butuh kendali penuh, pakai VPS.

## A. Deploy Situs Statis

1. Build lokal:

```bash
npm run build
```

Untuk Next.js statis gunakan konfigurasi export sesuai versi yang dipakai.

2. Upload isi folder hasil build ke `public_html` lewat File Manager atau FTP.
3. Pastikan `index.html` berada di root domain.
4. Aktifkan SSL (AutoSSL/Let's Encrypt) dari cPanel.
5. Paksa HTTPS dari menu Domains.

Routing SPA sering butuh aturan rewrite di `.htaccess` agar refresh halaman
dalam tidak menghasilkan 404.

## B. Node.js App (Setup Node.js App)

Tersedia hanya bila host mengaktifkannya.

1. cPanel → Setup Node.js App → Create Application.
2. Node version sesuai project.
3. Application root: folder project.
4. Application URL: domain/subdomain.
5. Startup file: file entry app.
6. Tambahkan environment variables di panel, bukan di file publik.
7. Run NPM Install lalu start aplikasi.
8. Restart setiap kali kode berubah.

Batasan umum: memori kecil, proses dibatasi, tidak ada root, build berat bisa
gagal. Build di lokal, unggah artefak bila memungkinkan.

## C. Database

1. cPanel → MySQL/PostgreSQL Database Wizard.
2. Buat database dan user, beri privilege.
3. Simpan kredensial di environment, bukan di repo.
4. Backup rutin lewat panel dan simpan salinan di luar hosting.

## D. Email Domain

Buat akun email domain dari panel bila diperlukan, dan set SPF/DKIM agar email
tidak masuk spam. Untuk email transaksional volume besar, gunakan layanan email
khusus.

## Keamanan

- Password panel kuat dan unik.
- Aktifkan 2FA bila tersedia.
- Jangan menaruh `.env`, backup DB, atau file rahasia di `public_html`.
- Hapus file uji dan installer setelah selesai.
- Perbarui CMS/plugin bila memakai WordPress.

## Checklist

- [ ] Jenis aplikasi cocok untuk shared hosting
- [ ] SSL aktif dan HTTPS dipaksa
- [ ] Secret tidak berada di folder publik
- [ ] Database punya backup
- [ ] Prosedur update aplikasi jelas
