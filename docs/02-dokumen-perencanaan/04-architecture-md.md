# 04 — Menulis ARCHITECTURE.md

## Tujuan

Menetapkan keputusan teknis supaya agent tidak menebak stack, struktur folder,
atau kontrak API.

## Isi Minimum

### 1. Tech Stack

```markdown
## Stack

| Layer | Teknologi | Alasan |
|---|---|---|
| Frontend | Next.js 14+ App Router | SSR + SEO + API Routes |
| Styling | Tailwind CSS | utility-first, cepat iterasi |
| Backend | Next.js API Routes / Express | satu repo, simpel |
| Database | PostgreSQL (Neon/Supabase) | relasional, gratis tier dev |
| ORM | Prisma | type-safe, migrasi mudah |
| Auth | NextAuth.js / custom JWT | session + OAuth siap |
| Deploy | Vercel (staging) + VPS Coolify (prod) | gratis staging, kontrol prod |
| Payment | Xendit / Lynk.id | Indonesia, webhook |
```

### 2. Struktur Folder

```markdown
## Folder

project-root/
├── app/                    # Next.js App Router pages
│   ├── (public)/           # halaman tanpa auth
│   ├── (member)/           # halaman butuh login
│   ├── (admin)/            # admin panel
│   └── api/                # API routes
├── components/             # komponen reusable
│   ├── ui/                 # primitif (Button, Input, Card)
│   └── sections/           # bagian halaman (Hero, Pricing)
├── lib/                    # utilitas, helper, config
├── prisma/                 # schema + migrasi
├── public/                 # asset statis
├── styles/                 # global CSS
├── types/                  # TypeScript types
├── tests/                  # test
├── AGENTS.md
├── PRD.md
├── SDLC.md
├── DESIGN.md
├── ARCHITECTURE.md
├── TASKS.md
└── .env.example
```

### 3. Kontrak API

```markdown
## API Endpoints

| Method | Path | Auth | Input | Output |
|---|---|---|---|---|
| POST | /api/auth/register | - | { email, password, name } | { user, token } |
| POST | /api/auth/login | - | { email, password } | { user, token } |
| GET | /api/products | - | ?page, ?limit | { products[], total } |
| GET | /api/products/:id | - | - | { product } |
| POST | /api/checkout | member | { productId } | { invoiceUrl } |
| GET | /api/admin/products | admin | ?page | { products[], total } |
| POST | /api/admin/products | admin | { title, price, ... } | { product } |
```

### 4. Keputusan & Alasan

```markdown
## Keputusan

- Satu repo (monolith) karena tim 1 orang.
- App Router, bukan Pages Router, karena standar baru Next.js.
- Prisma karena migrasi deklaratif dan type-safe.
- Tidak pakai microservice; belum perlu.
- Tidak pakai GraphQL; REST cukup untuk skala ini.
```

### 5. Environment Variables

```markdown
## Env

| Variabel | Tempat | Contoh |
|---|---|---|
| DATABASE_URL | .env | postgresql://user:pass@host/db |
| NEXTAUTH_SECRET | .env | random-32-char |
| NEXTAUTH_URL | .env | http://localhost:3000 |
| XENDIT_SECRET_KEY | .env (server only) | xnd_... |
```

Jangan tulis nilai asli. File `.env.example` berisi placeholder.

## Tips

- Tulis ARCHITECTURE setelah PRD dan DESIGN. Urutan: masalah → visual → teknis.
- Jangan tulis yang tidak dipakai — YAGNI.
- Kalau belum tahu, tulis "BELUM DIPUTUSKAN" — lebih jujur daripada tebakan.
- Update ARCHITECTURE setiap kali ada keputusan baru yang berdampak luas.

## Checklist

- [ ] Stack terdefinisi per layer
- [ ] Struktur folder terdokumentasi
- [ ] Kontrak API minimal ditulis
- [ ] Keputusan teknis ada alasannya
- [ ] Env variables terdaftar, nilai asli tidak ditulis
