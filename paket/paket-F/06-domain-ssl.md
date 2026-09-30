# 06 — Domain, DNS, dan SSL Manual (Certbot)

## Tujuan

DNS mengarah ke VPS, Nginx server block menerima trafik, dan Certbot menerbitkan sertifikat Let's Encrypt dengan auto-renew.

## Prasyarat

- Bab `05-deploy.md` selesai; Nginx sudah berjalan dan meneruskan web/API.
- Domain sudah dibeli.

## 1. Pilih Pengelola DNS

| Opsi | Kapan pakai |
|---|---|
| Cloudflare | Ingin CDN, proxy, firewall, rate limit |
| Registrar langsung | Cukup DNS saja tanpa fitur tambahan |

Jika memakai Cloudflare, ikuti langkah pemindahan nameserver sama seperti di Paket E (bab `06-domain-ssl.md`).

## 2. Buat DNS Record

| Type | Name | Content |
|---|---|---|
| A | `@` | `IP_SERVER` |
| A | `www` | `IP_SERVER` |
| A | `api` | `IP_SERVER` |
| A | `staging` | `IP_SERVER` |

Jika memakai Cloudflare dan mengelola TLS sendiri lewat Certbot, **set proxy ke DNS only** (abu-abu) pada tahap awal. Proxy oranye pada saat yang sama dengan Certbot dapat mengganggu validasi HTTP-01. Setelah sertifikat berhasil, bisa aktifkan proxy.

Verifikasi:

```bash
nslookup domain-kamu.com
nslookup api.domain-kamu.com
```

**Hasil yang diharapkan:** keduanya mengembalikan IP server.

## 3. Nginx Server Block untuk Setiap Domain

Jika belum ada dari bab 05, buat `/etc/nginx/sites-available/myapp.conf`:

```nginx
server {
    listen 80;
    server_name domain-kamu.com www.domain-kamu.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name api.domain-kamu.com;

    location / {
        proxy_pass http://127.0.0.1:4000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Symlink dan verifikasi:

```bash
sudo ln -sf /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/myapp.conf
sudo nginx -t
sudo systemctl reload nginx
```

**Hasil yang diharapkan:** `syntax is ok` dan `test is successful`. Jangan reload bila `nginx -t` gagal.

## 4. Pasang Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

## 5. Minta Sertifikat

DNS harus sudah mengarah ke IP server dan port 80 harus terbuka sebelum Certbot menjalankan validasi. Kalau belum, Certbot gagal dan rate limit Let's Encrypt mulai berlaku (5 kegagalan per jam per domain).

```bash
sudo certbot --nginx \
  -d domain-kamu.com \
  -d www.domain-kamu.com \
  -d api.domain-kamu.com
```

Ikuti prompt. Pilih redirect HTTP ke HTTPS saat ditanya.

**Hasil yang diharapkan:** Certbot menampilkan `Congratulations!` dan Nginx config diperbarui dengan blok `listen 443 ssl` dan path sertifikat.

Verifikasi:

```bash
curl -I https://domain-kamu.com
curl -I https://api.domain-kamu.com
```

**Hasil yang diharapkan:** HTTP 200; header menunjukkan HTTPS.

## 6. Uji Auto-Renew

Certbot otomatis memasang timer systemd atau cron. Uji tanpa benar-benar memperbarui:

```bash
sudo certbot renew --dry-run
```

**Hasil yang diharapkan:** `Congratulations, all simulated renewals succeeded`.

Jika gagal, periksa:

- apakah port 80 masih terbuka,
- apakah DNS masih mengarah ke server ini,
- apakah ada firewall atau proxy CDN yang menghalangi validasi.

## 7. Cloudflare + Certbot (Opsional)

Jika DNS memakai Cloudflare dan kamu tetap ingin proxy oranye:

1. Pastikan sertifikat Certbot sudah terbit dan aktif.
2. Aktifkan proxy (oranye) di Cloudflare.
3. Set SSL/TLS mode **Full (strict)** di Cloudflare.

Dengan konfigurasi ini, Cloudflare memeriksa sertifikat valid di origin. Jangan gunakan mode Flexible karena koneksi Cloudflare ke server menjadi HTTP — data tidak terenkripsi di jaringan antara edge dan origin.

## 8. Pisahkan Staging dan Produksi

| Lingkungan | Domain | Container | Database |
|---|---|---|---|
| Produksi | `domain-kamu.com` | compose produksi | DB produksi |
| Staging | `staging.domain-kamu.com` | compose staging | DB staging terpisah |

Jangan pakai database produksi untuk staging. Buat compose file atau profil terpisah.

## Kegagalan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Certbot `Could not bind to port 80` | Nginx atau proses lain menempati port | Gunakan plugin `--nginx`, bukan standalone |
| `NXDOMAIN` / DNS belum mengarah | Nameserver belum propagasi | Tunggu, atau cek nameserver di registrar |
| `ERR_TOO_MANY_REDIRECTS` | Redirect loop Cloudflare + Nginx | Matikan redirect Nginx kalau Cloudflare sudah memaksa HTTPS |
| Sertifikat expired | Cron/timer renew gagal | `sudo certbot renew`, lalu debug penyebab |
| Mixed content warning | Asset HTTP di halaman HTTPS | Perbaiki URL asset atau aktifkan Automatic HTTPS Rewrites di Cloudflare |

## Checklist

- [ ] DNS mengarah ke IP server dan sudah terverifikasi `nslookup`.
- [ ] Nginx server block lulus `nginx -t` dan aktif.
- [ ] Certbot menerbitkan sertifikat untuk web dan API.
- [ ] `curl -I https://domain` mengembalikan 200 HTTPS.
- [ ] Auto-renew dry run sukses.
- [ ] Jika Cloudflare: mode Full (strict) aktif.
- [ ] Staging dan produksi domain terpisah.
