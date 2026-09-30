# 06 — Domain, Cloudflare DNS, dan TLS Traefik

## Tujuan

Menghubungkan hostname frontend, API, dan status ke VPS serta menerbitkan sertifikat TLS otomatis melalui Traefik.

## Prasyarat

Stack produksi pada [`05-deploy.md`](05-deploy.md) sehat. Port `80` dan `443` terbuka. Domain aktif di Cloudflare.

## 1. Tentukan Hostname

| Hostname | Tujuan | Akses |
|---|---|---|
| `app.example.com` | Next.js | publik |
| `api.example.com` | NestJS API | publik, rate limit |
| `status.example.com` | Uptime Kuma | publik/read-only atau dilindungi auth |

Jangan buat DNS untuk PostgreSQL, Redis, atau worker.

## 2. Buat DNS Cloudflare

Tambahkan record:

| Tipe | Nama | Nilai | Proxy awal |
|---|---|---|---|
| A | `app` | IP VPS | DNS only |
| A | `api` | IP VPS | DNS only |
| A | `status` | IP VPS | DNS only |

Verifikasi:

```bash
dig +short app.example.com
dig +short api.example.com
dig +short status.example.com
```

**Hasil yang diharapkan:** semua mengembalikan IP publik VPS. Biarkan DNS only sampai sertifikat berhasil; proxy Cloudflare dapat diaktifkan setelah origin HTTPS valid.

## 3. Konfigurasi Traefik ACME

Contoh command Traefik:

```yaml
command:
  - --providers.docker=true
  - --providers.docker.exposedbydefault=false
  - --entrypoints.web.address=:80
  - --entrypoints.web.http.redirections.entrypoint.to=websecure
  - --entrypoints.web.http.redirections.entrypoint.scheme=https
  - --entrypoints.websecure.address=:443
  - --certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL}
  - --certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
  - --certificatesresolvers.letsencrypt.acme.httpchallenge=true
  - --certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web
```

Mount:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro
  - ./data/traefik:/letsencrypt
```

**Hasil yang diharapkan:** Traefik meminta sertifikat saat router HTTPS pertama diakses.

## 4. Aktifkan Router

Setiap service HTTP mendapat router unik. Contoh web:

```yaml
labels:
  - traefik.enable=true
  - traefik.http.routers.web.rule=Host(`${WEB_HOST}`)
  - traefik.http.routers.web.entrypoints=websecure
  - traefik.http.routers.web.tls=true
  - traefik.http.routers.web.tls.certresolver=letsencrypt
  - traefik.http.services.web.loadbalancer.server.port=3000
```

Ulangi untuk API dan Kuma dengan nama router/service berbeda.

```bash
docker compose up -d
docker compose logs --since=10m traefik
```

**Hasil yang diharapkan:** tidak ada error ACME atau router conflict.

## 5. Verifikasi HTTPS

```bash
curl --fail --head https://app.example.com
curl --fail https://api.example.com/health
curl --fail --head https://status.example.com
openssl s_client -connect app.example.com:443 -servername app.example.com </dev/null 2>/dev/null | openssl x509 -noout -issuer -subject -dates
```

**Hasil yang diharapkan:** HTTPS valid, redirect HTTP ke HTTPS bekerja, tanggal sertifikat belum kedaluwarsa.

Setelah valid, aktifkan proxy Cloudflare bila dibutuhkan dan set SSL/TLS mode **Full (strict)**. Jangan gunakan Flexible.

## 6. Wildcard Certificate Bila Perlu

Wildcard hanya perlu untuk subdomain dinamis. Gunakan DNS-01 challenge dan Cloudflare API token dengan izin minimum `Zone:DNS:Edit` pada satu zone.

```yaml
command:
  - --certificatesresolvers.cloudflare.acme.dnschallenge=true
  - --certificatesresolvers.cloudflare.acme.dnschallenge.provider=cloudflare
```

Simpan `CF_DNS_API_TOKEN` sebagai secret server, bukan label atau file repo. Router meminta domain utama dan wildcard melalui `tls.domains`.

Pilih HTTP-01 untuk tiga hostname tetap; lebih sederhana dan blast radius lebih kecil.

## Masalah Umum

| Gejala | Perbaikan |
|---|---|
| ACME `timeout` | cek DNS mengarah ke VPS dan firewall port 80/443 |
| sertifikat default Traefik | cek rule `Host`, resolver, dan jaringan proxy |
| redirect loop Cloudflare | gunakan Full (strict), bukan Flexible |
| rate limit Let's Encrypt | hentikan retry; perbaiki config dan uji staging CA |
| wildcard gagal | cek token zone-scoped dan DNS-01 propagation |

## Checklist

- [ ] DNS frontend, API, dan status mengarah ke VPS.
- [ ] Tidak ada DNS/port publik untuk DB, Redis, atau worker.
- [ ] `acme.json` permission `600`.
- [ ] HTTP redirect ke HTTPS.
- [ ] Sertifikat valid untuk setiap hostname.
- [ ] Cloudflare memakai Full (strict) jika proxy aktif.
- [ ] Wildcard hanya dipakai bila ada kebutuhan subdomain dinamis.
- [ ] API token DNS berizin minimum dan tersimpan sebagai secret.
