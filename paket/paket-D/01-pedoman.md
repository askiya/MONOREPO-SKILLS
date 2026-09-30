# 01 — Dokumen Perencanaan (Paket D)

## Tujuan

Kamu punya 6 dokumen perencanaan lengkap: PRD, SDLC, DESIGN, ARCHITECTURE,
TASKS, AGENTS — disesuaikan untuk arsitektur Next.js + Supabase + Vercel.

---

## Sebelum Mulai

- [ ] Sudah baca [`../../docs/02-dokumen-perencanaan/01-prd.md`](../../docs/02-dokumen-perencanaan/01-prd.md)
- [ ] Sudah lihat contoh terisi di [`../../examples/toko-produk-digital/`](../../examples/toko-produk-digital/)
- [ ] Punya folder project kosong

---

## Langkah

### 1. Buat Folder Project

```bash
mkdir my-supabase-app
cd my-supabase-app
git init
```

### 2. Copy Template Dokumen

```bash
cp ../../templates/PRD.md ./PRD.md
cp ../../templates/SDLC.md ./SDLC.md
cp ../../templates/DESIGN.md ./DESIGN.md
cp ../../templates/ARCHITECTURE.md ./ARCHITECTURE.md
cp ../../templates/TASKS.md ./TASKS.md
cp ../../templates/AGENTS.md ./AGENTS.md
cp ../../templates/env.example ./.env.example
```

### 3. Isi PRD

| Bagian | Yang kamu tulis |
|---|---|
| Ringkasan Produk | Deskripsi 1 kalimat, siapa penggunanya |
| Fitur Utama | List fitur MVP — **sebutkan mana yang butuh login** |
| User Role | Siapa saja yang login? (user, admin, guest) |
| Data yang Disimpan | Tabel apa saja, data siapa milik siapa |
| File yang Diupload | Avatar? Dokumen? Gambar produk? Ukuran max? |
| Stack | Next.js + Supabase (Auth + DB + Storage) + Vercel |
| Batasan | Serverless timeout, tidak ada background worker berat |

**Penting untuk Paket D:** tulis eksplisit **siapa boleh lihat data siapa**.
Ini nanti jadi dasar Row Level Security policy.

Contoh:
```markdown
## Aturan Akses Data
- User hanya bisa lihat & edit data miliknya sendiri (`user_id = auth.uid()`)
- Admin bisa lihat semua data
- Produk publik bisa dilihat semua orang tanpa login
- Order hanya bisa dilihat pembeli dan admin
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/01-prd.md`](../../docs/02-dokumen-perencanaan/01-prd.md)

### 4. Isi SDLC

Timeline khas Paket D:

```text
Hari 1: Perencanaan + setup Cursor + buat Supabase project + skema DB + RLS
Hari 2: Build frontend + auth flow + storage upload + tes localhost
Hari 3: Deploy Vercel + env vars + custom domain + monitoring
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/02-sdlc.md`](../../docs/02-dokumen-perencanaan/02-sdlc.md)

### 5. Isi DESIGN

Untuk Paket D, desain harus mencakup **halaman auth**:

| Halaman | Isi |
|---|---|
| `/` | Landing page publik |
| `/login` | Form email + password, tombol OAuth |
| `/register` | Form daftar |
| `/dashboard` | Halaman setelah login (protected) |
| `/profile` | Edit profil + upload avatar |

State yang harus didesain:
- Loading (saat auth check berjalan)
- Logged out (tampilkan tombol login)
- Logged in (tampilkan nama user + logout)
- Error (email sudah terdaftar, password salah, dll)

Panduan detail: [`../../docs/02-dokumen-perencanaan/03-design-md.md`](../../docs/02-dokumen-perencanaan/03-design-md.md)

### 6. Isi ARCHITECTURE

Arsitektur khas Paket D:

```text
┌──────────────────────────────────┐
│  Next.js (App Router)            │
│  ├─ Server Components  → @supabase/ssr (server client)
│  ├─ Client Components  → @supabase/ssr (browser client)
│  ├─ Server Actions     → mutations
│  └─ middleware.ts      → refresh session cookie
└────────────┬─────────────────────┘
             │
┌────────────▼─────────────────────┐
│  Supabase                        │
│  ├─ auth.users (built-in)        │
│  ├─ public.profiles (FK ke users)│
│  ├─ public.<tabel_app>           │
│  ├─ Storage bucket: avatars      │
│  └─ RLS policy di setiap tabel   │
└──────────────────────────────────┘
```

Yang **wajib** ditulis di ARCHITECTURE:

