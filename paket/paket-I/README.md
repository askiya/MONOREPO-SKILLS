# Paket I — Multi-Service Docker

> 🔴 **EXPERT** · Estimasi: **7 hari** · Biaya: **MENENGAH**

Paket produksi untuk aplikasi yang perlu frontend, API, worker antrean, cache, database, reverse proxy, TLS otomatis, dan pemantauan dalam satu Docker Compose.

## Stack

Next.js + NestJS/Fastify + PostgreSQL + Redis + BullMQ + Docker Compose + Traefik (auto SSL) + Uptime Kuma.

## Diagram Stack

```text
Internet
   │ HTTPS
   ▼
Traefik :80/:443 ───── ACME/Let's Encrypt
   ├── app.example.com ──► web (Next.js :3000)
   ├── api.example.com ──► api (NestJS/Fastify :4000)
   └── status.example.com ► uptime-kuma :3001
                              │
web ──HTTP──► api ──SQL────► PostgreSQL
                │
                ├──cache──► Redis
                └──job────► BullMQ/Redis ◄── worker
```

Hanya Traefik membuka port publik. PostgreSQL dan Redis tetap di jaringan internal Docker.

## Kapan Pakai

- API dan worker harus diskalakan atau dirilis terpisah.
- Pekerjaan berat perlu antrean, retry, dan observabilitas.
- Tim mampu mengelola VPS, Docker, DNS, backup, dan insiden.
- Biaya managed platform sudah lebih tinggi daripada satu VPS terukur.

## Kapan Jangan Pakai

- Landing page, MVP kecil, atau aplikasi tanpa proses latar belakang.
- Belum pernah mengelola Docker, firewall, dan pemulihan backup.
- Butuh high availability lintas node; gunakan orchestrator atau layanan terkelola.
- Tidak ada orang yang bertanggung jawab atas patch dan alarm server.

## Prasyarat

- Linux, jaringan TCP/IP, DNS, Git, Node.js, dan Docker sudah dipahami.
- VPS minimal 4 GB RAM untuk lab; ukur produksi dari beban nyata.
- Domain dikelola di Cloudflare dan email operasional tersedia untuk ACME.
- SSH key, akses `sudo`, dan rencana backup off-site tersedia.

## Alur 7 Hari

| Hari | Fokus | Bukti selesai |
|---|---|---|
| 1 | Dokumen dan batas layanan | PRD, ARCHITECTURE, TASKS disetujui |
| 2 | Hermes Agent dan kontrak API | konteks agent tervalidasi |
| 3 | Web, API, worker, DB, Redis | semua image berhasil dibangun |
| 4 | Compose lokal dan test per layanan | healthcheck hijau, job selesai |
| 5 | VPS dan Compose produksi | layanan pulih setelah restart |
| 6 | DNS, Traefik, TLS | tiga hostname HTTPS valid |
| 7 | Monitor, backup, keamanan | alarm dan restore drill lulus |

## Urutan Panduan

1. [`01-pedoman.md`](01-pedoman.md) — perencanaan multi-service.
2. [`02-ai-agent.md`](02-ai-agent.md) — Hermes Agent dan pembagian konteks.
3. [`03-build.md`](03-build.md) — web, API, worker, PostgreSQL, Redis.
4. [`04-preview.md`](04-preview.md) — Compose lokal dan testing.
5. [`05-deploy.md`](05-deploy.md) — VPS dan Compose produksi.
6. [`06-domain-ssl.md`](06-domain-ssl.md) — Cloudflare, Traefik, TLS.
7. [`07-maintenance.md`](07-maintenance.md) — log, monitor, backup, keamanan.

## Video Tutorial

> Slot video mentor: **belum tersedia**. Ikuti panduan teks dan simpan bukti tiap checklist.

## Batas Paket

Paket ini satu-node. Tidak mencakup Kubernetes, database cluster, failover otomatis lintas VPS, atau zero-downtime migration kompleks.

## Checklist Selesai

- [ ] Diagram memuat web, API, worker, PostgreSQL, Redis, Traefik, dan Uptime Kuma.
- [ ] `docker compose config` valid tanpa secret tertanam.
- [ ] Semua container sehat setelah `docker compose up -d`.
- [ ] Job BullMQ diproses worker dan retry teruji.
- [ ] PostgreSQL dan Redis tidak punya port publik.
- [ ] HTTPS valid untuk frontend, API, dan status.
- [ ] Restart policy dan healthcheck aktif.
- [ ] Alarm Uptime Kuma sampai ke kanal tim.
- [ ] Backup terenkripsi tersimpan off-site dan restore pernah diuji.
