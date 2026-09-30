# G1 — Dokumen Perencanaan

> 🔵 MENENGAH · Target waktu: 3–4 jam · Output: 6 dokumen di folder `docs/`

**Aturan emas: AI agent tidak boleh menebak.** Kalau tidak ada dokumen, agent akan
mengarang arsitektur — dan kamu yang membersihkan kekacauannya di hari ke-3.

---

## Tujuan Sesi Ini

Selesai sesi ini kamu punya 6 dokumen yang jadi "kontrak" antara kamu dan AI agent:

| Dokumen | Menjawab pertanyaan |
|---|---|
| `PRD.md` | Apa yang dibangun, untuk siapa, fitur apa saja |
| `SDLC.md` | Urutan kerja, branch, definisi selesai |
| `DESIGN.md` | Warna, font, spacing, komponen |
| `ARCHITECTURE.md` | Stack, struktur folder, skema data, **deploy target** |
| `TASKS.md` | Pecahan pekerjaan yang bisa dicentang |
| `AGENTS.md` | Aturan main untuk AI agent |

---

## Langkah 1 — Siapkan Folder Project

```bash
mkdir produk-digital-railway
cd produk-digital-railway
git init
mkdir -p docs
```

**Output yang diharapkan:**
```
Initialized empty Git repository in .../produk-digital-railway/.git/
```

---

## Langkah 2 — Copy Template

Ambil template dari repo pedoman ini.

```bash
cp ../PEDOMAN\ MONOREPO/templates/PRD.md docs/01-PRD.md
cp ../PEDOMAN\ MONOREPO/templates/SDLC.md docs/02-SDLC.md
cp ../PEDOMAN\ MONOREPO/templates/DESIGN.md docs/03-DESIGN.md
cp ../PEDOMAN\ MONOREPO/templates/ARCHITECTURE.md docs/04-ARCHITECTURE.md
cp ../PEDOMAN\ MONOREPO/templates/TASKS.md docs/05-TASKS.md
cp ../PEDOMAN\ MONOREPO/templates/AGENTS.md AGENTS.md
```

**Output yang diharapkan:** tidak ada pesan apa pun (berhasil). Cek:

```bash
ls docs/
```
```
01-PRD.md  02-SDLC.md  03-DESIGN.md  04-ARCHITECTURE.md  05-TASKS.md
```

---

## Langkah 3 — Isi PRD

Aturan keras untuk kelas ini:

- Maksimal **5 fitur MVP**. Lebih dari itu, 3 hari tidak cukup.
- Setiap fitur punya **1 kalimat kriteria selesai** yang bisa diuji.
- Fitur yang tidak dikerjakan masuk bagian "Bukan Sekarang".

Contoh isian ringkas:

```markdown
## Fitur MVP
1. Katalog produk digital — pengunjung bisa melihat daftar produk + harga.
   Selesai jika: halaman `/produk` menampilkan data dari database.
2. Detail produk — halaman per produk.
   Selesai jika: `/produk/[slug]` buka tanpa error dan data cocok.
3. Form pesanan — pengunjung isi nama + email, data masuk DB.
   Selesai jika: submit form membuat 1 baris di tabel `Order`.
4. Admin login sederhana — 1 akun admin.
   Selesai jika: `/admin` hanya bisa dibuka setelah login.
5. Cron pengingat harian — email ringkasan pesanan tiap 08:00.
   Selesai jika: log menunjukkan job jalan pada jadwal.

## Bukan Sekarang
- Pembayaran otomatis
- Multi-bahasa
- Dashboard analitik
```

> 💡 Fitur nomor 5 (cron) itu **alasan kamu pilih Railway**. Kalau project-mu tidak
> punya fitur semacam ini, jujur saja: pindah ke Paket A.

---

## Langkah 4 — Isi ARCHITECTURE.md (bagian paling penting)

Bagian ini yang dibaca agent paling sering. Tulis eksplisit.

```markdown
## Tech Stack
- Framework: Next.js 15 (App Router, TypeScript)
- Styling: Tailwind CSS
- ORM: Prisma
- Database: PostgreSQL (service Railway, bukan eksternal)
- Auth: sesi cookie sederhana (iron-session)
- Deploy target: **Railway** (web service + Postgres service, 1 project)

## Alasan Pilih Railway (bukan Vercel)
Aplikasi butuh **persistent process**:
- Cron harian jalan di dalam proses Node (`node-cron`), bukan HTTP trigger.
- Rencana ke depan: worker antrian email dan WebSocket notifikasi.
Serverless Vercel mematikan proses setelah response, jadi tidak cocok.

## Struktur Folder
```
src/
├── app/
│   ├── (public)/produk/page.tsx
│   ├── (public)/produk/[slug]/page.tsx
│   ├── admin/page.tsx
│   └── api/
│       ├── health/route.ts
│       └── orders/route.ts
├── lib/
│   ├── prisma.ts        # singleton Prisma Client
│   ├── session.ts
│   └── cron.ts          # scheduler, dipanggil sekali saat boot
└── components/
prisma/
└── schema.prisma
```

## Skema Data (ringkas)
- Product: id, slug (unique), nama, deskripsi, harga (Int, rupiah), aktif (Bool)
- Order: id, productId (FK), namaPembeli, email, status (enum), createdAt
- AdminUser: id, email (unique), passwordHash

## Variabel Lingkungan
| Nama | Contoh | Wajib |
|---|---|---|
| DATABASE_URL | postgresql://... | ya |
| SESSION_SECRET | string 32+ karakter | ya |
| CRON_ENABLED | true / false | ya |
| SMTP_URL | smtp://... | untuk fitur cron email |

## Aturan Railway
- Jangan hardcode port. Pakai `process.env.PORT`.
- `DATABASE_URL` diambil dari variable reference Postgres service.
- Migrasi produksi pakai `prisma migrate deploy`, bukan `migrate dev`.
- Cron hanya aktif kalau `CRON_ENABLED=true` (supaya tidak dobel saat lokal).
```

