# 01 — Pedoman: Menulis Dokumen Perencanaan

## Tujuan

Membuat 6 dokumen perencanaan yang menjadi "otak" AI agent. Tanpa dokumen ini,
agent menebak fitur dan membuat keputusan arsitektur sendiri — hasilnya kacau.

## Sebelum Mulai

- Sudah punya ide project (minimal tahu: untuk siapa, fitur inti apa)
- Sudah baca [`../../docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md`](../../docs/00-mulai-dari-sini/01-apa-itu-vibe-coding.md)

---

## 6 Dokumen yang Harus Ditulis

Tulis dokumen dalam urutan ini. Setiap dokumen bergantung pada dokumen sebelumnya.

| No | Dokumen | Isi | Template | Contoh Terisi |
|---|---|---|---|---|
| 1 | `PRD.md` | Apa yang dibangun, untuk siapa, fitur per fase | [`../../templates/PRD.md`](../../templates/PRD.md) | [`../../examples/toko-produk-digital/01-PRD.md`](../../examples/toko-produk-digital/01-PRD.md) |
| 2 | `SDLC.md` | Alur kerja: branch, commit, review, deploy | [`../../templates/SDLC.md`](../../templates/SDLC.md) | [`../../examples/toko-produk-digital/02-SDLC.md`](../../examples/toko-produk-digital/02-SDLC.md) |
| 3 | `DESIGN.md` | Warna, font, layout, komponen UI | [`../../templates/DESIGN.md`](../../templates/DESIGN.md) | [`../../examples/toko-produk-digital/03-DESIGN.md`](../../examples/toko-produk-digital/03-DESIGN.md) |
| 4 | `ARCHITECTURE.md` | Stack, folder structure, API routes, database schema | [`../../templates/ARCHITECTURE.md`](../../templates/ARCHITECTURE.md) | [`../../examples/toko-produk-digital/04-ARCHITECTURE.md`](../../examples/toko-produk-digital/04-ARCHITECTURE.md) |
| 5 | `TASKS.md` | Daftar task per fase, urutan pengerjaan | [`../../templates/TASKS.md`](../../templates/TASKS.md) | [`../../examples/toko-produk-digital/05-TASKS.md`](../../examples/toko-produk-digital/05-TASKS.md) |
| 6 | `AGENTS.md` | Aturan agent: apa boleh, apa tidak, cara kerja | [`../../templates/AGENTS.md`](../../templates/AGENTS.md) | [`../../examples/toko-produk-digital/06-AGENTS.md`](../../examples/toko-produk-digital/06-AGENTS.md) |

Panduan detail per dokumen ada di [`../../docs/02-dokumen-perencanaan/`](../../docs/02-dokumen-perencanaan/):

- [`01-prd.md`](../../docs/02-dokumen-perencanaan/01-prd.md) — cara menulis PRD
- [`02-sdlc.md`](../../docs/02-dokumen-perencanaan/02-sdlc.md) — cara menulis SDLC
- [`03-design-md.md`](../../docs/02-dokumen-perencanaan/03-design-md.md) — cara menulis DESIGN.md
- [`04-architecture-md.md`](../../docs/02-dokumen-perencanaan/04-architecture-md.md) — cara menulis ARCHITECTURE.md
- [`05-tasks-md.md`](../../docs/02-dokumen-perencanaan/05-tasks-md.md) — cara menulis TASKS.md
- [`06-agents-md.md`](../../docs/02-dokumen-perencanaan/06-agents-md.md) — cara menulis AGENTS.md

---

## Langkah-Langkah

### 1. Buat folder project

```bash
mkdir my-project
cd my-project
git init
```

### 2. Copy semua template

```bash
cp ../pedoman-monorepo/templates/PRD.md .
cp ../pedoman-monorepo/templates/SDLC.md .
cp ../pedoman-monorepo/templates/DESIGN.md .
cp ../pedoman-monorepo/templates/ARCHITECTURE.md .
cp ../pedoman-monorepo/templates/TASKS.md .
cp ../pedoman-monorepo/templates/AGENTS.md .
```

Atau copy manual dari folder `templates/`. Yang penting: **jangan edit template
asli**. Copy dulu, baru isi.

### 3. Isi setiap dokumen

Buka contoh terisi di `examples/toko-produk-digital/` sebagai referensi.

**Urutan pengisian wajib:**

1. **PRD.md** — Mulai dari sini. Tulis siapa pengguna, masalah apa, fitur MVP.
2. **SDLC.md** — Untuk pemula, copy saja contoh SDLC. Ubah nama branch kalau perlu.
3. **DESIGN.md** — Pilih warna, font, layout. Boleh pakai referensi website lain.
4. **ARCHITECTURE.md** — Untuk Paket A, stack-nya sudah ditentukan:
   - Frontend: Next.js 14 + TypeScript + Tailwind CSS
   - Backend: Next.js API Routes
   - Database: PostgreSQL (Neon free tier) + Prisma ORM
   - Auth: NextAuth.js (credentials provider)
   - Deploy: Vercel
5. **TASKS.md** — Pecah fitur PRD jadi task-task kecil. Satu task = 1 commit.
6. **AGENTS.md** — Aturan untuk AI agent. Contoh: "Jangan install package baru
   tanpa konfirmasi", "Pakai TypeScript strict", dll.

### 4. Review sendiri

Sebelum lanjut ke fase berikutnya, cek:

- PRD punya minimal 3 fitur MVP yang spesifik (bukan "fitur bagus")
- ARCHITECTURE.md mencantumkan stack yang sama dengan yang tertulis di atas
- TASKS.md punya minimal 5 task untuk fase 1

---

## Kesalahan Umum

| Kesalahan | Akibat | Solusi |
|---|---|---|
| Skip PRD, langsung coding | Agent bikin fitur random | Tulis PRD dulu, minimal 30 menit |
| ARCHITECTURE.md tidak sesuai paket | Agent install stack yang salah | Copy stack dari bagian 3 di atas |
| TASKS.md terlalu besar per task | Agent bingung, hasilnya berantakan | Pecah: 1 task = 1 file / 1 endpoint |
| AGENTS.md kosong | Agent pakai default behavior | Minimal tulis 5 aturan |
| Edit template asli, bukan copy | Template rusak untuk member lain | Selalu copy dulu |

---

## Checklist

- [ ] Folder project sudah dibuat dan `git init`
- [ ] 6 file dokumen sudah di-copy dari template
- [ ] PRD.md terisi lengkap (ringkasan, masalah, pengguna, fitur, batasan)
- [ ] SDLC.md terisi (branch strategy, commit convention)
- [ ] DESIGN.md terisi (warna, font, layout)
- [ ] ARCHITECTURE.md terisi dengan stack Paket A
- [ ] TASKS.md terisi minimal 5 task fase 1
- [ ] AGENTS.md terisi minimal 5 aturan
- [ ] Semua dokumen sudah di-commit: `git add . && git commit -m "docs: tambah 6 dokumen perencanaan"`
