# 06 — Domain, DNS, SSL, dan CORS

## Tujuan

Menghubungkan dua subdomain dengan HTTPS valid dan CORS ketat.

## Rekomendasi Domain

```text
app.domainmu.com  → frontend Cloudflare
api.domainmu.com  → backend Coolify
```

Root `domainmu.com` boleh diarahkan ke frontend atau landing page terpisah.

## 1. Frontend Cloudflare

Pages/Workers project → Custom Domains → tambahkan:

```text
app.domainmu.com
```

Kalau domain berada di akun Cloudflare yang sama, record DNS biasanya dibuat
otomatis. Pastikan status custom domain `Active`.

## 2. Backend Coolify

Di resource backend Coolify, tambahkan domain:

```text
https://api.domainmu.com
```

Di Cloudflare DNS buat record sesuai IP/hostname VPS:

```text
Type : A
Name : api
Value: IP_VPS_KAMU
Proxy: DNS only saat issuance SSL pertama
TTL  : Auto
```

Setelah sertifikat Coolify/Let's Encrypt aktif, proxy dapat dinyalakan bila
konfigurasi mendukung.

## 3. SSL Mode

Cloudflare → SSL/TLS → Overview:

```text
Encryption mode: Full (Strict)
```

Jangan gunakan Flexible; kombinasi Flexible + force HTTPS di origin menyebabkan
redirect loop dan koneksi Cloudflare→origin tidak terenkripsi.

## 4. CORS Backend

Production:

```text
FRONTEND_ORIGIN=https://app.domainmu.com
```

Staging:

```text
FRONTEND_ORIGIN=https://staging.domainmu.com
```

Kalau perlu beberapa origin, gunakan allowlist eksplisit. Jangan `*` ketika
mengirim cookie/credential.

## 5. Cookie/Auth

Untuk cookie lintas subdomain:

```text
Secure   = true
HttpOnly = true
SameSite = Lax/None sesuai alur
Domain   = .domainmu.com (hanya kalau benar-benar perlu berbagi)
```

`SameSite=None` wajib `Secure`. Prefer token/session yang tidak diekspos ke
JavaScript.

## Verifikasi

```bash
curl -I https://app.domainmu.com
curl -I https://api.domainmu.com/up
curl -I http://api.domainmu.com/up
```

Expected:
- frontend HTTPS → 200,
- backend HTTPS `/up` → 200,
- HTTP → 301/308 ke HTTPS.

Cek CORS dari browser frontend, bukan hanya curl.

## Checklist

- [ ] `app.domainmu.com` aktif di Cloudflare
- [ ] `api.domainmu.com` mengarah ke VPS
- [ ] SSL Cloudflare Full (Strict)
- [ ] Sertifikat origin backend valid
- [ ] HTTP redirect ke HTTPS
- [ ] CORS hanya mengizinkan frontend resmi
- [ ] Cookie auth memakai Secure + HttpOnly
