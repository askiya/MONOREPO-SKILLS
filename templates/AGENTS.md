# AGENTS.md

## Tentang Project

<!-- satu paragraf: produk apa, untuk siapa -->

## Stack

- Next.js 14 App Router + TypeScript
- Tailwind CSS
- Prisma + PostgreSQL
- Deploy: Vercel (staging), VPS Coolify (produksi)

## Perintah

| Tujuan | Perintah |
|---|---|
| Dev server | `npm run dev` |
| Build | `npm run build` |
| Lint | `npm run lint` |
| Test | `npm run test` |
| Migrasi DB | `npx prisma migrate dev` |
| Studio DB | `npx prisma studio` |

## Aturan Keras

- JANGAN commit atau push tanpa izin eksplisit.
- JANGAN tulis secret/API key ke file yang masuk Git. Secret hanya di `.env`.
- JANGAN install dependency baru tanpa izin; cek dulu yang sudah terpasang.
- JANGAN ubah struktur folder tanpa izin.
- JANGAN jalankan perintah destruktif DB (drop, truncate, reset) tanpa izin.
- JANGAN pakai `any` di TypeScript sebagai jalan pintas.
- SELALU baca file sebelum mengeditnya.
- SELALU jalankan `npm run build` setelah perubahan besar.
- SELALU pakai token dari DESIGN.md untuk warna/spacing.
- SELALU tanya kalau informasi kurang — jangan menebak.

## Gaya Kode

- TypeScript strict.
- Komponen: function component + named export.
- Nama file komponen: PascalCase. Utilitas: camelCase.
- Server-only code jangan diimpor ke client component.
- Validasi input di server, bukan hanya client.

## Definisi Selesai

Fitur selesai kalau:
- jalan di localhost,
- tidak ada error merah di console,
- state loading/empty/error ditangani,
- responsive di 375px,
- `npm run build` sukses.

## Bahasa

Jawab dalam bahasa Indonesia. Komentar kode boleh bahasa Inggris.
