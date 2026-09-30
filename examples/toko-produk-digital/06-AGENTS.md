# AGENTS.md — SantriLearn

## Tentang Project

Toko produk digital (e-book, video course, template) untuk santri belajar IT.
Member beli produk → akses otomatis setelah pembayaran terverifikasi.

## Stack

- Next.js 14 App Router + TypeScript
- Tailwind CSS
- Prisma + PostgreSQL (Neon)
- NextAuth.js credentials
- Deploy: Vercel (staging)

## Perintah

| Tujuan | Perintah |
|---|---|
| Dev server | `npm run dev` |
| Build | `npm run build` |
| Lint | `npm run lint` |
| Test | `npm run test` |
| Migrasi DB | `npx prisma migrate dev` |
| Seed | `npx prisma db seed` |
| Studio DB | `npx prisma studio` |

## Aturan Keras

- JANGAN commit atau push tanpa izin eksplisit.
- JANGAN tulis secret/API key ke file yang masuk Git. Secret hanya di `.env`.
- JANGAN install dependency baru tanpa izin.
- JANGAN ubah struktur folder tanpa izin.
- JANGAN jalankan `prisma migrate reset` atau perintah destruktif DB.
- JANGAN pakai `any` sebagai jalan pintas TypeScript.
- JANGAN percaya redirect browser sebagai bukti pembayaran — hanya webhook
  tervalidasi yang boleh mengaktifkan akses.
- SELALU baca file sebelum mengeditnya.
- SELALU validasi input di server, bukan hanya client.
- SELALU pakai token warna/spacing dari DESIGN.md.
- SELALU jalankan `npm run build` setelah perubahan besar.
- SELALU tanya kalau informasi kurang — jangan menebak.

## Gaya Kode

- TypeScript strict.
- Function component + named export.
- File komponen PascalCase, utilitas camelCase.
- Prisma client dari `lib/db.ts` (singleton), jangan `new PrismaClient()` per file.
- Server-only code jangan diimpor ke client component.
- Uang disimpan integer rupiah, bukan float.

## Definisi Selesai

- Jalan di localhost
- Tidak ada error merah di console
- Loading / empty / error state ditangani
- Responsive 375px
- `npm run build` sukses

## Bahasa

Jawab dalam bahasa Indonesia. Komentar kode boleh Inggris.
