# H6 — Domain Terpisah: app + api via Cloudflare

> 🟠 LANJUTAN · Target waktu: 1–2 jam (propagasi DNS dapat lebih lama)

## Tujuan

Web tersedia di `app.domainmu.com` (Vercel) dan API di `api.domainmu.com` (Railway/Render) dengan sertifikat SSL valid.

## Prasyarat

- Domain dengan nameserver Cloudflare.
- URL `*.vercel.app` dan `*.up.railway.app` (atau Render) sudah sehat.

## Rencana DNS

| Subdomain | Tipe | Target | Proxy |
|---|---|---|---|
| `app` | CNAME | `cname.vercel-dns.com` | DNS only saat verifikasi, lalu Proxied |
| `api` | CNAME | target dari Railway/Render | DNS only saat verifikasi |

Salin target asli dari masing-masing platform; jangan menebak hostname.

## Langkah 1 — Daftarkan Domain di Vercel

1. Vercel → project web → **Settings → Domains**.
2. Masukkan `app.domainmu.com`.
3. Vercel menampilkan CNAME target.

**Hasil yang diharapkan:** domain berstatus menunggu konfigurasi DNS.

## Langkah 2 — Daftarkan Domain di Railway/Render

**Railway:** service API → **Settings → Networking → Custom Domain** → masukkan `api.domainmu.com` → salin target.

**Render:** service API → **Settings → Custom Domains** → tambah `api.domainmu.com` → salin target.

**Hasil yang diharapkan:** platform menunggu verifikasi DNS.

## Langkah 3 — Buat DNS di Cloudflare

Cloudflare → domain → **DNS → Records**:

**Record 1 — Web:**

| Field | Nilai |
|---|---|
| Type | CNAME |
| Name | `app` |
| Target | target dari Vercel |
| Proxy | DNS only (abu-abu) saat verifikasi awal |

**Record 2 — API:**

| Field | Nilai |
|---|---|
| Type | CNAME |
| Name | `api` |
| Target | target dari Railway/Render |
| Proxy | DNS only (abu-abu) saat verifikasi awal |

## Langkah 4 — Verifikasi DNS

```bash
nslookup app.domainmu.com
nslookup api.domainmu.com
```

**Output yang diharapkan:** masing-masing mengembalikan target platform.

Tunggu Vercel dan Railway/Render menandai domain aktif.

## Langkah 5 — Uji HTTPS

```bash
curl -I https://app.domainmu.com
curl -I https://api.domainmu.com/health
```

**Output yang diharapkan:** keduanya koneksi TLS valid dan respons bukan error.

Buka browser; periksa ikon koneksi aman dan hostname sertifikat.

## Langkah 6 — Aktifkan Proxy Cloudflare (Opsional, per Subdomain)

Setelah HTTPS langsung terverifikasi:

- `app`: boleh Proxied (awan oranye) untuk cache/WAF.
- `api`: evaluasi kebutuhan. Proxy Cloudflare menambah lapisan pada koneksi API; WebSocket dan long-polling mungkin butuh penyesuaian.

Cloudflare **SSL/TLS → Overview**: pilih **Full (strict)**.

Uji ulang setelah setiap perubahan proxy:

```bash
curl -i https://app.domainmu.com
curl -i https://api.domainmu.com/health
```

## Langkah 7 — Perbarui Environment Variable

Setelah domain aktif, perbarui reference:

| Platform | Variable | Nilai baru |
|---|---|---|
| Railway/Render API | `CORS_ORIGIN` | `https://app.domainmu.com` |
| Vercel web | `NEXT_PUBLIC_API_URL` | `https://api.domainmu.com` |

Redeploy kedua service. Uji form web → API melalui domain, periksa tidak ada CORS error.

Perbarui callback auth, webhook, email link, dan referensi URL lain.

## Langkah 8 — Redirect www dan root (Opsional)

Jika pengguna mengetik `domainmu.com` atau `www.domainmu.com`:

Cloudflare **Rules → Redirect Rules** atau **Page Rules**:
- `www.domainmu.com/*` → `https://app.domainmu.com/$1` (301)
- `domainmu.com/*` → `https://app.domainmu.com/$1` (301)

Cara redirect dapat berbeda berdasarkan fitur plan Cloudflare. Periksa dokumentasi resmi.

## Error Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Vercel "Domain not configured" | CNAME salah | Cocokkan target dari Vercel dashboard |
| Railway "Waiting for DNS" | CNAME salah atau masih diproksi | DNS only; cocokkan target |
| `ERR_TOO_MANY_REDIRECTS` | SSL Flexible | Ubah ke Full (strict) |
| CORS gagal setelah domain | `CORS_ORIGIN` masih URL lama | Update dan redeploy API |
| API tidak reachable lewat domain | Record DNS salah | Periksa `api` CNAME dan target |
| Sertifikat hostname salah | Domain didaftarkan ke service lain | Pindahkan custom domain ke service yang benar |

## Checklist

- [ ] `app.domainmu.com` → Vercel web, HTTPS valid
- [ ] `api.domainmu.com` → Railway/Render API, HTTPS valid
- [ ] Cloudflare SSL mode Full (strict)
- [ ] `CORS_ORIGIN` = `https://app.domainmu.com`
- [ ] `NEXT_PUBLIC_API_URL` = `https://api.domainmu.com`
- [ ] Form web → API domain produksi tanpa CORS error
- [ ] Redirect www/root aktif (bila dikonfigurasi)
- [ ] Auth callback dan webhook memakai domain baru

➡️ Lanjut ke **[07-maintenance.md](07-maintenance.md)**.
