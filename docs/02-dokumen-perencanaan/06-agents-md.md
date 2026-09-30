# 06 — Menulis AGENTS.md Project

## Tujuan

Aturan permanen yang dibaca agent setiap sesi. Ini "kontrak kerja" agent di
project kamu.

## Kenapa Penting

Chat hilang, memori agent terbatas, agent bisa diganti. `AGENTS.md` tetap.
Semua aturan yang kamu ulang-ulang di chat → pindahkan ke sini.

## Nama File per Agent

| Agent | File yang dibaca |
|---|---|
| Hermes, Codex, banyak agent modern | `AGENTS.md` |
| Claude Code | `CLAUDE.md` (atau `AGENTS.md`) |
| Cursor | `.cursorrules` atau `.cursor/rules/` |
| Windsurf | `.windsurfrules` |

Cara aman: tulis isi lengkap di `AGENTS.md`, lalu buat file lain yang isinya
"Baca AGENTS.md" untuk agent yang butuh nama berbeda.

## Template Isi

```markdown
# AGENTS.md

## Tentang Project
<satu paragraf: produk apa, untuk siapa>

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

## Aturan Keras
- JANGAN commit atau push tanpa izin eksplisit.
- JANGAN tulis secret/API key ke file yang masuk Git. Secret hanya di `.env`.
- JANGAN install dependency baru tanpa izin; cek dulu yang sudah ada.
- JANGAN ubah struktur folder tanpa izin.
- JANGAN jalankan perintah destruktif DB (drop, truncate, reset) tanpa izin.
- SELALU baca file sebelum mengeditnya.
- SELALU jalankan `npm run build` setelah perubahan besar.
- SELALU pakai token dari DESIGN.md untuk warna/spacing.

## Gaya Kode
- TypeScript strict, hindari `any`.
- Komponen: function component + named export.
- Nama file komponen: PascalCase. Utilitas: camelCase.
- Server-only code jangan diimpor ke client component.

## Definisi Selesai
Fitur selesai kalau: jalan di localhost, tidak ada error console,
state loading/empty/error ditangani, responsive 375px, build sukses.

## Bahasa
Jawab dalam bahasa Indonesia. Komentar kode boleh Inggris.
```

## Cara Merawat

Setiap kali kamu mengoreksi agent untuk hal yang sama dua kali, tambahkan
koreksi itu ke `AGENTS.md`. Dokumen ini harus tumbuh seiring project.

## Checklist

- [ ] `AGENTS.md` ada di root project
- [ ] Berisi stack, perintah, aturan keras, gaya kode, definisi selesai
- [ ] Agent membaca dan bisa merangkumnya
- [ ] Diupdate setiap ada aturan baru
