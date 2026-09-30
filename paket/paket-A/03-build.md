# 03 — Build: Next.js + Prisma + Neon + NextAuth

## Tujuan

Aplikasi berjalan di localhost dengan halaman, database terhubung, dan login
berfungsi. Semua dikerjakan oleh AI agent berdasarkan dokumen perencanaan.

## Sebelum Mulai

- Antigravity sudah punya konteks 6 dokumen ([`02-ai-agent.md`](02-ai-agent.md))
- Akun Neon sudah dibuat ([neon.tech](https://neon.tech))

Referensi teknis:
- [`../../docs/04-build-frontend/01-build-nextjs-dari-nol.md`](../../docs/04-build-frontend/01-build-nextjs-dari-nol.md)
- [`../../docs/05-build-backend/01-api-dan-validasi.md`](../../docs/05-build-backend/01-api-dan-validasi.md)
- [`../../docs/05-build-backend/02-auth-dan-authorization.md`](../../docs/05-build-backend/02-auth-dan-authorization.md)
- [`../../docs/06-database/01-postgresql-prisma.md`](../../docs/06-database/01-postgresql-prisma.md)

---

## Langkah 1 — Scaffold Next.js

Kirim prompt ini ke Antigravity:

```
Buat project Next.js 14 dengan App Router di folder ini.

Konfigurasi wajib:
- TypeScript strict mode
- Tailwind CSS
- ESLint
- App Router (bukan Pages Router)
- src/ directory
- Import alias: @/*

Setelah selesai, jalankan npm install dan pastikan tidak ada error.
Jangan buat halaman apa pun dulu, cukup scaffold dasar.
```

**Expected output:**
```
my-project/
├── src/
│   └── app/
│       ├── layout.tsx
│       ├── page.tsx
│       └── globals.css
├── public/
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── .eslintrc.json
```

Verifikasi manual:
```bash
npm run dev
```
Buka `http://localhost:3000` — harus muncul halaman default Next.js.

Tekan `Ctrl+C` untuk stop.

---

## Langkah 2 — Buat database Neon

Ini dikerjakan **manual**, bukan oleh agent (agent tidak punya akses dashboard).

1. Login ke [console.neon.tech](https://console.neon.tech)
2. **Create Project**
   - Name: nama project kamu
   - Region: pilih yang terdekat (Singapore kalau ada)
   - PostgreSQL version: default
3. Setelah dibuat, klik **Connection String**
4. Pilih **Prisma** dari dropdown
5. Copy connection string — bentuknya:
   ```
   postgresql://USER:PASSWORD@HOST/DBNAME?sslmode=require
   ```

> 🔒 **Jangan pernah commit connection string ke Git.** Simpan di `.env` saja.

### Buat file `.env`

```bash
touch .env
```

Isi `.env`:
```env
DATABASE_URL="<PASTE_CONNECTION_STRING_NEON_DI_SINI>"
NEXTAUTH_SECRET="<GENERATE_DENGAN_COMMAND_DI_BAWAH>"
NEXTAUTH_URL="http://localhost:3000"
```

Generate `NEXTAUTH_SECRET`:
```bash
openssl rand -base64 32
```

Pastikan `.env` masuk `.gitignore`:
```bash
grep -q "^.env$" .gitignore || echo ".env" >> .gitignore
```

Verifikasi `.env` tidak akan ter-commit:
```bash
git check-ignore -v .env
```
**Expected output:** `.gitignore:NN:.env	.env` — kalau kosong, `.env` BELUM
di-ignore. Perbaiki sebelum lanjut.

---

## Langkah 3 — Setup Prisma

Kirim prompt ini:

```
Setup Prisma ORM di project ini:

1. Install prisma dan @prisma/client
2. Jalankan npx prisma init
3. Set provider ke postgresql, datasource pakai env("DATABASE_URL")
4. Buat schema sesuai bagian "Database Schema" di ARCHITECTURE.md
5. Wajib ada model User dengan field:
   - id (String, @id, @default(cuid()))
   - email (String, @unique)
   - password (String) — akan menyimpan hash, bukan plaintext
   - name (String?)
   - createdAt (DateTime, @default(now()))
   - updatedAt (DateTime, @updatedAt)
6. Buat file src/lib/prisma.ts dengan singleton PrismaClient
   (supaya tidak bikin koneksi baru setiap hot reload)

Jangan jalankan migrate dulu. Tunggu konfirmasi saya.
```

**Expected output:** file `prisma/schema.prisma` dan `src/lib/prisma.ts`.

Setelah review schema, jalankan:
```bash
npx prisma db push
```

**Expected output:**
```
Your database is now in sync with your Prisma schema. Done in XXXms
```

Kalau error `Can't reach database server` → connection string salah atau salah
copy. Cek ulang dari dashboard Neon.

Verifikasi tabel sudah dibuat:
```bash
npx prisma studio
```
Buka `http://localhost:5555` — harus muncul tabel `User`.

---

## Langkah 4 — Setup NextAuth (credentials)

Kirim prompt ini:

```
Setup NextAuth.js dengan Credentials provider:

1. Install next-auth dan bcryptjs (+ @types/bcryptjs)
2. Buat src/lib/auth.ts berisi authOptions:
   - CredentialsProvider dengan field email dan password
   - authorize(): cari user by email via Prisma, bandingkan password
     pakai bcrypt.compare, return user tanpa field password
   - session strategy: "jwt"
   - pages.signIn: "/login"
3. Buat route handler src/app/api/auth/[...nextauth]/route.ts
4. Buat API route POST /api/register:
   - validasi email format dan password minimal 8 karakter
   - cek email belum terdaftar
   - hash password pakai bcrypt dengan salt rounds 10
   - simpan user, return user tanpa password
5. Buat halaman /login dan /register dengan form sesuai DESIGN.md
6. Buat src/app/providers.tsx berisi SessionProvider, pakai di layout.tsx
7. Buat halaman /dashboard yang hanya bisa diakses kalau sudah login

WAJIB:
- Password disimpan sebagai hash bcrypt, TIDAK PERNAH plaintext
- Jangan pernah return field password dari API mana pun
- Validasi input di server, bukan hanya di client
```

**Expected output:**
```
src/
├── app/
│   ├── api/
│   │   ├── auth/[...nextauth]/route.ts
│   │   └── register/route.ts
│   ├── login/page.tsx
│   ├── register/page.tsx
│   ├── dashboard/page.tsx
│   ├── providers.tsx
│   └── layout.tsx
└── lib/
    ├── auth.ts
    └── prisma.ts
```

---

## Langkah 5 — Build halaman sesuai TASKS.md

Kerjakan task satu per satu. **Jangan** kirim semua task sekaligus.

Prompt per task:

```
Kerjakan task nomor [N] dari TASKS.md.

Aturan:
- Ikuti DESIGN.md untuk warna, font, spacing
- Responsive: mobile-first, breakpoint sm/md/lg
- TypeScript strict, tidak ada `any`
- Validasi input di server untuk semua API route
- Setelah selesai, jalankan npm run lint dan perbaiki semua error

Laporkan file apa saja yang kamu ubah.
```

Setelah setiap task selesai:
```bash
npm run lint
npm run build
git add .
git commit -m "feat: [deskripsi task]"
```

Kalau `npm run build` gagal → jangan lanjut task berikutnya. Perbaiki dulu.

---

## Kesalahan Umum

| Error | Penyebab | Solusi |
|---|---|---|
| `Can't reach database server` | `DATABASE_URL` salah | Copy ulang dari Neon, pastikan `?sslmode=require` ada |
| `PrismaClient is unable to run in browser` | Prisma dipakai di Client Component | Pindahkan ke Server Component atau API route |
| `NEXTAUTH_SECRET` warning | Env var belum diset | Generate dengan `openssl rand -base64 32` |
| `Too many connections` di Neon | PrismaClient dibuat berulang | Pakai singleton di `src/lib/prisma.ts` |
| Login selalu gagal | Password disimpan plaintext, dibandingkan dengan hash | Pastikan register pakai `bcrypt.hash` |
| Agent kirim semua task sekaligus | Prompt terlalu besar | Satu task per prompt |

Troubleshooting lengkap: [`../../docs/15-troubleshooting/02-decision-tree.md`](../../docs/15-troubleshooting/02-decision-tree.md)

---

## Checklist

- [ ] Next.js 14 + TypeScript + Tailwind ter-scaffold
- [ ] `npm run dev` jalan, `localhost:3000` terbuka
- [ ] Project Neon dibuat, connection string didapat
- [ ] `.env` dibuat dan sudah masuk `.gitignore` (dicek dengan `git check-ignore -v .env`)
- [ ] `NEXTAUTH_SECRET` di-generate, bukan ditulis manual
- [ ] Prisma schema dibuat, `npx prisma db push` sukses
- [ ] `npx prisma studio` menampilkan tabel `User`
- [ ] NextAuth credentials provider terpasang
- [ ] Register berhasil, password tersimpan sebagai hash bcrypt (cek di Prisma Studio)
- [ ] Login berhasil, redirect ke `/dashboard`
- [ ] `/dashboard` tidak bisa diakses tanpa login
- [ ] Semua task fase 1 dari TASKS.md selesai
- [ ] `npm run lint` — 0 error
- [ ] `npm run build` — sukses
