# 05 — Menulis TASKS.md

## Tujuan

Memecah PRD jadi potongan kerja kecil yang bisa dikerjakan agent satu per satu
dan bisa dicentang.

## Format

```markdown
# TASKS.md

## Legenda
- [ ] belum  |  [~] sedang  |  [x] selesai
- Prioritas: P0 (blocker), P1 (penting), P2 (nanti)

## Fase 1 — Fondasi

- [x] T-001 P0 Inisialisasi Next.js + Tailwind + TypeScript
      Selesai kalau: `npm run dev` jalan, halaman kosong tampil
- [x] T-002 P0 Setup Prisma + koneksi PostgreSQL
      Selesai kalau: `npx prisma db push` sukses
- [ ] T-003 P0 Komponen UI dasar: Button, Input, Card
      Selesai kalau: 3 komponen ada di components/ui, sesuai DESIGN.md

## Fase 2 — Auth

- [ ] T-010 P0 Halaman /register
      File: app/(public)/register/page.tsx, app/api/auth/register/route.ts
      Selesai kalau: user baru tersimpan di DB, password ter-hash
- [ ] T-011 P0 Halaman /login + session
      Selesai kalau: login benar → redirect /dashboard; salah → pesan error
- [ ] T-012 P1 Middleware proteksi route member
      Selesai kalau: akses /dashboard tanpa login → redirect /login
```

## Aturan Menulis Task

1. Satu task = satu hasil yang bisa dilihat/dites.
2. Selalu tulis **"Selesai kalau:"** — ini kontrak dengan agent.
3. Sebutkan file kalau sudah tahu.
4. Beri nomor stabil (T-010). Jangan renumber; nanti referensi rusak.
5. Maksimal 90 menit per task. Lebih dari itu, pecah.

## Task Buruk vs Baik

Buruk:
```
- [ ] Bikin auth
```

Baik:
```
- [ ] T-010 P0 Endpoint POST /api/auth/register
      Input: { email, password, name }
      Validasi: email format, password min 8 karakter, email unik
      Output sukses: 201 { user: { id, email, name } }
      Output gagal: 400 dengan pesan field spesifik
      Password di-hash dengan bcrypt, tidak pernah dikembalikan di response
      Selesai kalau: 3 kasus di atas terbukti via test/manual
```

## Alur Harian

```
1. Buka TASKS.md, pilih task P0 teratas yang [ ]
2. Ubah jadi [~]
3. Kirim ke agent: "Kerjakan T-010 saja..."
4. Verifikasi "Selesai kalau"
5. Lolos → [x] + commit
```

## Checklist

- [ ] Semua fitur PRD punya task turunan
- [ ] Setiap task punya "Selesai kalau"
- [ ] Nomor task unik dan stabil
- [ ] Prioritas terisi
