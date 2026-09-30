# 02 — Setup Hermes Agent

## Tujuan

Hermes Agent bekerja di root repository, membaca dokumen, mengubah kode secara
terukur, menjalankan gate, dan tidak melakukan commit/deploy tanpa izin.

## Instalasi

Ikuti dokumentasi Hermes terbaru dan materi setup repo. UI/perintah dapat berubah;
cek dokumentasi resmi sebelum instalasi.

Setelah terpasang, buka Hermes di **root repository**, bukan hanya folder frontend.
Hermes perlu melihat frontend, backend, docs, dan aturan repo sekaligus.

## Struktur yang Direkomendasikan

```text
project/
├── AGENTS.md
├── PRD.md
├── ARCHITECTURE.md
├── TASKS.md
├── apps/
│   ├── web/       # frontend Cloudflare
│   └── api/       # backend Coolify
├── packages/
│   └── shared/    # type/schema bersama bila perlu
└── .github/workflows/
```

Project sederhana boleh memakai `frontend/` dan `backend/`; jangan membuat
monorepo kompleks kalau dua folder cukup.

## Aturan di AGENTS.md

```markdown
- Jawab dalam bahasa Indonesia.
- Baca file sebelum mengedit.
- Satu task aktif sekali jalan.
- Jangan commit/push/deploy tanpa izin eksplisit.
- Jangan membaca/menulis secret nyata.
- Frontend tidak boleh mengimpor kode database/backend.
- URL API berasal dari environment variable.
- Setelah perubahan jalankan test, lint, dan build target terkait.
- Sebelum command DB/server, nyatakan destruktif atau tidak.
```

## Prompt Onboarding

```text
Baca semua dokumen perencanaan dan AGENTS.md. Jangan mengubah file.
Jelaskan arsitektur frontend Cloudflare + backend/PostgreSQL Coolify,
struktur folder, command per app, task aktif, dan risiko integrasi.
```

## Prompt Pengerjaan

```text
Kerjakan TASKS.md item FE-001 saja.
Baca file terkait sebelum edit. Jangan menyentuh backend kecuali kontrak API.
Jalankan test/lint/build frontend. Jangan commit atau deploy.
Laporkan file berubah dan output gate nyata.
```

Untuk backend, ganti ID menjadi `BE-xxx` dan gate backend.

## Hasil yang Benar

- Hermes mengedit hanya scope task.
- Tidak ada secret ditampilkan.
- Gate dijalankan dan output nyata dilaporkan.
- Tidak ada commit/push/deploy tanpa izin.

## Checklist

- [ ] Hermes dibuka dari root repository
- [ ] AGENTS.md memisahkan batas frontend/backend
- [ ] Hermes memahami Cloudflare vs Coolify
- [ ] Prompt menggunakan satu task ID
- [ ] Command lint/test/build tersedia di AGENTS.md