---

## Langkah 5 — Isi DESIGN.md

Cukup token dasar. Jangan bikin design system 40 halaman.

```markdown
## Warna
- Primary: #1D4ED8
- Background: #FFFFFF / #0B1120 (dark)
- Teks: #0F172A / #E2E8F0
- Sukses: #15803D · Error: #B91C1C

## Font
- Heading & body: Inter (next/font)

## Spacing
Skala 4px: 4, 8, 12, 16, 24, 32, 48

## Radius
- Kartu: 12px · Tombol: 8px · Input: 8px

## Komponen Wajib
Button (primary/secondary/danger), Input, Card, Badge, Toast, EmptyState
```

---

## Langkah 6 — Isi TASKS.md

Pecah sampai setiap task bisa selesai dalam ≤45 menit.

```markdown
## Hari 2 — Build
- [ ] T01 Inisialisasi Next.js + Tailwind + TypeScript
- [ ] T02 Setup Prisma + koneksi PostgreSQL lokal
- [ ] T03 Tulis schema.prisma (Product, Order, AdminUser)
- [ ] T04 Migrasi pertama + seed 3 produk contoh
- [ ] T05 Halaman /produk (list dari DB)
- [ ] T06 Halaman /produk/[slug]
- [ ] T07 API POST /api/orders + validasi Zod
- [ ] T08 Form pesanan di halaman detail
- [ ] T09 Login admin + proteksi /admin
- [ ] T10 Route /api/health
- [ ] T11 lib/cron.ts + job pengingat harian

## Hari 3 — Deploy
- [ ] T12 Push ke GitHub
- [ ] T13 Railway project + Postgres service
- [ ] T14 Env vars + migrate deploy
- [ ] T15 Custom domain + Cloudflare DNS
- [ ] T16 Uji backup pg_dump
```

---

## Langkah 7 — Isi AGENTS.md

Tambahkan aturan khusus Railway di bawah template bawaan:

```markdown
## Aturan Khusus Paket G (Railway)

1. Deploy target adalah Railway, BUKAN Vercel. Jangan buat `vercel.json`.
2. Jangan pakai API serverless-only (mis. `@vercel/edge`).
3. Server harus dengarkan `process.env.PORT`.
4. Prisma Client dibuat singleton di `src/lib/prisma.ts`. Jangan `new PrismaClient()`
   di setiap file — proses hidup terus, koneksi bisa bocor.
5. Cron ditulis di `src/lib/cron.ts` dan hanya jalan bila `CRON_ENABLED === 'true'`.
6. Jangan pernah tulis nilai secret ke dalam kode atau ke file yang di-commit.
   Semua secret lewat environment variable.
7. Sebelum bilang "selesai", jalankan `npm run build` dan laporkan hasilnya.
8. Kalau butuh dependency baru, tanya dulu dan sebutkan alasannya.
```

---

## Langkah 8 — Commit Dokumen

```bash
git add .
git commit -m "docs: dokumen perencanaan Paket G (Railway fullstack)"
```

> Catatan kelas: perintah git di atas kamu jalankan **di project-mu sendiri**,
> bukan di repo pedoman ini.

---

## Error Umum

| Gejala | Sebab | Solusi |
|---|---|---|
| Agent bikin `vercel.json` | AGENTS.md tidak sebut deploy target | Tegaskan Railway di AGENTS.md + ARCHITECTURE.md |
| Fitur MVP membengkak jadi 12 | PRD tidak punya bagian "Bukan Sekarang" | Pindahkan sisa fitur ke sana, jangan dihapus |
| Agent bikin skema DB beda tiap sesi | Skema tidak ditulis di ARCHITECTURE.md | Tulis skema final, suruh agent ikuti |
| Cron jalan dobel di lokal | Tidak ada flag `CRON_ENABLED` | Tambah flag, default `false` di `.env.local` |

---

## Checklist Sebelum Lanjut

- [ ] 6 dokumen ada dan terisi (bukan masih template kosong)
- [ ] Fitur MVP ≤ 5, masing-masing punya kriteria selesai
- [ ] ARCHITECTURE.md menulis **Railway** sebagai deploy target
- [ ] Alasan butuh persistent process tertulis jelas
- [ ] Skema data final (nama tabel & kolom sudah fix)
- [ ] Tabel variabel lingkungan lengkap
- [ ] AGENTS.md berisi aturan khusus Railway
- [ ] Semua sudah di-commit

➡️ Lanjut ke **[02-ai-agent.md](02-ai-agent.md)**.
