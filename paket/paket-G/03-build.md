# G3 — Build Next.js + Prisma + PostgreSQL

> 🔵 MENENGAH · Target waktu: 1 hari · Deploy target: Railway

## Tujuan

Aplikasi Next.js fullstack berjalan lokal, menyimpan data lewat Prisma ke PostgreSQL, punya health check, dan siap dibangun Railway.

## Prasyarat

- Node.js LTS dan npm tersedia.
- PostgreSQL lokal berjalan, atau gunakan koneksi development terpisah.
- Dokumen Paket G selesai.

```bash
node --version
npm --version
psql --version
```

**Output yang diharapkan:** tiga perintah menampilkan versi. Jangan lanjut jika `command not found`.

## Langkah 1 — Inisialisasi Next.js

Jalankan di folder kosong project:

```bash
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-npm
npm install prisma @prisma/client zod
npx prisma init
```

**Output yang diharapkan:** `src/app/`, `prisma/schema.prisma`, `.env`, dan `package.json` terbentuk.

Pastikan `.env` sudah tercantum di `.gitignore`.

## Langkah 2 — Siapkan Database Lokal

Buat database development:

```bash
createdb produk_digital_dev
```

Isi `.env` lokal, jangan commit:

```dotenv
DATABASE_URL="postgresql://postgres:<ISI_PASSWORD_LOKAL>@localhost:5432/produk_digital_dev?schema=public"
CRON_ENABLED="false"
SESSION_SECRET="<ISI_RANDOM_MINIMAL_32_KARAKTER>"
```

Uji koneksi:

```bash
npx prisma db pull
```

**Output yang diharapkan:** Prisma berhasil terhubung. Untuk database kosong, pesan tanpa tabel masih wajar.

## Langkah 3 — Tulis Schema Prisma

`prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum OrderStatus {
  BARU
  DIPROSES
  SELESAI
  BATAL
}

model Product {
  id        String   @id @default(cuid())
  slug      String   @unique
  nama      String
  deskripsi String
  harga     Int
  aktif     Boolean  @default(true)
  orders    Order[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Order {
  id           String      @id @default(cuid())
  productId    String
  product      Product     @relation(fields: [productId], references: [id])
  namaPembeli  String
  email        String
  status       OrderStatus @default(BARU)
  createdAt    DateTime    @default(now())
}
```

Terapkan migrasi:

```bash
npx prisma format
npx prisma migrate dev --name init
npx prisma generate
```

**Output yang diharapkan:** `prisma/migrations/..._init/migration.sql` dibuat dan database sinkron.

## Langkah 4 — Prisma Client Singleton

Buat `src/lib/prisma.ts`:

```ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma =
  globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

Singleton mencegah koneksi berlipat saat hot reload dan proses Railway hidup lama.

## Langkah 5 — API Produk dan Pesanan

`src/app/api/products/route.ts`:

```ts
import { prisma } from "@/lib/prisma";

export async function GET() {
  const products = await prisma.product.findMany({
    where: { aktif: true },
    orderBy: { createdAt: "desc" },
  });
  return Response.json(products);
}
```

`src/app/api/orders/route.ts`:

```ts
import { prisma } from "@/lib/prisma";
import { z } from "zod";

const orderSchema = z.object({
  productId: z.string().min(1),
  namaPembeli: z.string().trim().min(2).max(100),
  email: z.string().email(),
});

export async function POST(request: Request) {
  const parsed = orderSchema.safeParse(await request.json());
  if (!parsed.success) {
    return Response.json({ error: parsed.error.flatten() }, { status: 400 });
  }
  const order = await prisma.order.create({ data: parsed.data });
  return Response.json(order, { status: 201 });
}
```

## Langkah 6 — Health Check

`src/app/api/health/route.ts`:

```ts
import { prisma } from "@/lib/prisma";

export async function GET() {
  try {
    await prisma.$queryRaw`SELECT 1`;
    return Response.json({ status: "ok", database: "ok" });
  } catch {
    return Response.json(
      { status: "error", database: "unreachable" },
      { status: 503 },
    );
  }
}
```

## Langkah 7 — Script Railway

Ubah `package.json` bagian scripts:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "prisma generate && next build",
    "start": "next start",
    "lint": "next lint",
    "db:deploy": "prisma migrate deploy"
  }
}
```

Railway otomatis menyediakan `PORT`; `next start` membacanya. Jangan hardcode port.

Tambahkan `postinstall` hanya bila build provider tidak menjalankan `prisma generate` dari script build:

```json
"postinstall": "prisma generate"
```

## Langkah 8 — Seed dan Uji

Buat produk via Prisma Studio:

```bash
npx prisma studio
```

Tambah minimal 3 baris `Product`, lalu:

```bash
npm run dev
```

Terminal kedua:

```bash
curl http://localhost:3000/api/health
curl http://localhost:3000/api/products
```

**Output yang diharapkan:** health `{"status":"ok","database":"ok"}` dan array produk.

## Langkah 9 — Gate Build

```bash
npm run lint
npm run build
npm start
```

**Output yang diharapkan:** lint tanpa error, build selesai, server produksi aktif di port 3000.

## Error Umum

| Error | Penyebab | Solusi |
|---|---|---|
| `P1001 Can't reach database server` | URL/port/Postgres salah | Periksa service dan `DATABASE_URL` |
| `P2021 table does not exist` | Migrasi belum jalan | `npx prisma migrate dev` |
| `Too many connections` | Prisma dibuat tiap request | Gunakan singleton |
| `PrismaClientInitializationError` saat build | Kode query dipanggil saat static build | Jadikan halaman dinamis atau pindahkan query ke runtime |
| Build Railway gagal generate client | Generate tidak masuk build | Tambah `prisma generate` pada script build |

## Checklist

- [ ] Next.js App Router + TypeScript terbentuk
- [ ] Prisma schema sesuai ARCHITECTURE.md
- [ ] Migrasi `init` tersimpan
- [ ] Prisma Client singleton
- [ ] Input API divalidasi Zod
- [ ] `/api/health` menguji database dan memberi 503 saat gagal
- [ ] `npm run lint` sukses
- [ ] `npm run build` sukses
- [ ] Tidak ada secret nyata di source code

➡️ Lanjut ke **[04-preview.md](04-preview.md)**.
