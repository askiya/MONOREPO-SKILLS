# Paket F — VPS Docker Manual

> 🟠 **LANJUTAN** · Estimasi **5 hari** · Biaya **MENENGAH**

## Hasil Akhir

Frontend Next.js standalone, API Express atau Fastify, dan PostgreSQL berjalan sebagai container melalui Docker Compose. Nginx menjadi reverse proxy, Certbot mengurus TLS, dan GitHub Actions mengirim rilis lewat SSH.

## Stack

| Bagian | Teknologi |
|---|---|
| Frontend | Next.js standalone |
| Backend | Express atau Fastify |
| Database | PostgreSQL container |
| Runtime | Docker Compose |
| Proxy | Nginx host |
| TLS | Certbot / Let's Encrypt |
| CI/CD | GitHub Actions + SSH |

```text
Pengguna
   │ HTTPS
   ▼
Nginx :443 ─┬─ domain.com     → web container :3000
             └─ api.domain.com → api container :4000
                                      │
                                      ▼
                              PostgreSQL container :5432
                              (network internal, volume)

GitHub Actions ── SSH ──> VPS ── docker compose build/up
```

## Bedanya dengan Paket E

| Paket E — Coolify | Paket F — Manual |
|---|---|
| Panel mengatur build, proxy, TLS, env | Semua file dan perintah dikelola sendiri |
| Lebih cepat dioperasikan | Kontrol dan pembelajaran lebih dalam |
| Abstraksi platform | Debug langsung Docker, Nginx, systemd, jaringan |
| Lebih sedikit titik salah konfigurasi | Lebih bisa rusak karena semua manual |

## Kapan Pakai

- Paham Linux, SSH, Docker, jaringan, log, dan rollback.
- Memerlukan web dan API terpisah.
- Ingin kontrol image, network, volume, proxy, dan pipeline.
- Siap menangani patch keamanan dan pemulihan server.

## Jangan Pakai

- Ini deploy VPS pertama.
- Tidak bisa memulihkan akses SSH atau restore database.
- Satu aplikasi sederhana cukup dengan platform managed atau Coolify.
- Tidak tersedia waktu maintenance rutin.

## Alur 5 Hari

| Hari | Fokus | Gerbang selesai |
|---|---|---|
| 1 | Pedoman dan CLI agent | Dokumen serta batas layanan jelas |
| 2 | Web, API, PostgreSQL | Integrasi lokal lulus |
| 3 | Docker multi-stage dan Compose | Semua container sehat |
| 4 | VPS, Nginx, Certbot | HTTPS dan hardening lulus |
| 5 | CI/CD, backup, operasi | Deploy dan rollback diuji |

```text
Rencana → CLI Agent → Web + API + DB → Docker lokal
→ Hardening VPS → Compose produksi → Nginx → Certbot
→ GitHub Actions → Monitoring + Backup
```

## Urutan Panduan

1. [`01-pedoman.md`](01-pedoman.md)
2. [`02-ai-agent.md`](02-ai-agent.md)
3. [`03-build.md`](03-build.md)
4. [`04-preview.md`](04-preview.md)
5. [`05-deploy.md`](05-deploy.md)
6. [`06-domain-ssl.md`](06-domain-ssl.md)
7. [`07-maintenance.md`](07-maintenance.md)
8. [`08-rekomendasi-hosting.md`](08-rekomendasi-hosting.md) — rekomendasi hosting dan provider.

## Slot Video

> **Video Paket F:** _belum direkam_. Tempel URL video di sini setelah tersedia.

## Checklist Kelulusan

- [ ] Dokumen perencanaan memisahkan tanggung jawab web, API, dan DB.
- [ ] CLI agent mengikuti AGENTS dan tidak menyentuh secret/produksi tanpa izin.
- [ ] Image web dan API memakai multi-stage build serta user non-root.
- [ ] `docker compose up --build` sehat dan test lulus.
- [ ] SSH key, UFW, Fail2ban, Docker, dan user deploy aman.
- [ ] Nginx meneruskan web/API; database tidak terekspos publik.
- [ ] Certbot auto-renew teruji.
- [ ] GitHub Actions deploy dan prosedur rollback teruji.
- [ ] Backup PostgreSQL tersimpan di lokasi lain dan restore diuji.

## Rujukan

- [`../../docs/10-deploy-vps/01-setup-vps.md`](../../docs/10-deploy-vps/01-setup-vps.md)
- [`../../docs/10-deploy-vps/03-docker-nginx-ssl.md`](../../docs/10-deploy-vps/03-docker-nginx-ssl.md)
- [`../../deployment-examples/docker/`](../../deployment-examples/docker/)
- [`../../deployment-examples/nginx/`](../../deployment-examples/nginx/)
