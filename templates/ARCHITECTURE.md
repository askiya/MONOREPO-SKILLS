# ARCHITECTURE.md

## Tentang Project

<!-- satu paragraf: apa ini -->

## Stack

| Layer | Teknologi | Alasan |
|---|---|---|
| Frontend | Next.js 14+ App Router + TypeScript | ... |
| Styling | Tailwind CSS | ... |
| Komponen | shadcn/ui | ... |
| Backend | Next.js API Routes | ... |
| Database | PostgreSQL (Neon/Supabase) | ... |
| ORM | Prisma | ... |
| Auth | NextAuth.js / JWT | ... |
| Deploy staging | Vercel | ... |
| Deploy produksi | VPS + Coolify / Docker | ... |
| Payment | Xendit / Lynk.id | ... |
| Storage | Cloudflare R2 / S3 | ... |

## Struktur Folder

```
project/
├── app/
│   ├── (public)/
│   ├── (auth)/
│   ├── (member)/
│   ├── (admin)/
│   └── api/
├── components/
│   ├── ui/
│   └── sections/
├── lib/
├── prisma/
├── public/
├── styles/
├── types/
├── tests/
├── .env.example
├── AGENTS.md
├── PRD.md
├── SDLC.md
├── DESIGN.md
├── ARCHITECTURE.md
└── TASKS.md
```

## API Endpoints

| Method | Path | Auth | Input | Output |
|---|---|---|---|---|
| POST | /api/auth/register | - | { email, password, name } | { user, token } |
| POST | /api/auth/login | - | { email, password } | { user, token } |
| GET | /api/products | - | ?page, ?limit | { data, total } |
| ... | | | | |

## Role & Akses

| Aksi | Publik | Member | Admin |
|---|---:|---:|---:|
| Lihat katalog | ya | ya | ya |
| Beli | - | ya | ya |
| CRUD produk | - | - | ya |
| Kelola user | - | - | ya |

## Environment Variables

| Variabel | Tempat | Contoh |
|---|---|---|
| DATABASE_URL | .env | postgresql://USER:PASS@HOST/DB |
| NEXTAUTH_SECRET | .env | <GENERATE_RANDOM> |
| NEXTAUTH_URL | .env | http://localhost:3000 |
| ... | | |

## Keputusan & Alasan

<!-- contoh: -->
- Satu repo (monolith) — tim 1 orang.
- TIDAK pakai microservice — belum perlu.
- TIDAK pakai GraphQL — REST cukup.
