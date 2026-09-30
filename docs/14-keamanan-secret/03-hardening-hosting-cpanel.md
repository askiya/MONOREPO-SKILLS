# 03 — Hardening Hosting cPanel

## Tujuan

Mengurangi risiko akun hosting, file, database, dan email dibajak.

## Akun Panel

- Password unik, minimal 16 karakter, simpan di password manager.
- Aktifkan 2FA.
- Jangan bagikan login utama ke developer; buat akun FTP/email/DB terpisah.
- Hapus akun lama saat kerja selesai.
- Waspadai email phishing "hosting expired"; buka panel dari bookmark resmi.

## File Permission

Umum:

```text
Folder: 755
File: 644
File secret: 600 atau 640
```

Jangan pakai `777`. Itu membuat semua proses bisa menulis file.

## Lindungi File Sensitif (.htaccess)

```apache
Options -Indexes

<FilesMatch "(^\.env|composer\.(json|lock)|package(-lock)?\.json|.*\.(sql|log|bak|ini))$">
  Require all denied
</FilesMatch>

<FilesMatch "^\.git">
  Require all denied
</FilesMatch>
```

Tetap taruh credential di luar `public_html`; aturan ini lapisan tambahan.

## Nonaktifkan Directory Listing

Pastikan `.htaccess` berisi:

```apache
Options -Indexes
```

Akses folder tanpa index harus menghasilkan 403, bukan daftar file.

## PHP

- Pilih versi PHP yang masih didukung.
- Nonaktifkan `display_errors` di produksi.
- Aktifkan logging server-side.
- Hapus plugin/theme/CMS yang tidak dipakai.
- Update WordPress/core/plugin/theme rutin.
- Jangan install plugin nulled/bajakan.

## Database

- Satu user DB per aplikasi.
- Hak hanya database aplikasi tersebut.
- Remote MySQL allowlist IP tertentu, bukan `%`.
- Jangan pakai user DB yang sama untuk staging dan produksi.
- Backup terenkripsi di luar hosting.

## FTP dan File Manager

- Pakai SFTP bila tersedia.
- Jangan simpan password FTP di browser komputer publik.
- Hapus file ZIP installer/backup setelah extract.
- Cari file asing atau perubahan waktu yang tidak dikenal.

## Email Domain

- SPF, DKIM, DMARC aktif.
- Password email unik.
- Batasi forwarder; penyerang sering membuat forwarder tersembunyi.
- Monitor spam/outbound quota.

## Backup

Backup sebelum update besar. Simpan minimal:
- file website,
- database,
- konfigurasi DNS/env (tanpa dipublikasi),
- daftar cron job.

Uji restore ke subdomain staging. Backup tanpa restore test belum terbukti.

## Jika Diretas

1. Aktifkan maintenance atau isolasi site.
2. Simpan salinan bukti/log sebelum membersihkan.
3. Ganti password cPanel, FTP, DB, email; rotate API keys.
4. Pulihkan dari backup bersih, bukan sekadar hapus file terlihat aneh.
5. Patch akar masalah (plugin rentan, password bocor, permission).
6. Scan seluruh akun; shared hosting bisa punya banyak site.
7. Cek cron, email forwarder, admin account, `.htaccess`, startup file.
8. Hubungi provider bila akses sistem/log dibutuhkan.

## Checklist

- [ ] 2FA dan password unik
- [ ] Permission tanpa 777
- [ ] Directory listing mati
- [ ] Secret/backup/log diblokir dan di luar web root
- [ ] PHP/CMS/plugin up-to-date
- [ ] DB remote tidak terbuka umum
- [ ] SPF/DKIM/DMARC aktif
- [ ] Backup offsite + restore test