```markdown
## Aturan Keamanan Supabase
1. RLS WAJIB aktif di semua tabel di schema `public`
2. `NEXT_PUBLIC_SUPABASE_ANON_KEY` boleh di client — aman karena RLS melindungi
3. `SUPABASE_SERVICE_ROLE_KEY` HANYA di server — BYPASS RLS, jangan pernah ke client
4. Semua akses data user harus lewat RLS policy, bukan filter di kode aplikasi
5. Storage bucket punya policy sendiri (terpisah dari tabel)

## Tabel & Policy
| Tabel | SELECT | INSERT | UPDATE | DELETE |
|---|---|---|---|---|
| profiles | own row + public fields | own row | own row | — |
| products | semua (public) | admin | admin | admin |
| orders | own + admin | own | — | — |
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/04-architecture-md.md`](../../docs/02-dokumen-perencanaan/04-architecture-md.md)

### 7. Isi TASKS

```markdown
## Fase 1: Setup
- [ ] Buat project Next.js + TypeScript + Tailwind
- [ ] Buat project Supabase di dashboard
- [ ] Install @supabase/supabase-js dan @supabase/ssr
- [ ] Setup .env.local dengan URL + anon key
- [ ] Buat supabase client (browser + server)

## Fase 2: Database
- [ ] Buat tabel profiles (trigger auto-create saat register)
- [ ] Buat tabel aplikasi sesuai PRD
- [ ] Aktifkan RLS di semua tabel
- [ ] Tulis RLS policy per tabel
- [ ] Test policy dengan user berbeda

## Fase 3: Auth
- [ ] Halaman /login (email + password)
- [ ] Halaman /register
- [ ] OAuth provider (Google / GitHub)
- [ ] middleware.ts untuk refresh session
- [ ] Protected route /dashboard
- [ ] Logout

## Fase 4: Storage
- [ ] Buat bucket 'avatars' di Supabase
- [ ] Storage policy (user hanya bisa upload ke folder sendiri)
- [ ] Komponen upload avatar
- [ ] Tampilkan gambar dari Supabase Storage

## Fase 5: Deploy
- [ ] Push ke GitHub
- [ ] Import ke Vercel
- [ ] Isi env vars di Vercel
- [ ] Tambah Vercel URL ke Supabase redirect URLs
- [ ] Custom domain + DNS
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/05-tasks-md.md`](../../docs/02-dokumen-perencanaan/05-tasks-md.md)

### 8. Isi AGENTS

```markdown
## Agent: Cursor

### Boleh
- Baca/tulis file di project
- Jalankan terminal (npm, supabase CLI)
- Refactor komponen

### Tidak Boleh
- Menulis SUPABASE_SERVICE_ROLE_KEY di file client component
- Membuat tabel tanpa RLS policy
- Menggunakan service role key untuk operasi yang bisa pakai anon key + RLS
- Commit .env.local

### Aturan Wajib
1. Setiap tabel baru → langsung buat RLS policy
2. Client component pakai createBrowserClient, server pakai createServerClient
3. Gunakan @supabase/ssr, bukan @supabase/auth-helpers-nextjs (deprecated)
4. Semua secret via process.env, tidak pernah hardcode
```

Panduan detail: [`../../docs/02-dokumen-perencanaan/06-agents-md.md`](../../docs/02-dokumen-perencanaan/06-agents-md.md)

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| RLS lupa dibuat | ARCHITECTURE tidak menyebut RLS | Tulis tabel policy matrix di ARCHITECTURE |
| Agent pakai service role key di client | AGENTS tidak melarang eksplisit | Tulis larangan di AGENTS.md |
| Aturan akses data tidak jelas | PRD tidak menyebut siapa lihat apa | Isi bagian "Aturan Akses Data" di PRD |
| Terlalu banyak fitur | Ambisius untuk 3 hari | Batasi: auth + 1 fitur utama + upload |

---

## Checklist

- [ ] PRD terisi, ada bagian **Aturan Akses Data**
- [ ] SDLC ada timeline 3 hari
- [ ] DESIGN mencakup halaman login, register, dashboard, profile
- [ ] DESIGN mencakup state loading / logged out / logged in / error
- [ ] ARCHITECTURE ada **tabel policy matrix** untuk RLS
- [ ] ARCHITECTURE menyebut larangan service role key di client
- [ ] TASKS dipecah per fase (setup, database, auth, storage, deploy)
- [ ] AGENTS ada aturan wajib RLS dan larangan secret di client
- [ ] `.env.local` sudah masuk `.gitignore`
