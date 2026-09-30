# 01 — Beli dan Mulai Hosting cPanel

## Tujuan

Punya akses cPanel hosting, paham panel, dan siap upload.

## Provider Indonesia yang Populer

| Provider | URL | Catatan |
|---|---|---|
| Niagahoster | niagahoster.co.id | harga kompetitif, panel Bahasa Indonesia |
| Rumahweb | rumahweb.com | cPanel stabil |
| IDCloudHost | idcloudhost.com | pilihan server SG/ID |
| Domainesia | domainesia.com | cPanel + LiteSpeed |
| Dewaweb | dewaweb.com | ISO 27001 |
| Jagoan Hosting | jagoanhosting.com | murah untuk pemula |
| Hostinger ID | hostinger.co.id | hPanel (bukan cPanel) |

## Cara Beli

1. Buka website provider, pilih paket **shared hosting** (mulai dari ~20rb/bulan).
2. Pilih durasi — bulanan lebih mahal per-bulan; tahunan lebih hemat.
3. Kalau belum punya domain, beli sekaligus. Kalau sudah punya, pilih "domain
   sudah ada" dan arahkan nanti.
4. Checkout + bayar.
5. Cek email — dapat: URL panel, username, password temporer.

## Login Pertama Kali

1. Buka URL cPanel dari email (biasanya `https://namaserver:2083` atau
   `https://domainmu.com/cpanel`).
2. Login dengan username + password dari email.
3. **Ganti password segera**: cPanel → Security → Password & Security.
4. Aktifkan **Two-Factor Authentication** kalau tersedia.

## Orientasi Panel cPanel

Area penting:

| Menu | Fungsi |
|---|---|
| **File Manager** | upload/edit file tanpa FTP |
| **MySQL/PostgreSQL Database Wizard** | buat database + user |
| **Domains** / **Addon Domains** | domain & subdomain |
| **SSL/TLS** | sertifikat HTTPS |
| **Email Accounts** | email domain |
| **Setup Node.js App** | (kalau tersedia) jalankan Node |
| **PHP Version** / **MultiPHP** | ubah versi PHP |
| **Softaculous** | installer WordPress dll |
| **Backup** | backup manual |
| **Zone Editor** / **DNS Zone** | kelola DNS record |
| **Error Pages** | custom 404/500 |
| **Cron Jobs** | tugas terjadwal |

## `public_html` = Root Website

Semua file yang bisa diakses pengunjung ada di `public_html/`:

```
public_html/
├── index.html    ← halaman utama domain.com
├── css/
├── js/
├── images/
└── .htaccess     ← konfigurasi Apache
```

- **Jangan** taruh `.env`, backup DB, file konfigurasi rahasia di sini.
- **Jangan** taruh seluruh project Node.js di sini kalau pakai API — folder
  project Node.js biasanya di luar `public_html`.

## Checklist

- [ ] Hosting aktif dan bisa login cPanel
- [ ] Password diganti dari default
- [ ] 2FA aktif (kalau ada)
- [ ] Tahu letak File Manager, Database, Domains, SSL
- [ ] Paham `public_html` = folder publik
