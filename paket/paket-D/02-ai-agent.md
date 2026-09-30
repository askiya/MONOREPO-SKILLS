# 02 — Setup AI Agent: Cursor (Paket D)

## Tujuan

Cursor terpasang, folder project terbuka, terminal terintegrasi berfungsi,
dan agent sudah memahami aturan Supabase (RLS + secret handling).

---

## Sebelum Mulai

- [ ] Node.js LTS terinstal (`node -v`)
- [ ] Git terinstal (`git --version`)
- [ ] Folder project dari [01-pedoman.md](01-pedoman.md) sudah ada

---

## Kenapa Cursor untuk Paket D

| Alasan | Penjelasan |
|---|---|
| Editor-first | Berbasis VS Code — familiar, extension jalan |
| Multi-file edit | Bisa edit beberapa file sekaligus (berguna untuk auth flow) |
| Codebase context | `@codebase` memberi agent konteks seluruh project |
| Terminal terintegrasi | Jalankan `npx supabase` tanpa pindah window |
| Rules file | `.cursorrules` / Project Rules bisa memaksa aturan RLS |

> Kalau kamu lebih nyaman Antigravity, boleh. Paket menyebut agent yang
> paling cocok, bukan wajib. Lihat [`../../docs/01-setup-ai-agent/02-alternatif-ai-agent.md`](../../docs/01-setup-ai-agent/02-alternatif-ai-agent.md)

---

## Langkah

### 1. Instal Cursor

