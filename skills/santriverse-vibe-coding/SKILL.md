---
name: santriverse-vibe-coding
description: "Use when building an app from Santriverse PRD/SDLC/DESIGN/ARCHITECTURE/TASKS files. Implements one task at a time with verification gates."
version: 1.0.0
author: Santriverse
license: MIT
---

# Santriverse Vibe Coding

## Kapan Dipakai

Aktifkan skill ini ketika project punya `PRD.md`, `SDLC.md`, `DESIGN.md`,
`ARCHITECTURE.md`, dan `TASKS.md`, lalu user minta build/fix/review/deploy.

## Urutan Wajib

1. Baca `AGENTS.md` project; aturannya paling tinggi.
2. Baca lima dokumen konteks.
3. Jalankan `git status`; jangan timpa perubahan user.
4. Pilih satu task belum selesai dari `TASKS.md`.
5. Baca definisi dan semua usage sebelum mengedit simbol.
6. Buat diff minimum sesuai stack yang sudah dipilih.
7. Verifikasi dengan test relevan, lint, lalu build.
8. Laporkan file berubah, output verifikasi nyata, dan sisa masalah.
9. Jangan commit, push, migrasi produksi, atau deploy tanpa izin eksplisit.

## Aturan Keras

- Dokumen repo adalah sumber kebenaran; jangan mengarang keputusan kosong.
- Kalau dokumen konflik, berhenti dan minta keputusan.
- Satu task per putaran. Jangan "sekalian" refactor di luar scope.
- Jangan install dependency bila native/stdlib/dependency lama cukup.
- Secret hanya environment variable; jangan baca/print `.env`.
- Perintah destruktif DB/Git/server wajib persetujuan.
- Frontend data wajib punya loading, empty, error, success.
- UI wajib diuji pada 375px dan desktop; kontrol harus berfungsi atau dihapus.
- Backend wajib validasi input dan authorization di server.
- Payment webhook wajib signature verification, amount/reference match,
  idempotency, dan transaksi DB atomik.

## Prompt Internal Sebelum Mulai

Jawab diam-diam:

- Apa task tunggalnya?
- Apa definisi selesai?
- File mana yang relevan?
- Test minimum apa yang membuktikan selesai?
- Apakah tindakan berisiko/merusak data?

Kalau salah satu tidak jelas, cari di repo atau tanya user.

## Gate Selesai

Task hanya boleh ditandai selesai ketika:

```text
[ ] Implementasi sesuai definisi task
[ ] Test relevan lulus
[ ] Lint lulus
[ ] Build lulus
[ ] Tidak ada secret/file liar di git status
[ ] Tidak ada perubahan luar scope
```

## Laporan Akhir

Format singkat:

```text
Selesai: <hasil>.
Berubah: <file utama>.
Verifikasi: <command> — <hasil nyata>.
Sisa: <tidak ada / blocker nyata>.
```
