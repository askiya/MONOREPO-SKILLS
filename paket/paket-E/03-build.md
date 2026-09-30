# 03 — Build: Next.js Standalone + Prisma + PostgreSQL

## Tujuan

Aplikasi Next.js berjalan secara lokal dengan database PostgreSQL, Prisma ORM, dan output standalone siap deploy ke Coolify.

## Prasyarat

- Node.js LTS terpasang (periksa `node -v`).
- PostgreSQL lokal berjalan atau pakai Docker untuk development.
- Dokumen perencanaan sudah jadi.

## 1. Inisialisasi Next.js

```bash
npx create-next-app@latest nama-project --typescript --app --tailwind --eslint
cd nama-project
```

**Hasil yang diharapkan:** folder proyek tercipta, `npm run dev` menampilkan halaman default.

## 2. Aktifkan Output Standalone

Edit `next.config.js` (atau `next.config.ts`):

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: "standalone",
};

module.exports = nextConfig;
```

> **Penting:** tanpa `output: "standalone"`, Coolify membangun image berukuran sangat besar karena menyertakan seluruh `node_modules`. Standalone hanya menyalin file yang diperlukan ke `.next/standalone/`.

Verifikasi:

```bash
npm run build
ls .next/standalone/server.js
```

**Hasil yang diharapkan:** file `server.js` ada di `.next/standalone/`.

## 3. Pasang Prisma

```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

**Hasil yang diharapkan:** folder `prisma/` dan file `.env` tercipta.

## 4. Buat Skema Database

Edit `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Product {
  id          String   @id @default(cuid())
  name        String
  description String?
  price       Int
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

Sesuaikan model dengan PRD. Ini contoh minimal.

## 5. Siapkan PostgreSQL Lokal

Gunakan Docker agar tidak mengotori mesin:

```bash
docker run -d \
  --name dev-postgres \
  -e POSTGRES_USER=dev \
  -e POSTGRES_PASSWORD=dev \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  postgres:16-alpine
```

Isi `.env`:

```
DATABASE_URL="postgresql://dev:dev@localhost:5432/myapp"
```

> Jangan commit `.env`. Pastikan file ini sudah ada di `.gitignore`.

## 6. Jalankan Migrasi

```bash
npx prisma migrate dev --name init
```

**Hasil yang diharapkan:** tabel terbentuk, file migrasi tercatat di `prisma/migrations/`.

```bash
npx prisma studio
```

Buka Prisma Studio di browser. Tabel harus terlihat dan kosong.

## 7. Buat Prisma Client Singleton

Buat file `lib/prisma.ts`:

```ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

Singleton mencegah kebocoran koneksi saat hot reload development.

## 8. Buat Route Handler Contoh

File `app/api/products/route.ts`:

```ts
import { prisma } from "@/lib/prisma";
import { NextResponse } from "next/server";

export async function GET() {
  const products = await prisma.product.findMany({
    orderBy: { createdAt: "desc" },
  });
  return NextResponse.json(products);
}

export async function POST(request: Request) {
  const body = await request.json();
  const product = await prisma.product.create({ data: body });
  return NextResponse.json(product, { status: 201 });
}
```

## 9. Tambah Health Endpoint

File `app/api/health/route.ts`:

```ts
import { prisma } from "@/lib/prisma";
import { NextResponse } from "next/server";

export async function GET() {
  try {
    await prisma.$queryRaw`SELECT 1`;
    return NextResponse.json({ status: "ok" });
  } catch {
    return NextResponse.json({ status: "error" }, { status: 503 });
  }
}
```

## 10. Lint dan Build

```bash
npm run lint
npm run build
```

**Hasil yang diharapkan:**

- Tidak ada error lint.
- Build sukses, folder `.next/standalone/` berisi `server.js`.
- Prisma client tergenerate.

## Kegagalan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| `Can't reach database server` | PostgreSQL belum jalan | `docker ps` untuk memastikan container hidup |
| Build gagal, error TypeScript | Import salah atau tipe kurang | Baca pesan error, perbaiki tipe |
| `.next/standalone` kosong | `output: "standalone"` belum diset | Edit `next.config.js` lalu build ulang |
| Migrasi gagal | `DATABASE_URL` salah format | Periksa format `postgresql://user:pass@host:port/db` |
| Koneksi bocor saat development | Prisma Client dibuat ulang terus | Gunakan singleton `lib/prisma.ts` |

## Checklist

- [ ] `next.config.js` menyertakan `output: "standalone"`.
- [ ] `npm run build` menghasilkan `.next/standalone/server.js`.
- [ ] Prisma terhubung ke PostgreSQL.
- [ ] Migrasi tercatat di `prisma/migrations/`.
- [ ] Health endpoint merespons `{ status: "ok" }`.
- [ ] `.env` tercantum di `.gitignore`.
- [ ] Lint lulus tanpa error.
