# 01 — Pedoman Arsitektur Santriverse Hybrid

## Tujuan

Mendefinisikan batas frontend, backend, database, dan deployment sebelum Hermes
menulis kode. Paket hybrid gagal kalau kontrak antar-layanan hanya ada di kepala.

## Dokumen Wajib

Copy file dari [`../../templates/`](../../templates/) ke root project:

```text
PRD.md
SDLC.md
DESIGN.md
ARCHITECTURE.md
TASKS.md
AGENTS.md
.env.example
```

Contoh terisi: [`../../examples/toko-produk-digital/`](../../examples/toko-produk-digital/).

## Isi ARCHITECTURE.md secara Spesifik

### 1. Topologi

```text
Frontend public URL : https://app.domainmu.com
Backend public URL  : https://api.domainmu.com
Database            : PostgreSQL private network di Coolify
Source repository   : GitHub private
```

### 2. Batas Tanggung Jawab

| Layer | Boleh | Tidak boleh |
|---|---|---|
| Frontend | render UI, form, panggil API | akses DB langsung, simpan secret server |
| Backend | validasi, auth, business logic, DB | percaya validasi client |
| Database | constraint, transaction, persistence | dibuka publik tanpa alasan |
| Cloudflare | host/CDN frontend, DNS, proxy | menjadi sumber data aplikasi |
| Coolify | run API/worker/DB, env, logs | menyimpan source sebagai satu-satunya copy |
| GitHub | source of truth + CI | menyimpan `.env`/secret |

### 3. Kontrak API

Tulis setiap endpoint sebelum implementasi:

```text
POST /api/auth/login
Request : { email: string, password: string }
Success : 200 { user: { id, name, role }, token? }
Failure : 400 validation, 401 credentials, 429 rate limit
Auth    : public
```

### 4. Environment Matrix

| Nama | Frontend | Backend | Secret? |
|---|---|---|---|
| `NEXT_PUBLIC_API_URL` | ya | tidak | tidak |
| `DATABASE_URL` | tidak | ya | **ya** |
| `AUTH_SECRET` | tidak | ya | **ya** |
| `FRONTEND_ORIGIN` | tidak | ya | tidak |

Nilai nyata hanya di dashboard Cloudflare/Coolify. `.env.example` berisi nama
kosong, bukan credential.

### 5. Deployment Map

Tulis folder mana menuju target mana:

```text
apps/web atau frontend/  → Cloudflare
apps/api atau backend/   → Coolify
migrations/              → dijalankan backend release job
```

## TASKS.md

Pisahkan task per layanan:

```text
FE-001 Setup halaman login
BE-001 POST /api/auth/login
DB-001 Tambah tabel users + unique email
INFRA-001 Buat PostgreSQL Coolify
INFRA-002 Deploy backend Coolify
INFRA-003 Deploy frontend Cloudflare
QA-001 Uji CORS dari domain staging
```

## Prompt Validasi untuk Hermes

```text
Baca PRD.md, SDLC.md, DESIGN.md, ARCHITECTURE.md, TASKS.md, dan AGENTS.md.
Jangan edit file. Rangkum:
1. batas frontend/backend/database,
2. folder yang deploy ke Cloudflare dan Coolify,
3. kontrak API,
4. nama environment variable tanpa nilainya,
5. task aktif dan konflik dokumen.
Kalau ada konflik, berhenti dan laporkan.
```

## Hasil yang Benar

Hermes menyebut dua target deploy, tidak memasukkan `DATABASE_URL` ke frontend,
dan tidak menebak endpoint di luar kontrak.

## Checklist

- [ ] 6 dokumen utama terisi, tidak ada `TODO`
- [ ] Diagram frontend→API→DB ada
- [ ] Kontrak API tertulis
- [ ] Environment matrix memisahkan public vs secret
- [ ] Task memakai prefix FE/BE/DB/INFRA/QA
- [ ] Hermes berhasil merangkum tanpa konflik
