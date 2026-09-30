# 06 — Domain & SSL (Cloudflare DNS)

## Tujuan

Domain custom mengarah ke aplikasi di Vercel, HTTPS aktif dengan sertifikat
valid, dan SSL mode Full (Strict) di Cloudflare.

## Sebelum Mulai

- Deploy di Vercel sudah berhasil ([`05-deploy.md`](05-deploy.md))
- Sudah punya domain (beli di Niagahoster, Cloudflare Registrar, dll)
- Akun Cloudflare sudah ada

Referensi utama:
[`../../docs/12-domain-dns-cloudflare/01-domain-dns.md`](../../docs/12-domain-dns-cloudflare/01-domain-dns.md)

---

## Langkah 1 — Tambahkan domain ke Cloudflare

> Kalau domain sudah di Cloudflare, langsung ke Langkah 2.

1. Login [dash.cloudflare.com](https://dash.cloudflare.com)
2. **Add a Site** → masukkan domain (misal: `namaproject.com`)
3. Pilih plan **Free**
4. Cloudflare menampilkan nameservers (misal: `ada.ns.cloudflare.com`, `bob.ns.cloudflare.com`)
5. Buka dashboard registrar domain kamu (Niagahoster, Namecheap, dll)
6. Ganti nameserver ke yang diberikan Cloudflare
7. Tunggu propagasi (biasanya 1–24 jam, sering kurang dari 1 jam)

Cek status propagasi:
```bash
dig NS namaproject.com +short
```

**Expected output:** nameserver Cloudflare muncul.

---

## Langkah 2 — Tambahkan domain di Vercel

1. Buka project di dashboard Vercel
2. **Settings** → **Domains**
3. Tambahkan domain: `namaproject.com`
4. Vercel menampilkan instruksi DNS. Biasanya:
   - `CNAME` record `@` → `cname.vercel-dns.com`
   - Atau `A` record `@` → `76.76.21.21`

### Untuk subdomain `www`:

- `CNAME` record `www` → `cname.vercel-dns.com`

Vercel akan menampilkan centang hijau kalau DNS sudah mengarah.

---

## Langkah 3 — Set DNS record di Cloudflare

1. Di Cloudflare dashboard → domain kamu → **DNS** → **Records**
2. Tambahkan record:

**Opsi A — CNAME (direkomendasikan):**

| Type | Name | Content | Proxy | TTL |
|---|---|---|---|---|
| CNAME | `@` | `cname.vercel-dns.com` | **OFF** (DNS only) | Auto |
| CNAME | `www` | `cname.vercel-dns.com` | **OFF** (DNS only) | Auto |

**Opsi B — A record:**

| Type | Name | Content | Proxy | TTL |
|---|---|---|---|---|
| A | `@` | `76.76.21.21` | **OFF** (DNS only) | Auto |
| CNAME | `www` | `cname.vercel-dns.com` | **OFF** (DNS only) | Auto |

> ⚠️ **Proxy (awan oranye) harus OFF** untuk Vercel. Kalau ON, SSL akan
> bentrok antara Cloudflare dan Vercel. Vercel menangani SSL sendiri.

Hapus record lama yang mengarah ke tempat lain supaya tidak konflik.

---

## Langkah 4 — Konfigurasi SSL di Cloudflare

1. Cloudflare dashboard → domain → **SSL/TLS** → **Overview**
2. Set mode: **Full (Strict)**

> Mode "Flexible" → **JANGAN dipakai**. Menyebabkan redirect loop atau
> koneksi tidak terenkripsi antara Cloudflare dan Vercel.

---

## Langkah 5 — Update NEXTAUTH_URL

Di Vercel dashboard:
1. **Settings** → **Environment Variables**
2. Ubah `NEXTAUTH_URL` dari `https://nama-project.vercel.app` ke `https://namaproject.com`
3. **Redeploy** supaya perubahan env var berlaku:
   - Deployments → klik "..." di deployment teratas → Redeploy

---

## Langkah 6 — Verifikasi

### DNS sudah mengarah

```bash
dig namaproject.com +short
```
Harus menampilkan IP Vercel (kalau A record) atau CNAME chain.

### HTTPS aktif

```bash
curl -I https://namaproject.com
```
**Expected output:** `HTTP/2 200` dan `strict-transport-security` header.

### Redirect www

```bash
curl -I https://www.namaproject.com
```
Harus redirect ke `namaproject.com` (atau sebaliknya, tergantung setting Vercel).

### Cek sertifikat

Buka di browser → klik gembok di address bar → lihat sertifikat.

- Issued to: `namaproject.com`
- Issued by: Let's Encrypt atau Vercel
- Valid: belum expired

### Tes fungsionalitas

Login dari URL custom domain → harus berfungsi sama seperti dari `.vercel.app`.

---

## Kesalahan Umum

| Gejala | Penyebab | Solusi |
|---|---|---|
| `ERR_TOO_MANY_REDIRECTS` | SSL mode Flexible di Cloudflare | Ubah ke Full (Strict) |
| `DNS_PROBE_FINISHED_NXDOMAIN` | Nameserver belum pindah | Tunggu propagasi, cek `dig NS domain` |
| Vercel bilang "Invalid Configuration" | Proxy Cloudflare (awan oranye) ON | Matikan proxy, set DNS Only |
| Login gagal setelah pakai domain | `NEXTAUTH_URL` masih URL lama | Update env var dan redeploy |
| Sertifikat not trusted | DNS mengarah tapi SSL belum provisioned | Tunggu 5–10 menit, Vercel provision otomatis |

---

## Checklist

- [ ] Domain ada di Cloudflare (nameserver sudah pindah)
- [ ] Domain ditambahkan di Vercel dashboard
- [ ] DNS record CNAME/A diset, **proxy OFF**
- [ ] SSL/TLS mode di Cloudflare: **Full (Strict)**
- [ ] `NEXTAUTH_URL` diupdate ke domain custom
- [ ] Redeploy setelah update env var
- [ ] `curl -I https://domain` → HTTP/2 200
- [ ] Browser: gembok hijau, sertifikat valid
- [ ] Login berfungsi via domain custom
- [ ] `www` redirect berfungsi
