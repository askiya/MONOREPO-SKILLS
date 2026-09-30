# 03 — SSL / HTTPS di cPanel

## Tujuan

Website diakses lewat `https://` dengan gembok hijau.

## Opsi SSL

### A. AutoSSL (gratis, paling mudah)

Kebanyakan cPanel sudah menyalakan AutoSSL (Let's Encrypt / Sectigo).

1. cPanel → **SSL/TLS Status**.
2. Lihat domain — kalau statusnya ✅ "AutoSSL Domain Validated", sudah aktif.
3. Kalau belum, klik **Run AutoSSL**. Tunggu beberapa menit.
4. AutoSSL memperpanjang otomatis setiap 60–90 hari.

**Syarat AutoSSL berhasil:**
- Domain sudah mengarah ke hosting (DNS benar).
- Tidak ada `.htaccess` yang memblokir validasi.
- Kalau pakai Cloudflare proxy: mode SSL **Full (Strict)**.

### B. Let's Encrypt Manual (via cPanel plugin)

Beberapa hosting punya plugin **Let's Encrypt for cPanel**:

1. cPanel → cari "Let's Encrypt".
2. Pilih domain → **Issue**.
3. Centang `www` juga.
4. Done. Auto-renew biasanya aktif.

### C. SSL Berbayar

Untuk EV/OV certificate (jarang dibutuhkan pemula):

1. Generate CSR di cPanel → **SSL/TLS** → **Generate CSR**.
2. Beli sertifikat di provider SSL.
3. Validasi domain (email/DNS/HTTP).
4. Install di cPanel → **SSL/TLS** → **Install Certificate**.

### D. Cloudflare SSL (paling fleksibel)

Kalau DNS lewat Cloudflare:

1. Cloudflare → SSL/TLS → **Full (Strict)**.
2. Hosting tetap perlu SSL origin (AutoSSL sudah cukup).
3. Cloudflare menangani sertifikat publik otomatis.

**Jangan pakai Flexible** di produksi — koneksi Cloudflare ke origin jadi HTTP.

## Paksa HTTPS

### Via cPanel

cPanel → **Domains** → centang **Force HTTPS Redirect**.

### Via .htaccess

```apache
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

### Via Cloudflare

Cloudflare → SSL/TLS → Edge Certificates → **Always Use HTTPS** → On.

## Verifikasi

```bash
curl -I https://domainmu.com
# Harus: HTTP/2 200, header strict-transport-security (kalau HSTS aktif)

curl -I http://domainmu.com
# Harus: 301 redirect ke https://
```

Buka browser → klik gembok → pastikan sertifikat valid dan belum expired.

## Checklist

- [ ] SSL aktif (AutoSSL / Let's Encrypt / Cloudflare)
- [ ] HTTPS dipaksa (redirect 301)
- [ ] `http://` redirect ke `https://`
- [ ] Tidak ada mixed content warning
- [ ] Auto-renew aktif
