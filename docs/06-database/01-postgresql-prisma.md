# 01 — PostgreSQL dan Prisma

## Tujuan

Database lokal/hosted terhubung, skema dapat dimigrasikan, dan data contoh ada.

## Pilihan Database Latihan

- Lokal: PostgreSQL via Docker/Desktop installer.
- Hosted gratis: Neon atau Supabase. Batas gratis berubah; cek situs resmi.
- Produksi: managed PostgreSQL atau container terpisah dengan backup.

## Setup Prisma

```bash
npm install prisma @prisma/client
npx prisma init
```

Isi `DATABASE_URL` di `.env`, bukan kode. `.env.example`:

```env
DATABASE_URL=postgresql://USER:PASSWORD@HOST:5432/DATABASE
```

Contoh model:

```prisma
model Product {
  id          String   @id @default(cuid())
  title       String
  slug        String   @unique
  price       Int
  isPublished Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

## Migrasi

Development:

```bash
npx prisma migrate dev --name create-products
npx prisma generate
```

Produksi:

```bash
npx prisma migrate deploy
```

Jangan jalankan `migrate reset`, `db push --force-reset`, `drop`, atau `truncate`
pada DB yang berisi data tanpa backup dan persetujuan eksplisit.

## Seed

Seed hanya data palsu dan idempotent. Jangan menaruh user/password produksi.

```bash
npx prisma db seed
npx prisma studio
```

Prisma Studio cocok untuk lihat data lokal. Untuk GUI PostgreSQL umum, pakai
pgAdmin atau DBeaver.

## Aturan Skema

- Uang simpan integer satuan terkecil (rupiah bulat: integer), bukan float.
- Waktu simpan UTC.
- Email/slug unik pakai constraint DB.
- Foreign key eksplisit.
- Data penting punya `createdAt` dan `updatedAt`.
- Status terbatas pakai enum bila stabil.

## Checklist

- [ ] `.env` masuk `.gitignore`
- [ ] Migrasi dev sukses
- [ ] Seed bisa dijalankan dua kali tanpa rusak
- [ ] Constraint unik dan foreign key ada
- [ ] Prisma Studio/pgAdmin bisa melihat data