1. Buka situs resmi Cursor: [cursor.com](https://cursor.com)
2. Download installer sesuai OS (Windows / macOS / Linux)
3. Jalankan installer dengan opsi standar
4. Buka Cursor
5. Login / daftar akun
6. Pilih model AI yang tersedia di plan kamu

> UI, nama menu, dan batas plan Cursor bisa berubah. Selalu cocokkan dengan
> dokumentasi resmi Cursor saat kamu mengikuti panduan ini.

**Opsional — import setting VS Code:**
Saat pertama buka, Cursor menawarkan import extension & keybinding dari VS Code.
Kalau kamu sudah pakai VS Code, pilih **Import** agar setting tidak hilang.

### 2. Buka Folder Project

1. **File** → **Open Folder**
2. Arahkan ke folder project kamu (`my-supabase-app`)
3. Klik **Open**

> Buka **folder project**, bukan drive atau folder induk yang berisi banyak
> project. Konteks agent jadi kotor dan respons melambat.

**Berhasil kalau:** sidebar kiri menampilkan file project (`PRD.md`, `ARCHITECTURE.md`, dll).

### 3. Buka Terminal Terintegrasi

Cara membuka:

| OS | Shortcut |
|---|---|
| Windows / Linux | `Ctrl` + `` ` `` (backtick) |
| macOS | `Cmd` + `` ` `` |

Atau: menu **Terminal** → **New Terminal**

Tes terminal:

```bash
node -v
git --version
pwd
```

**Output yang benar:**
```text
v20.x.x        (atau versi LTS lain)
git version 2.x.x
/path/ke/my-supabase-app
```

**Berhasil kalau:** `pwd` menunjukkan folder project kamu, bukan home directory.

### 4. Tes Agent Dasar

Buka chat agent:

| OS | Shortcut |
|---|---|
| Windows / Linux | `Ctrl` + `L` (chat) / `Ctrl` + `I` (inline edit) |
| macOS | `Cmd` + `L` / `Cmd` + `I` |

Kirim prompt:

```text
Baca folder ini. Buat file hello.txt berisi "Cursor siap".
Lalu baca kembali file tersebut dan laporkan isinya.
```

**Berhasil kalau:** `hello.txt` muncul di sidebar dengan isi tepat.

### 5. Berikan Konteks Project

Kirim prompt ini dengan `@codebase` agar agent baca seluruh project:

```text
@codebase

Baca PRD.md, ARCHITECTURE.md, dan AGENTS.md.

Ini project Next.js + Supabase + Vercel. Aturan wajib:

1. Gunakan @supabase/ssr untuk integrasi Next.js App Router.
   JANGAN gunakan @supabase/auth-helpers-nextjs (deprecated).
2. Client component → createBrowserClient
   Server component / Server Action / Route Handler → createServerClient
3. NEXT_PUBLIC_SUPABASE_URL dan NEXT_PUBLIC_SUPABASE_ANON_KEY
   boleh diakses dari client (aman karena dilindungi RLS).
4. SUPABASE_SERVICE_ROLE_KEY HANYA boleh di server-side code.
   Key ini BYPASS Row Level Security. Jangan pernah ada di client component,
   jangan pernah diberi prefix NEXT_PUBLIC_.
5. SETIAP tabel baru di schema public WAJIB punya RLS enabled + policy.
   Tabel tanpa RLS = data terbuka untuk semua orang yang punya anon key.
6. Jangan pernah hardcode secret. Semua via process.env.
7. Jangan pernah commit .env.local.

Konfirmasi kamu memahami aturan ini, lalu rangkum poin 4 dan 5 dengan kata-katamu.
```

**Berhasil kalau:** agent merangkum bahaya service role key dan kewajiban RLS
dengan benar.

### 6. Buat Project Rules (Sangat Disarankan)

Agar aturan di atas berlaku otomatis di setiap percakapan, buat file
`.cursorrules` di root project:

```text
# Project: Next.js + Supabase + Vercel

## Stack
- Next.js App Router + TypeScript + Tailwind CSS
- Supabase: Auth + PostgreSQL + Storage
- Deploy: Vercel

## Aturan Supabase (WAJIB)
- Gunakan @supabase/ssr. JANGAN @supabase/auth-helpers-nextjs (deprecated).
- Client component: createBrowserClient
- Server component / Server Action / Route Handler: createServerClient
- SUPABASE_SERVICE_ROLE_KEY hanya di server. BYPASS RLS. Jangan pernah NEXT_PUBLIC_.
- Setiap tabel baru di schema public WAJIB: ALTER TABLE ... ENABLE ROW LEVEL SECURITY
  plus minimal satu policy. Tabel tanpa RLS = data bocor.
- Storage bucket WAJIB punya policy sendiri.

## Aturan Umum
- TypeScript strict. Jangan pakai `any` tanpa alasan tertulis.
- Semua secret via process.env. Jangan hardcode.
- Jangan commit .env.local, .env.production.
- Validasi input di server side, bukan hanya client.
```

> Di versi Cursor yang lebih baru, ini bisa juga diatur lewat
> **Settings → Rules → Project Rules**. Cek dokumentasi resmi untuk lokasi
> terbaru; isi aturannya tetap sama.

### 7. Setup .gitignore

```bash
# Pastikan secret tidak pernah ikut commit
echo ".env.local" >> .gitignore
echo ".env*.local" >> .gitignore
echo ".vercel" >> .gitignore
echo "node_modules" >> .gitignore
```

Verifikasi:

```bash
git status --short
```

**Output yang benar:** `.env.local` TIDAK muncul di daftar file untracked.

### 8. Hapus File Tes

```bash
rm hello.txt
```

---

## Fitur Cursor yang Berguna untuk Paket D

| Fitur | Cara pakai | Kapan berguna |
|---|---|---|
| `@codebase` | Ketik di chat | Agent perlu konteks seluruh project |
| `@file` | `@app/page.tsx` | Fokus ke file tertentu |
| `@docs` | `@Supabase` | Agent baca dokumentasi resmi Supabase |
| Inline edit | `Ctrl`/`Cmd` + `K` | Ubah blok kode terpilih |
| Terminal | `Ctrl`/`Cmd` + `` ` `` | Jalankan `npx supabase`, `npm run dev` |
| Composer / Agent mode | `Ctrl`/`Cmd` + `I` | Edit banyak file sekaligus (auth flow) |

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Agent pakai `auth-helpers-nextjs` | Training data lama | Kirim ulang aturan, atau tambahkan `@docs Supabase` di prompt |
| Agent taruh service role key di client | Tidak ada rules file | Buat `.cursorrules` (langkah 6) |
| Agent buat tabel tanpa RLS | Tidak diberi aturan | Tulis RLS wajib di `.cursorrules` |
| Terminal `pwd` bukan folder project | Folder salah yang dibuka | Tutup, Open Folder ulang |
| Agent respons lambat / tidak relevan | Folder yang dibuka terlalu luas | Buka folder project saja |
| `.env.local` muncul di `git status` | `.gitignore` belum berisi | Ulangi langkah 7 |

---

## Checklist

- [ ] Cursor terinstal dan bisa dibuka
- [ ] Folder project terbuka (sidebar menampilkan `PRD.md`, `ARCHITECTURE.md`)
- [ ] Terminal terintegrasi berfungsi, `pwd` = folder project
- [ ] Agent bisa buat dan baca file (`hello.txt` test lulus)
- [ ] Agent sudah diberi konteks aturan Supabase (RLS + service role key)
- [ ] File `.cursorrules` dibuat di root project
- [ ] `.env.local` ada di `.gitignore` dan tidak muncul di `git status`
- [ ] File tes `hello.txt` sudah dihapus
