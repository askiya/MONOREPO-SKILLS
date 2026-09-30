# H1 — Dokumen Perencanaan Monorepo

> 🟠 LANJUTAN · Target waktu: 4–5 jam

## Tujuan

Mengunci batas aplikasi, arah dependency, kontrak API, ownership, dan strategi deploy sebelum membuat Turborepo.

## Dokumen Wajib

```text
docs/
├── 01-PRD.md
├── 02-SDLC.md
├── 03-DESIGN.md
├── 04-ARCHITECTURE.md
└── 05-TASKS.md
AGENTS.md
```

Copy template repo pedoman ke project, lalu isi—jangan biarkan placeholder.

## Langkah 1 — Tulis PRD Berdasarkan Produk, Bukan Folder

PRD menjelaskan pengalaman pengguna. Jangan menulis “buat apps/web” sebagai fitur.

Contoh batas MVP:

```markdown
## Fitur MVP
1. Pengguna melihat katalog di web.
2. Pengguna membuat pesanan melalui API.
3. Admin melihat status pesanan.
4. Web dan API berbagi schema validasi pesanan.

## Di luar MVP
- Mobile app
- Event bus
- Microservice terpisah per domain
- Shared UI package
```

**Hasil yang diharapkan:** setiap fitur punya kriteria selesai yang terlihat pengguna atau dapat diuji API.

## Langkah 2 — Tulis Struktur Monorepo di ARCHITECTURE

Bagian ini wajib ada:

```markdown
## Struktur Monorepo
```
```text
santriverse/
├── apps/
│   ├── web/                 # Next.js; deploy Vercel
│   │   ├── src/app/
│   │   └── package.json
│   └── api/                 # Express; deploy Railway/Render
│       ├── src/
│       └── package.json
├── packages/
│   └── shared/              # Type, Zod schema, utility murni
│       ├── src/index.ts
│       ├── package.json
│       └── tsconfig.json
├── docs/
├── AGENTS.md
├── package.json             # npm workspaces + root scripts
├── turbo.json
└── tsconfig.base.json
```

Tambahkan keputusan:

```markdown
## Arah Dependency
- apps/web → packages/shared
- apps/api → packages/shared
- packages/shared TIDAK BOLEH import dari apps/*
- apps/web TIDAK BOLEH import source langsung dari apps/api
- Komunikasi web–API hanya lewat HTTP berdasarkan kontrak shared

## Ownership
| Area | Pemilik | Deploy target |
|---|---|---|
| apps/web | tim frontend | Vercel |
| apps/api | tim backend | Railway (utama) / Render |
| packages/shared | bersama; perubahan butuh review web+API | dibundel oleh consumer |

## Kontrak API
- Base path: /v1
- Format sukses: { data: ... }
- Format gagal: { error: { code, message, details? } }
- Schema input/output penting berada di @santriverse/shared
- Breaking change wajib endpoint versi baru atau migrasi consumer lebih dulu

## Environment
| Workspace | Variable | Scope |
|---|---|---|
| apps/web | NEXT_PUBLIC_API_URL | browser, boleh publik |
| apps/api | PORT | runtime backend |
| apps/api | DATABASE_URL | secret |
| apps/api | CORS_ORIGIN | server |

## Deploy
- apps/web: Vercel, Root Directory apps/web
- apps/api: Railway atau Render
- Dua service deploy terpisah dari satu repository
```

> Jangan taruh `DATABASE_URL` di root `.env` bila hanya API yang butuh. Scope secret sekecil mungkin.

## Langkah 3 — Catat Keputusan Arsitektur

Tambahkan ADR singkat di ARCHITECTURE:

```markdown
## ADR-001: Monorepo Turborepo
Status: diterima
Konteks: web dan API deploy terpisah tetapi berbagi schema TypeScript.
Keputusan: npm workspaces + Turborepo; shared hanya kode runtime-agnostic.
Konsekuensi: satu install dan cache build lebih cepat; konfigurasi deploy lebih rumit.
Alternatif ditolak: dua repo karena duplikasi kontrak dan koordinasi versi.
```

## Langkah 4 — SDLC untuk Banyak Workspace

```markdown
## Gate Perubahan
1. Ubah shared schema secara backward-compatible.
2. Update dan test API.
3. Update dan test web.
4. Jalankan turbo lint/test/build dari root.
5. Deploy API lebih dulu bila web membutuhkan kemampuan baru.
6. Deploy web setelah health API hijau.

## Definition of Done
- Test workspace yang berubah lulus.
- Consumer shared yang terdampak lulus.
- Root build lulus.
- Env docs diperbarui.
- Tidak ada breaking API tanpa rencana migrasi.
```

## Langkah 5 — TASKS per Workspace

```markdown
- [ ] H01 Init npm workspaces + Turborepo
- [ ] H02 Scaffold apps/web
- [ ] H03 Scaffold apps/api
- [ ] H04 Buat packages/shared
- [ ] H05 Definisikan OrderInputSchema + types
- [ ] H06 API GET /health dan POST /v1/orders
- [ ] H07 Web katalog + form memakai shared schema
- [ ] H08 Turbo dev/build/lint/test pipeline
- [ ] H09 Test API terisolasi
- [ ] H10 Test integrasi web → API
- [ ] H11 Deploy API
- [ ] H12 Deploy web
- [ ] H13 Domain, SSL, CORS
```

Setiap task harus menyebut workspace dan perintah verifikasi.

## Langkah 6 — Aturan AGENTS.md

```markdown
## Aturan Monorepo
- Jalankan agent dari root repository.
- Sebelum edit, sebutkan workspace yang terdampak.
- Jangan import lintas apps/*.
- Shared hanya types, schema, constants, utility murni; tanpa env/DB/UI.
- Jangan mengubah root config untuk memperbaiki satu app tanpa menguji semua consumer.
- Selesai hanya setelah test workspace dan `turbo build` root lulus.
- Secret ditempatkan di platform/workspace pemiliknya, tidak di root/source.
```

## Error Umum

| Masalah | Penyebab | Perbaikan |
|---|---|---|
| Shared menjadi “folder sampah” | Batas tidak ditulis | Batasi runtime-agnostic contract/utilities |
| Web import handler API | Dependency direction kabur | Gunakan HTTP + shared schema |
| Semua deploy saat satu file berubah | Filtering/path config belum dirancang | Dokumentasikan workspace deploy |
| Env tersebar di root | Scope tidak ditentukan | Buat tabel pemilik variable |

## Checklist

- [ ] PRD ≤5 fitur MVP
- [ ] ARCHITECTURE memuat tree lengkap
- [ ] Arah dependency eksplisit dan satu arah
- [ ] Kontrak API dan aturan breaking change tertulis
- [ ] Web/API/shared punya ownership
- [ ] Env vars dipisah per workspace
- [ ] Urutan deploy API/web tertulis
- [ ] TASKS punya workspace + verifikasi
- [ ] AGENTS.md punya aturan monorepo

➡️ Lanjut ke **[02-ai-agent.md](02-ai-agent.md)**.
