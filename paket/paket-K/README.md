# Paket K — Santriverse Hybrid

> 🟠 **LANJUTAN** · Estimasi: 5 hari · Biaya: **MENENGAH** · 🎯 Stack nyata Santriverse

Paket studi kasus berdasarkan arsitektur Santriverse: source code disimpan di
GitHub, frontend dibangun dan di-host terpisah di Cloudflare, backend/API serta
PostgreSQL berjalan di Coolify pada VPS, dan pengembangan dibantu Hermes Agent.

## Stack Diagram

```text
                    ┌──────────────────┐
                    │  HERMES AGENT    │
                    │  baca docs, kode │
                    │  test, review    │
                    └────────┬─────────┘
                             │ edit lokal
                             ▼
                    ┌──────────────────┐
                    │  GITHUB PRIVATE  │
                    │  source of truth │
                    └───────┬──┬───────┘
                            │  │
                  auto build│  │auto deploy
                            ▼  ▼
            ┌─────────────────┐   ┌────────────────────┐
USER ──────▶│ CLOUDFLARE      │──▶│ COOLIFY DI VPS     │
Browser     │ Pages/Workers   │API│ Backend/API        │
            │ Frontend        │   │                    │
            └─────────────────┘   │ PostgreSQL         │
                                  │ persistent volume  │
                                  └────────────────────┘
```

## Komponen

| Layer | Pilihan | Fungsi |
|---|---|---|
| AI Agent | Hermes Agent | coding, debugging, test, dokumentasi |
| Source | GitHub private | sumber kebenaran dan trigger deploy |
| Frontend | Next.js/static-compatible build | UI member/admin |
| Frontend hosting | Cloudflare Pages/Workers | CDN global, domain frontend |
| Backend | Node.js API | auth, CRUD, payment, business logic |
| Backend hosting | Coolify di VPS | container, env, log, auto deploy |
| Database | PostgreSQL di Coolify | data aplikasi, persistent volume |
| DNS/CDN | Cloudflare | domain, proxy, DNS, proteksi dasar |
| CI | GitHub Actions | lint, test, build sebelum deploy |
| Monitoring | Coolify logs + uptime monitor | deteksi downtime/error |

## Kenapa Level Lanjutan?

Bukan karena kode lebih rumit, tapi karena ada **dua target deployment**:

1. frontend dan backend punya build/deploy terpisah,
2. CORS dan URL API harus dikonfigurasi benar,
3. environment staging/production harus sinkron namun terpisah,
4. database berada di VPS dan wajib punya backup eksternal,
5. error bisa berasal dari Cloudflare, DNS, Coolify, container, atau database.

Selesaikan minimal satu paket 🟢 dan satu paket 🔵 sebelum mengambil paket ini.

## Kapan Pakai Paket Ini

- Produk sudah punya frontend dan backend yang jelas terpisah.
- Frontend butuh distribusi CDN global.
- Backend butuh persistent process, webhook, cron, worker, atau kontrol VPS.
- Tim ingin source tetap di GitHub dan deploy otomatis.
- Butuh biaya lebih terkendali daripada semua layanan managed terpisah.

## Kapan JANGAN Pakai

- Project pertama dan belum pernah deploy.
- Aplikasi hanya landing page statis — gunakan Paket B.
- Next.js sederhana tanpa worker/WebSocket — Paket A lebih cepat.
- Tidak ada orang yang bertanggung jawab update VPS dan backup DB.
- Tidak siap mengelola CORS, environment, dan dua pipeline deploy.

## Prasyarat

- [ ] Pernah deploy project sampai online
- [ ] Paham Git branch, commit, push, pull request
- [ ] Paham frontend memanggil API melalui URL
- [ ] Paham environment variable dan CORS
- [ ] Punya VPS minimal sesuai beban aplikasi
- [ ] Punya domain di Cloudflare
- [ ] Hermes Agent sudah terpasang

## Alur Lengkap

1. [Pedoman dan arsitektur](01-pedoman.md)
2. [Setup Hermes Agent](02-ai-agent.md)
3. [Build frontend, backend, database](03-build.md)
4. [Preview dan test lokal](04-preview.md)
5. [Deploy GitHub → Cloudflare + Coolify](05-deploy.md)
6. [Domain, DNS, SSL, CORS](06-domain-ssl.md)
7. [Backup, monitoring, maintenance](07-maintenance.md)
8. [Rekomendasi hosting dan provider](08-rekomendasi-hosting.md)

## Video Tutorial

> 🎬 Video belum tersedia. Mentor dapat merekam seri:
> 1. membuat repo dan dokumen,
> 2. build dengan Hermes,
> 3. deploy backend + PostgreSQL ke Coolify,
> 4. deploy frontend ke Cloudflare,
> 5. menghubungkan domain, CORS, dan environment,
> 6. backup/restore serta troubleshooting.

## Checklist Selesai

- [ ] Dokumen perencanaan frontend/backend konsisten
- [ ] Hermes Agent membaca aturan repo dan tidak menebak stack
- [ ] Frontend dan backend lulus test/build lokal
- [ ] Backend dan PostgreSQL aktif di Coolify
- [ ] Frontend aktif di Cloudflare
- [ ] Source kedua layanan berasal dari GitHub
- [ ] Frontend hanya memakai URL API environment, bukan hardcode localhost
- [ ] CORS hanya mengizinkan domain frontend
- [ ] HTTPS valid di frontend dan backend
- [ ] Database punya backup otomatis + restore drill
- [ ] Monitoring frontend, API, dan database aktif
