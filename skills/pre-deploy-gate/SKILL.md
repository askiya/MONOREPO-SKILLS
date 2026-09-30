---
name: pre-deploy-gate
description: "Use before deploying a web app. Runs evidence-based test, secret, environment, migration, rollback, and backup checks."
version: 1.0.0
author: Santriverse
license: MIT
---

# Pre-Deploy Gate

## Tujuan

Menolak deploy bila bukti belum cukup. Jangan mengganti test nyata dengan
perkiraan.

## Prosedur

1. Baca `AGENTS.md`, manifest, script package, dan konfigurasi deploy.
2. Jalankan `git status` dan cek diff target deploy.
3. Pastikan `.env`, key, token, dump DB, dan file sesi tidak tracked/staged.
4. Jalankan command repo yang nyata: test, lint, build.
5. Daftar nama environment variable wajib; jangan tampilkan nilainya.
6. Bila ada migrasi, tentukan backward-compatible atau destructive.
7. Pastikan backup terbaru dan prosedur rollback tersedia.
8. Deploy hanya bila user memberi izin eksplisit.
9. Setelah deploy, verifikasi URL/health endpoint dan alur kritis.

## Gate

```text
[ ] Working tree dipahami
[ ] Secret scan manual lolos
[ ] Test lulus
[ ] Lint lulus
[ ] Build lulus
[ ] Environment lengkap
[ ] Migrasi dinilai aman
[ ] Backup + rollback siap
[ ] Izin deploy eksplisit
```

## Stop Conditions

Berhenti bila:
- test/lint/build gagal,
- secret terdeteksi,
- migrasi destructive tanpa backup,
- target produksi tidak jelas,
- user belum mengizinkan deploy.

Laporkan blocker dengan output mentah relevan. Jangan push perubahan spekulatif.
