# 06 — Domain & SSL: Cloudflare DNS + Vercel (Paket D)

## Tujuan

Custom domain mengarah ke Vercel melalui Cloudflare DNS. HTTPS aktif otomatis.

---

## Sebelum Mulai

- [ ] Deploy Vercel berhasil dan URL `.vercel.app` bisa diakses ([05-deploy.md](05-deploy.md))
- [ ] Punya domain aktif
- [ ] Punya akun Cloudflare

---

## Referensi Utama

> Detail DNS lengkap: [`../../docs/12-domain-dns-cloudflare/01-domain-dns.md`](../../docs/12-domain-dns-cloudflare/01-domain-dns.md)

---

## Urutan Aman

> ⚠️ Ikuti urutan. Jangan hapus domain lama sebelum domain baru terverifikasi.

1. Tambah domain di Vercel
2. Catat DNS record yang diminta Vercel
3. Tambah DNS record di Cloudflare (DNS only dulu)
4. Tunggu Vercel verifikasi
5. Test HTTPS
6. Baru aktifkan Cloudflare proxy kalau memang dibutuhkan
7. Update Supabase Site URL + Redirect URLs

---

## Langkah

### 1. Tambah Custom Domain di Vercel

1. Vercel Dashboard → pilih project
2. **Settings** → **Domains**
3. Masukkan domain, contoh: `domainmu.com`
4. Klik **Add**
5. Tambahkan juga `www.domainmu.com`

Vercel akan menampilkan DNS record yang harus dibuat. Biasanya:

| Type | Name | Value |
|---|---|---|
| A | `@` | `76.76.21.21` (cek nilai terbaru di Vercel) |
| CNAME | `www` | `cname.vercel-dns.com` (cek nilai terbaru di Vercel) |

> ⚠️ Nilai DNS bisa berubah. **Ikuti yang ditampilkan dashboard Vercel**,
> bukan copy mentah dari tabel di atas.

### 2. Setup Domain di Cloudflare

Kalau domain belum di Cloudflare:

1. Cloudflare Dashboard → **Add a Site**
2. Masukkan domain → pilih Free plan
3. Cloudflare memberi 2 nameserver
4. Buka panel registrar domain → ganti nameserver ke nameserver Cloudflare
5. Tunggu nameserver aktif (beberapa menit sampai 24 jam)

### 3. Tambah DNS Records di Cloudflare

Cloudflare → domain → **DNS** → **Records**:

| Type | Name | Target | Proxy status |
|---|---|---|---|
| A | `@` | nilai dari Vercel | **DNS only** (awan abu-abu) |
| CNAME | `www` | nilai dari Vercel | **DNS only** (awan abu-abu) |

> Gunakan **DNS only** saat verifikasi awal. Proxy Cloudflare bisa menghambat
> verifikasi domain Vercel.

Hapus record A/CNAME lama yang bentrok (record dengan Name `@` atau `www`).

### 4. Tunggu Verifikasi Vercel

Kembali ke Vercel → Settings → Domains.

**Berhasil kalau:** status domain = **Valid Configuration** dengan centang hijau.

Cek via terminal:

```bash
nslookup domainmu.com
nslookup www.domainmu.com
```

Atau:

```bash
dig domainmu.com +short
dig www.domainmu.com +short
```

### 5. SSL Otomatis

Vercel otomatis menerbitkan sertifikat SSL setelah DNS valid.
Tidak perlu Certbot, AutoSSL, atau upload certificate manual.

Tunggu beberapa menit, lalu:

```bash
curl -I https://domainmu.com
curl -I http://domainmu.com
```

**Output yang benar:**
- HTTPS: `HTTP/2 200`
- HTTP: `301` atau `308` redirect ke HTTPS

### 6. Pilih Domain Utama

Di Vercel → Settings → Domains:
- Pilih domain utama: `domainmu.com` atau `www.domainmu.com`
- Set redirect dari domain yang lain ke domain utama

Contoh:
```text
www.domainmu.com → 308 redirect → domainmu.com
```

### 7. Update Supabase Auth URLs

Di Supabase → **Authentication** → **URL Configuration**:

| Setting | Nilai baru |
|---|---|
| Site URL | `https://domainmu.com` |
| Redirect URLs | `https://domainmu.com/**` |

**Jangan hapus** URL Vercel preview — tetap dibutuhkan untuk test PR.

### 8. Update OAuth App (Kalau Perlu)

Untuk Supabase Auth, OAuth callback tetap:
```text
https://<PROJECT_REF>.supabase.co/auth/v1/callback
```

Tidak perlu ganti ke custom domain, kecuali kamu setup Supabase custom domain
(fitur berbayar terpisah).

### 9. Cloudflare Proxy (Opsional)

Untuk Vercel, **DNS only** sudah cukup. Vercel punya CDN + DDoS protection sendiri.

Kalau mau aktifkan proxy Cloudflare (awan oranye):
1. SSL/TLS mode → **Full (strict)**
2. Aktifkan proxy pada record
3. Test semua route + auth setelah aktif

> **Rekomendasi:** tetap DNS only. Double CDN (Cloudflare proxy + Vercel CDN)
> menambah kompleksitas cache tanpa manfaat besar untuk MVP.

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Vercel "Invalid Configuration" | DNS record salah atau belum propagasi | Copy ulang record dari dashboard Vercel, tunggu |
| SSL certificate pending lama | DNS belum valid | Pastikan proxy Cloudflare = DNS only |
| Login redirect ke `.vercel.app` | Supabase Site URL belum diupdate | Ganti Site URL ke custom domain |
| OAuth selesai tapi balik ke localhost | Redirect URL masih localhost | Update Supabase URL Configuration |
| www jalan tapi root tidak | A record `@` belum ada | Tambahkan A record sesuai Vercel |
| Root jalan tapi www tidak | CNAME `www` belum ada | Tambahkan CNAME sesuai Vercel |
| Loop redirect | Cloudflare SSL = Flexible | Ganti ke Full (strict), atau matikan proxy |

---

## Checklist

- [ ] Domain ditambahkan di Vercel (root + www)
- [ ] DNS records dibuat di Cloudflare sesuai nilai dashboard Vercel
- [ ] Proxy Cloudflare = DNS only saat verifikasi
- [ ] Vercel menunjukkan "Valid Configuration"
- [ ] `https://domainmu.com` bisa diakses (HTTP 200)
- [ ] HTTP redirect ke HTTPS (301/308)
- [ ] www redirect ke domain utama
- [ ] Supabase Site URL = custom domain
- [ ] Supabase Redirect URLs mencakup custom domain
- [ ] Register/login/OAuth berfungsi via custom domain
