# Latihan Praktik: Sesi 6 — Hosting cPanel + Domain

## Target Praktik
Situs online di domain sendiri lewat shared hosting cPanel, dengan HTTPS aktif.

## Estimasi Waktu
120 menit (belum termasuk propagasi DNS).

## Yang Harus Disiapkan
- hosting cPanel sudah dibeli (`docs/11-hosting-cpanel/01-beli-hosting.md`),
- domain sudah dibeli,
- hasil build siap upload (folder static) atau app Node kecil.

## Langkah

### 1. Login cPanel

1. Buka `https://domainmu.com:2083` atau link dari email hosting.
2. Login dengan username + password dari email.
3. **Segera** ganti password: **Preferences** → **Password & Security**.
4. Aktifkan 2FA kalau tersedia.

**Hasil yang benar:** dashboard cPanel tampil dengan menu Files, Databases, Domains.

### 2. Arahkan DNS

Detail: `docs/11-hosting-cpanel/02-dns-record.md`.

Kalau domain di registrar lain, set A record:
```text
Type : A
Name : @
Value: [IP server hosting dari email/cPanel]
TTL  : Auto / 3600
```
Dan:
```text
Type : CNAME
Name : www
Value: domainmu.com
```

Verifikasi:
```bash
nslookup domainmu.com 8.8.8.8
```

**Hasil yang benar:** IP yang muncul = IP hosting.
**Gagal kalau:** NXDOMAIN atau IP lama → tunggu propagasi (15 menit – 24 jam).

### 3. Upload File

Detail: `docs/11-hosting-cpanel/04-upload-file.md`.

1. **Files** → **File Manager** → masuk `public_html`.
2. Hapus file default (`default.html`, dll).
3. Upload hasil build sebagai `.zip`.
4. Klik kanan → **Extract**.
5. Pastikan `index.html` berada **langsung** di `public_html`, bukan di subfolder.

**Hasil yang benar:** buka `http://domainmu.com` → situs tampil.
**Gagal kalau:**
- 404 → file di subfolder, pindahkan
- 403 → permission salah (folder 755, file 644)

### 4. Aktifkan HTTPS

Detail: `docs/11-hosting-cpanel/03-ssl-https.md`.

1. **Security** → **SSL/TLS Status**.
2. Centang domain → **Run AutoSSL**.
3. Tunggu sampai status hijau.

Verifikasi:
```bash
curl -I https://domainmu.com
curl -I http://domainmu.com
```

**Hasil yang benar:**
```text
https → HTTP/2 200
http  → HTTP/1.1 301 Moved Permanently
        Location: https://domainmu.com/
```

**Gagal kalau:** AutoSSL gagal → DNS belum mengarah, atau Cloudflare proxy ON.
Set DNS only sementara, jalankan ulang.

### 5. Database (kalau perlu)

Detail: `docs/11-hosting-cpanel/06-database-dan-credential.md`.

1. **Databases** → **MySQL Database Wizard**.
2. Buat DB → buat user → beri ALL PRIVILEGES.
3. Catat: nama DB dan user punya prefix `usernamecpanel_`.
4. Simpan credential di environment/file config **di luar** `public_html`.

**JANGAN** taruh credential di file yang bisa diakses dari browser.

### 6. Node.js (kalau hosting mendukung)

Detail: `docs/11-hosting-cpanel/05-nodejs-di-cpanel.md`.

**Software** → **Setup Node.js App**. Kalau menu tidak ada → hosting tidak
mendukung Node. Gunakan static build atau pindah ke VPS/Vercel.

## Cara Verifikasi
- [ ] `nslookup` → IP hosting benar
- [ ] `https://domainmu.com` → 200, gembok hijau
- [ ] `http://domainmu.com` → 301 ke https
- [ ] `https://domainmu.com/.env` → 403/404 (TIDAK boleh tampil isi)
- [ ] Password cPanel sudah diganti
- [ ] 2FA aktif

## Error yang Sering Terjadi

| Gejala | Penyebab | Solusi |
|---|---|---|
| domain tidak terbuka | DNS belum propagasi | tunggu, cek nslookup |
| 403 Forbidden | permission / tidak ada index | 755 folder, 644 file |
| 404 semua halaman | Document Root salah | cek lokasi file |
| 500 error | .htaccess / syntax | baca `error_log` di File Manager |
| Too many redirects | Cloudflare Flexible | ganti ke Full (Strict) |
| AutoSSL gagal | Cloudflare proxy ON | set DNS only, ulang AutoSSL |
| `.env` terbaca publik | file di `public_html` | pindahkan + blokir via `.htaccess` |

## Tugas Mandiri

Buat subdomain `staging.domainmu.com`:
1. **Domains** → **Create A New Domain**
2. Document Root: `/public_html/staging`
3. Upload versi uji ke situ
4. Jalankan AutoSSL untuk subdomain

## Bukti Kelulusan

Kirim ke mentor:
1. URL domain produksi,
2. screenshot gembok HTTPS,
3. output `curl -I https://domainmu.com`,
4. output `curl -I http://domainmu.com` (harus 301),
5. screenshot `https://domainmu.com/.env` menampilkan 403/404,
6. URL subdomain staging.
