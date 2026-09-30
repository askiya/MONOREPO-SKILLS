# 06 — Domain, DNS & SSL (Paket C)

## Tujuan

Domain mengarah ke hosting cPanel, AutoSSL aktif, semua traffic HTTPS.

---

## Sebelum Mulai

- [ ] Website sudah bisa diakses di IP atau subdomain default hosting ([05-deploy.md](05-deploy.md))
- [ ] Punya akses ke panel registrar domain (tempat beli domain)

---

## Referensi Utama

> Detail lengkap ada di:
> - [`../../docs/11-hosting-cpanel/02-dns-record.md`](../../docs/11-hosting-cpanel/02-dns-record.md)
> - [`../../docs/11-hosting-cpanel/03-ssl-https.md`](../../docs/11-hosting-cpanel/03-ssl-https.md)
> - [`../../docs/12-domain-dns-cloudflare/01-domain-dns.md`](../../docs/12-domain-dns-cloudflare/01-domain-dns.md)

---

## Opsi 1: DNS Langsung di cPanel (Paling Simpel)

### 1. Arahkan Nameserver Domain ke Hosting

Di panel registrar domain (Niagahoster, Namecheap, GoDaddy, dll):
1. Cari menu **Nameserver** atau **DNS**
2. Ganti nameserver ke nameserver hosting kamu (dapat dari provider hosting)

Contoh:
```text
ns1.hostingmu.com
ns2.hostingmu.com
```

> Propagasi DNS memakan waktu 1-48 jam. Biasanya 1-4 jam.

### 2. Verifikasi Domain Terhubung

```bash
# Cek apakah domain sudah mengarah ke server hosting
nslookup domainmu.com

# Atau
dig domainmu.com +short
```

**Berhasil kalau:** IP yang muncul = IP server hosting kamu.

### 3. Aktifkan AutoSSL

1. cPanel → **SSL/TLS Status**
2. Klik **Run AutoSSL**
3. Tunggu beberapa menit
4. Status berubah ke ✅ hijau

**Berhasil kalau:** `https://domainmu.com` bisa diakses dengan gembok hijau.

> AutoSSL bisa gagal kalau DNS belum propagasi. Tunggu DNS selesai, baru jalankan AutoSSL.

### 4. Force HTTPS

Pastikan `.htaccess` sudah ada rule redirect (dari [05-deploy.md](05-deploy.md)):

```apache
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

Atau aktifkan via cPanel:
1. cPanel → **Domains** → domain kamu → aktifkan **Force HTTPS**

> Jangan pakai dua-duanya sekaligus (`.htaccess` + Force HTTPS cPanel) —
> bisa menyebabkan redirect loop.

---

## Opsi 2: DNS via Cloudflare (Disarankan)

Kenapa Cloudflare:
- CDN gratis (website lebih cepat)
- Protection DDoS
- DNS propagasi lebih cepat
- Bisa pakai Cloudflare SSL (flexible atau full)

### 1. Daftar Cloudflare

1. Buka [cloudflare.com](https://cloudflare.com)
2. Add Site → masukkan domain
3. Pilih plan **Free**
4. Cloudflare scan DNS records existing

### 2. Ganti Nameserver ke Cloudflare

Di panel registrar domain:
```text
ns1.cloudflare.com (contoh — nama asli dari dashboard Cloudflare)
ns2.cloudflare.com
```

> Nama nameserver Cloudflare berbeda tiap akun. Copy dari dashboard Cloudflare, bukan dari sini.

### 3. Tambah DNS Records

Di Cloudflare DNS:

| Type | Name | Content | Proxy |
|---|---|---|---|
| A | `@` | `IP_SERVER_HOSTING` | ☁️ Proxied |
| A | `www` | `IP_SERVER_HOSTING` | ☁️ Proxied |

IP server hosting bisa dilihat di cPanel → sidebar kanan → **Shared IP Address**.

### 4. Setting SSL di Cloudflare

Cloudflare → **SSL/TLS**:
- Pilih **Full** (bukan Flexible, bukan Full Strict)
- Full = Cloudflare → HTTPS → server hosting (pakai AutoSSL dari cPanel)

> ⚠️ **Jangan pakai Flexible** kalau server sudah punya SSL.
> Flexible berarti Cloudflare → HTTP → server, padahal server mau HTTPS.
> Ini menyebabkan redirect loop.

### 5. Aktifkan Always HTTPS

Cloudflare → **SSL/TLS** → **Edge Certificates**:
- Aktifkan **Always Use HTTPS**
- Aktifkan **Automatic HTTPS Rewrites**

### 6. Verifikasi

```bash
curl -I https://domainmu.com
```

**Output yang benar:**
```text
HTTP/2 200
server: cloudflare
...
```

---

## Subdomain (Opsional)

Kalau mau subdomain (misal `app.domainmu.com`):

1. cPanel → **Subdomains** → buat subdomain
2. Di DNS (cPanel atau Cloudflare) → tambah A record:
   - Type: A
   - Name: `app`
   - Content: IP server hosting

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| "DNS_PROBE_FINISHED_NXDOMAIN" | DNS belum propagasi | Tunggu 1-4 jam, cek `nslookup` |
| AutoSSL gagal | DNS belum mengarah ke server | Pastikan DNS propagasi selesai dulu |
| "Too many redirects" | SSL Cloudflare = Flexible + server force HTTPS | Ganti ke Full di Cloudflare |
| "ERR_SSL_VERSION_OR_CIPHER_MISMATCH" | SSL belum aktif di server | Jalankan AutoSSL di cPanel |
| www tidak bisa diakses | Tidak ada DNS record untuk www | Tambah A record `www` |
| Mixed content warning | Ada asset HTTP di halaman HTTPS | Ganti semua URL ke HTTPS atau relative |

---

## Urutan Aman

> ⚠️ **Ikuti urutan ini agar tidak terkunci dari website:**

1. ✅ Pastikan website bisa diakses via IP/subdomain default hosting
2. ✅ Arahkan DNS ke hosting (nameserver atau A record)
3. ✅ Tunggu propagasi — verifikasi dengan `nslookup`
4. ✅ Jalankan AutoSSL di cPanel
5. ✅ Baru aktifkan force HTTPS

Jangan force HTTPS sebelum SSL aktif — website tidak bisa diakses sama sekali.

---

## Checklist

- [ ] Domain mengarah ke IP server hosting (verifikasi `nslookup`)
- [ ] AutoSSL aktif di cPanel (gembok hijau)
- [ ] `https://domainmu.com` bisa diakses
- [ ] `http://domainmu.com` redirect ke HTTPS (301)
- [ ] `www.domainmu.com` bisa diakses
- [ ] Tidak ada mixed content warning di console browser
- [ ] (Kalau pakai Cloudflare) SSL mode = Full
