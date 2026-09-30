# ARCHITECTURE.md — SantriLearn

## Tentang Project

Toko produk digital: e-book, video course, template untuk santri belajar IT.

## Stack

| Layer | Teknologi | Alasan |
|---|---|---|
| Frontend | Next.js 14 App Router + TypeScript | SSR + SEO + satu repo |
| Styling | Tailwind CSS | cepat, utility-first |
| Komponen | shadcn/ui (dicopy, bukan npm) | bisa diubah bebas |
| Backend | Next.js API Routes | satu repo, serverless-ready |
| Database | PostgreSQL (Neon free) | relasional, tier gratis |
| ORM | Prisma | type-safe, migrasi deklaratif |
| Auth | NextAuth.js credentials provider | email+password sederhana |
| Deploy staging | Vercel | gratis, auto-deploy dari GitHub |
| Deploy produksi | VPS + Coolify | kontrol penuh (kalau naik) |
| Payment | Xendit / Lynk.id | Indonesia, webhook |
| Storage | Cloudflare R2 (nanti) | file produk |

## Struktur Folder

```
santrilearn/
├── app/
│   ├── (public)/
│   │   ├── page.tsx              # landing
│   │   ├── produk/page.tsx       # katalog
│   │   └── produk/[slug]/page.tsx
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (member)/
│   │   ├── layout.tsx            # proteksi session
│   │   ├── dashboard/page.tsx
│   │   └── library/page.tsx      # produk yang dimiliki
│   ├── (admin)/
│   │   ├── layout.tsx            # cek role admin
│   │   ├── admin/products/page.tsx
│   │   └── admin/orders/page.tsx
│   ├── api/
│   │   ├── auth/[...nextauth]/route.ts
│   │   ├── products/route.ts
│   │   ├── products/[id]/route.ts
│   │   ├── checkout/route.ts
│   │   └── webhook/payment/route.ts
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── ui/                       # Button, Input, Card, Badge, Modal
│   └── sections/                 # Hero, Pricing, ProductGrid
├── lib/
│   ├── auth.ts
│   ├── db.ts                     # Prisma client singleton
│   ├── payment.ts
│   └── utils.ts
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
├── public/
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
| POST | /api/auth/register | - | { email, password, name } | { user } |
| POST | /api/auth/login | - | via NextAuth | session |
| GET | /api/products | - | ?page, ?limit, ?type | { data[], total } |
| GET | /api/products/:id | - | - | { product } |
| POST | /api/checkout | member | { productId } | { invoiceUrl } |
| POST | /api/webhook/payment | webhook | raw body + signature | 200 |
| GET | /api/admin/products | admin | ?page | { data[], total } |
| POST | /api/admin/products | admin | { title, slug, price, type, ... } | { product } |
| PATCH | /api/admin/products/:id | admin | partial fields | { product } |
| DELETE | /api/admin/products/:id | admin | - | 204 |
| GET | /api/admin/orders | admin | ?page, ?status | { data[], total } |

## Role & Akses

| Aksi | Publik | Member | Admin |
|---|---:|---:|---:|
| Lihat landing & katalog | ya | ya | ya |
| Lihat detail produk | ya | ya | ya |
| Beli produk | - | ya | ya |
| Akses library | - | ya | ya |
| CRUD produk | - | - | ya |
| Lihat semua pesanan | - | - | ya |

## Environment Variables

| Variabel | Tempat | Contoh |
|---|---|---|
| DATABASE_URL | .env | postgresql://user:pass@host/db |
| NEXTAUTH_SECRET | .env | <openssl rand -base64 32> |
| NEXTAUTH_URL | .env | http://localhost:3000 |
| XENDIT_SECRET_KEY | .env | xnd_development_... |
| XENDIT_WEBHOOK_TOKEN | .env | <dari dashboard> |

## Keputusan & Alasan

- Satu repo Next.js (monolith) karena tim 1 orang.
- App Router karena standar Next.js terbaru.
- Credentials provider dulu; Google OAuth nanti.
- Prisma karena migrasi deklaratif dan Neon-friendly.
- TIDAK pakai microservice.
- TIDAK pakai GraphQL.
- TIDAK pakai keranjang/multi-item checkout — beli satu per satu, lebih simpel.
- TIDAK pakai Redis — belum ada masalah performa yang memerlukannya.
