# 01 — Dokumen Perencanaan (Paket C)

## Tujuan

Kamu punya 6 dokumen perencanaan lengkap: PRD, SDLC, DESIGN, ARCHITECTURE,
TASKS, AGENTS — disesuaikan untuk arsitektur cPanel + Next.js static export.

---

## Sebelum Mulai

- [ ] Sudah baca [`../../docs/02-dokumen-perencanaan/01-prd.md`](../../docs/02-dokumen-perencanaan/01-prd.md)
- [ ] Sudah lihat contoh terisi di [`../../examples/toko-produk-digital/`](../../examples/toko-produk-digital/)
- [ ] Punya folder project kosong

---

## Langkah

### 1. Buat Folder Project

```bash
mkdir my-cpanel-project
cd my-cpanel-project
git init
```

### 2. Copy Template Dokumen

Copy semua template dari monorepo ke project kamu:

```bash
cp ../../templates/PRD.md ./PRD.md
cp ../../templates/SDLC.md ./SDLC.md
cp ../../templates/DESIGN.md ./DESIGN.md
cp ../../templates/ARCHITECTURE.md ./ARCHITECTURE.md
cp ../../templates/TASKS.md ./TASKS.md
cp ../../templates/AGENTS.md ./AGENTS.md
cp ../../templates/env.example ./.env.example
```

Atau minta Antigravity copy otomatis (lihat [02-ai-agent.md](02-ai-agent.md)).

### 3. Isi PRD

Buka `PRD.md`, isi bagian:

| Bagian | Yang kamu tulis |
|---|---|
| Ringkasan Produk | Deskripsi 1 kalimat, siapa penggunanya |
| Fitur Utama | List fitur MVP (contoh: landing page, form kontak, galeri) |
| Stack | Next.js static export + PHP API + MySQL + cPanel |
| Target Deploy | cPanel shared hosting |
| Batasan | Tidak ada SSR, tidak ada WebSocket, upload manual |

Panduan detail: [`../../docs/02-dokumen-perencanaan/01-prd.md`](../../docs/02-dokumen-perencanaan/01-prd.md)

### 4. Isi SDLC

Timeline khas Paket C:

```text
Hari 1: Perencanaan + setup AI agent + mulai build frontend
Hari 2: Selesaikan frontend + PHP API + MySQL + tes localhost
Hari 3: Upload ke cPanel + domain + SSL + backup pertama
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/02-sdlc.md`](../../docs/02-dokumen-perencanaan/02-sdlc.md)

### 5. Isi DESIGN

Tentukan:
- Warna, font, spacing (boleh dari template Tailwind)
- Layout mobile-first (cPanel hosting biasanya diakses dari HP juga)
- Komponen utama: header, hero, fitur, CTA, footer

Panduan detail: [`../../docs/02-dokumen-perencanaan/03-design-md.md`](../../docs/02-dokumen-perencanaan/03-design-md.md)

### 6. Isi ARCHITECTURE

Arsitektur khas Paket C:

```text
┌─────────────────────────┐
│  Next.js Static Export   │
│  (HTML/CSS/JS di out/)   │
├─────────────────────────┤
│  PHP API (opsional)      │
│  /api/contact.php        │
├─────────────────────────┤
│  MySQL (phpMyAdmin)      │
├─────────────────────────┤
│  cPanel Shared Hosting   │
│  Apache + AutoSSL        │
└─────────────────────────┘
```

Yang penting ditulis di ARCHITECTURE:
- `output: 'export'` di `next.config.js` — WAJIB untuk cPanel
- Semua data fetching harus client-side (`useEffect` / SWR) atau build-time
- Tidak ada `getServerSideProps` — itu butuh Node server
- PHP API hanya untuk endpoint sederhana (form submit, data query)

Panduan detail: [`../../docs/02-dokumen-perencanaan/04-architecture-md.md`](../../docs/02-dokumen-perencanaan/04-architecture-md.md)

### 7. Isi TASKS

Pecah jadi task kecil:

```markdown
- [ ] Setup Next.js project dengan `output: 'export'`
- [ ] Buat halaman utama (landing page)
- [ ] Buat komponen header, footer, hero
- [ ] Setup Tailwind CSS
- [ ] Buat PHP endpoint /api/contact.php
- [ ] Buat MySQL database + tabel
- [ ] Test build → cek folder out/
- [ ] Upload ke cPanel
- [ ] Setup domain + SSL
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/05-tasks-md.md`](../../docs/02-dokumen-perencanaan/05-tasks-md.md)

### 8. Isi AGENTS

Isi sesuai AI agent yang kamu pakai (Antigravity untuk Paket C):

```markdown
## Agent: Antigravity
- Bisa: baca/tulis file, jalankan terminal, bantu coding
- Tidak bisa: upload ke cPanel (kamu upload manual)
- Aturan: jangan buat getServerSideProps, semua harus static-compatible
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/06-agents-md.md`](../../docs/02-dokumen-perencanaan/06-agents-md.md)

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Agent bikin SSR code | ARCHITECTURE tidak menyebut static export | Tulis eksplisit di ARCHITECTURE: `output: 'export'`, dilarang `getServerSideProps` |
| Terlalu banyak fitur di PRD | Ambisius, lupa batasan cPanel | Batasi 3-5 fitur untuk MVP |
| Dokumen kosong template saja | Belum diisi | Lihat contoh terisi di `examples/toko-produk-digital/` |

---

## Checklist

- [ ] PRD terisi, ada ringkasan produk dan fitur MVP
- [ ] SDLC ada timeline 3 hari
- [ ] DESIGN ada warna, font, layout
- [ ] ARCHITECTURE menyebut `output: 'export'` dan larangan SSR
- [ ] TASKS ada minimal 8 task spesifik
- [ ] AGENTS ada aturan untuk Antigravity
- [ ] Semua file sudah di-commit ke repo
