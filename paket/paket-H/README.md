# PAKET H — Monorepo Turborepo

![Level](https://img.shields.io/badge/Level-%F0%9F%9F%A0%20LANJUTAN-orange)
![Estimasi](https://img.shields.io/badge/Estimasi-5%20hari-informational)
![Biaya](https://img.shields.io/badge/Biaya-MURAH-yellow)

> Satu repository, dua aplikasi, satu paket bersama: Next.js frontend + Express API + shared types/utilities.

## Ringkasan

| Item | Keterangan |
|---|---|
| Level | 🟠 LANJUTAN |
| Estimasi | 5 hari (±6 jam/hari) |
| Biaya | MURAH; hosting frontend bisa gratis, API/DB berbayar ringan |
| Orkestrator | Turborepo + npm workspaces |
| Frontend | `apps/web`: Next.js, deploy Vercel |
| Backend | `apps/api`: Express, deploy Railway atau Render |
| Kode bersama | `packages/shared`: types, schema validasi, utilities murni |
| Domain | `app.domain.com` + `api.domain.com` melalui Cloudflare |
| Prasyarat | Nyaman dengan TypeScript, REST API, environment variable, deploy |

## Stack Diagram

```text
                         ┌──────────────────────┐
                         │       Pengguna       │
                         └──────────┬───────────┘
                                    │ HTTPS
                   ┌────────────────┴────────────────┐
                   ▼                                 ▼
       app.domain.com                    api.domain.com
       Cloudflare DNS                    Cloudflare DNS
                   │                                 │
                   ▼                                 ▼
┌──────────────────────────┐           ┌──────────────────────────┐
│ Vercel                   │  HTTPS    │ Railway / Render         │
│ apps/web — Next.js       ├──────────►│ apps/api — Express       │
│ NEXT_PUBLIC_API_URL      │           │ CORS_ORIGIN              │
└──────────────────────────┘           └────────────┬─────────────┘
                                                   │ DATABASE_URL
                                                   ▼
                                      ┌──────────────────────────┐
                                      │ PostgreSQL               │
                                      └──────────────────────────┘

┌──────────────────── SATU GITHUB REPOSITORY ────────────────────┐
│ apps/web ───────┐                                              │
│ apps/api ───────┼── import @santriverse/shared                 │
│ packages/shared ┘                                              │
│ turbo.json: dev, build, lint, test pipeline                    │
└────────────────────────────────────────────────────────────────┘
```

## Kapan Pakai

✅ Pakai kalau:
- Web dan API punya siklus deploy berbeda.
- Lebih dari satu aplikasi memakai type/schema yang sama.
- Backend butuh proses persistent, worker, WebSocket, atau runtime non-Vercel.
- Tim frontend dan backend bekerja paralel tetapi perlu kontrak bersama.
- Siap menjaga batas dependency dan pipeline lint/test/build.

## Kapan Jangan Pakai

❌ Jangan pakai kalau:
- Produk hanya CRUD sederhana dalam satu Next.js.
- Belum paham deploy satu aplikasi.
- `packages/shared` hanya akan berisi satu konstanta.
- Tim satu orang dan pemisahan API tidak memberi kebutuhan nyata.
- Menganggap monorepo berarti semua service harus deploy bersama.

Monorepo mengurangi duplikasi, tetapi menambah konfigurasi. Pilih karena kebutuhan, bukan tren.

## Alur 5 Hari

```text
HARI 1  01-pedoman.md  → PRD, SDLC, DESIGN, ARCHITECTURE monorepo, TASKS
        02-ai-agent.md → setup Hermes Agent di root monorepo
HARI 2  03-build.md    → init Turborepo + apps/web + packages/shared
HARI 3  03-build.md    → apps/api + kontrak API + turbo build
HARI 4  04-preview.md  → turbo dev, test per app, integration test
HARI 5  05-deploy.md   → Vercel web + Railway/Render API
        06-domain-ssl.md → app + api domain, Cloudflare SSL
        07-maintenance.md → dependency, shared update, cache
```

## Isi Paket

| File | Isi |
|---|---|
| [01-pedoman.md](01-pedoman.md) | Dokumen perencanaan + struktur monorepo di ARCHITECTURE |
| [02-ai-agent.md](02-ai-agent.md) | Setup Hermes Agent dari root monorepo |
| [03-build.md](03-build.md) | Turborepo, Next.js, Express, shared package, pipeline |
| [04-preview.md](04-preview.md) | `turbo dev`, test tiap workspace dan integrasi |
| [05-deploy.md](05-deploy.md) | Vercel web + Railway/Render API + env terpisah |
| [06-domain-ssl.md](06-domain-ssl.md) | `app.domain.com` + `api.domain.com` |
| [07-maintenance.md](07-maintenance.md) | Dependency, shared update, Turborepo cache |
| [08-rekomendasi-hosting.md](08-rekomendasi-hosting.md) | Rekomendasi hosting dan provider |

## Video Panduan

<!-- VIDEO SLOT: paket-H-overview -->
> 🎬 **Video 1 — Kapan Monorepo Layak Dipakai** _(coming soon)_ · target 10 menit

<!-- VIDEO SLOT: paket-H-build -->
> 🎬 **Video 2 — Turborepo: Web, API, Shared** _(coming soon)_ · target 35 menit

<!-- VIDEO SLOT: paket-H-deploy -->
> 🎬 **Video 3 — Dua Deploy dari Satu Repo** _(coming soon)_ · target 25 menit

<!-- VIDEO SLOT: paket-H-cache -->
> 🎬 **Video 4 — Cache dan Maintenance** _(coming soon)_ · target 12 menit

## Checklist Kelulusan

**Perencanaan**
- [ ] ARCHITECTURE memuat tree `apps/` dan `packages/`
- [ ] Ownership, dependency direction, kontrak API, dan deploy target tertulis
- [ ] Environment variable web/API dipisahkan
- [ ] Hermes Agent diarahkan ke root monorepo

**Build & test**
- [ ] `npm install` dari root sukses
- [ ] `npm run dev` menjalankan web dan API
- [ ] `@santriverse/shared` dipakai web dan API
- [ ] `npx turbo run lint test build` sukses
- [ ] Perubahan shared memicu build consumer
- [ ] API health dan integrasi web → API lulus

**Deploy**
- [ ] Vercel Root Directory benar untuk `apps/web`
- [ ] API Railway/Render memakai konfigurasi monorepo yang benar
- [ ] Env vars web/API terpisah dan tidak bocor
- [ ] `app.domain.com` dan `api.domain.com` aktif dengan HTTPS
- [ ] CORS hanya mengizinkan origin frontend produksi
- [ ] Deploy frontend dan backend dapat dilakukan terpisah

**Bukti ke mentor:** URL dua domain, output turbo build, screenshot struktur service, dan satu request API produksi sukses.

## Perkiraan Biaya

| Komponen | Kisaran |
|---|---|
| Vercel web | gratis untuk proyek hobi sesuai ketentuan terbaru |
| Railway/Render API | gratis terbatas atau ~$5–10/bulan, cek pricing terbaru |
| PostgreSQL | ~$1–7/bulan sesuai pemakaian/provider |
| Domain | ~Rp180.000/tahun |
| Cloudflare DNS/SSL | gratis |

➡️ Mulai dari **[01-pedoman.md](01-pedoman.md)**.
