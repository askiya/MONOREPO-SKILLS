# 02 — Hardening Aplikasi Web

## Tujuan

Menutup risiko web umum sebelum produksi tanpa security theater.

## 1. Security Headers

Minimum yang relevan:

```http
Content-Security-Policy: default-src 'self'; object-src 'none'; frame-ancestors 'none'
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

CSP harus disesuaikan untuk CDN, analytics, payment, dan image host. Jangan
menyalin CSP ketat lalu aplikasi rusak. Uji `Content-Security-Policy-Report-Only`
dulu. Aktifkan HSTS setelah semua subdomain siap HTTPS.

## 2. Validasi dan Output Encoding

- Validasi semua body/query/path/upload di server.
- Gunakan query parameterized/ORM; jangan gabung SQL string.
- React meng-escape teks secara default; hindari `dangerouslySetInnerHTML`.
- Sanitasi HTML user dengan library matang bila HTML memang diperlukan.
- Nama file upload dibuat ulang server; jangan percaya nama user.

## 3. Auth dan Session

- Password: Argon2id atau bcrypt dengan cost wajar.
- Cookie: `HttpOnly`, `Secure`, `SameSite`, expiry.
- Rotasi session setelah login dan perubahan privilege.
- Logout mencabut session server bila memakai session store.
- MFA wajib untuk admin bila tersedia.
- Rate limit per IP + identifier untuk login/reset.

## 4. CSRF dan CORS

Cookie auth membutuhkan perlindungan CSRF untuk request mutasi: token CSRF,
SameSite, dan verifikasi Origin sesuai arsitektur.

CORS:
- allow hanya origin yang perlu,
- jangan gabungkan `Access-Control-Allow-Origin: *` dengan credentials,
- batasi method dan header.

## 5. Upload File

- Batasi ukuran.
- Allowlist MIME dan ekstensi; cek magic bytes bila penting.
- Simpan di object storage, bukan direktori executable.
- Gunakan nama acak.
- Scan malware untuk alur berisiko.
- Jangan render SVG/HTML user tanpa sanitasi.

## 6. SSRF dan URL Fetch

Jika server mengambil URL dari user:
- hanya `http/https`,
- blok IP private, loopback, link-local, metadata cloud,
- validasi ulang setelah redirect dan DNS resolution,
- batas response size + timeout,
- jangan meneruskan credential internal.

## 7. Dependency

```bash
npm audit
npm outdated
```

Nilai apakah vulnerability mencapai kode produksi. Jangan melakukan upgrade
major massal hanya demi membuat angka audit nol.

## 8. Error dan Logging

- Response publik: pesan generik + request ID.
- Log server: stack trace, request ID, actor, event; sensor secret/PII.
- Jangan log authorization header, cookie, password, payment credential.
- Retensi log dibatasi dan akses least privilege.

## Checklist

- [ ] Header keamanan diuji tanpa merusak app
- [ ] Input divalidasi dan output di-encode
- [ ] Cookie/session aman
- [ ] CSRF/CORS sesuai arsitektur
- [ ] Upload aman
- [ ] URL fetch tahan SSRF bila ada
- [ ] Log tidak menyimpan secret/PII sensitif
