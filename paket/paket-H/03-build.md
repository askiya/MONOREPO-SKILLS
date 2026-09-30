# H3 — Build Turborepo: Web, API, Shared

> 🟠 LANJUTAN · Target waktu: 2 hari

## Tujuan

Membuat npm-workspaces monorepo berisi Next.js, Express, paket shared, dan pipeline Turborepo yang dapat dibangun dari root.

## Langkah 1 — Scaffold Turborepo

Cara cepat:

```bash
npx create-turbo@latest santriverse --package-manager npm
cd santriverse
npm install
```

**Output yang diharapkan:** root mempunyai `apps/`, `packages/`, `turbo.json`, `package-lock.json`.

Hapus contoh app/package yang tidak dipakai secara hati-hati, lalu pastikan root `package.json`:

```json
{
  "name": "santriverse",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "lint": "turbo lint",
    "test": "turbo test"
  },
  "devDependencies": { "turbo": "latest" }
}
```

Gunakan versi yang dikunci `package-lock.json`; jangan mempertahankan `latest` sebagai ekspektasi reproducibility tanpa lockfile.

## Langkah 2 — Buat Next.js Web

```bash
npx create-next-app@latest apps/web --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-npm
```

Pastikan nama `apps/web/package.json` unik:

```json
"name": "@santriverse/web"
```

**Output yang diharapkan:** `npm --workspace @santriverse/web run dev` membuka <http://localhost:3000>.

## Langkah 3 — Buat Express API

```bash
mkdir -p apps/api/src
npm init -y --workspace apps/api
npm install --workspace apps/api express cors zod
npm install --workspace apps/api -D typescript tsx @types/node @types/express @types/cors
```

`apps/api/package.json`:

```json
{
  "name": "@santriverse/api",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc -p tsconfig.json",
    "start": "node dist/index.js",
    "lint": "tsc --noEmit",
    "test": "node --test"
  }
}
```

`apps/api/tsconfig.json`:

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src",
    "noEmit": false
  },
  "include": ["src/**/*.ts"]
}
```

## Langkah 4 — Buat Shared Package

```bash
mkdir -p packages/shared/src
npm init -y --workspace packages/shared
npm install --workspace packages/shared zod
npm install --workspace packages/shared -D typescript
```

`packages/shared/package.json`:

```json
{
  "name": "@santriverse/shared",
  "version": "0.0.0",
  "private": true,
  "type": "module",
  "exports": { ".": "./dist/index.js" },
  "types": "./dist/index.d.ts",
  "scripts": {
    "build": "tsc -p tsconfig.json",
    "lint": "tsc --noEmit",
    "test": "node --test"
  },
  "dependencies": { "zod": "^3.0.0" }
}
```

Sesuaikan versi Zod dengan hasil install nyata. `packages/shared/tsconfig.json`:

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "noEmit": false
  },
  "include": ["src/**/*.ts"]
}
```

`tsconfig.base.json` di root:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true
  }
}
```

`packages/shared/src/index.ts`:

```ts
import { z } from "zod";

export const orderInputSchema = z.object({
  productId: z.string().min(1),
  namaPembeli: z.string().trim().min(2).max(100),
  email: z.string().email(),
});

export type OrderInput = z.infer<typeof orderInputSchema>;

export type ApiResult<T> =
  | { data: T }
  | { error: { code: string; message: string; details?: unknown } };
```

## Langkah 5 — Hubungkan Workspace

```bash
npm install --workspace apps/api @santriverse/shared@workspace:*
npm install --workspace apps/web @santriverse/shared@workspace:*
```

Jika npm versi lokal tidak menerima protokol `workspace:*`, set dependency ke `"*"`; npm workspaces akan melakukan symlink lokal.

## Langkah 6 — Express API

`apps/api/src/index.ts`:

```ts
import cors from "cors";
import express from "express";
import { orderInputSchema } from "@santriverse/shared";

const app = express();
const port = Number(process.env.PORT ?? 4000);
const origin = process.env.CORS_ORIGIN ?? "http://localhost:3000";

app.use(cors({ origin }));
app.use(express.json({ limit: "100kb" }));

app.get("/health", (_req, res) => {
  res.json({ status: "ok" });
});

app.post("/v1/orders", (req, res) => {
  const parsed = orderInputSchema.safeParse(req.body);
  if (!parsed.success) {
    res.status(400).json({
      error: { code: "INVALID_INPUT", message: "Data tidak valid", details: parsed.error.flatten() },
    });
    return;
  }
  res.status(201).json({ data: { id: crypto.randomUUID(), ...parsed.data } });
});

app.listen(port, "0.0.0.0", () => console.log(`API aktif di port ${port}`));
```

Penyimpanan masih in-memory/demo response. Tambah DB hanya bila PRD meminta; jangan menambah Prisma sebagai dekorasi.

## Langkah 7 — Turbo Pipeline

`turbo.json`:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "dev": { "cache": false, "persistent": true },
    "lint": { "dependsOn": ["^lint"] },
    "test": { "dependsOn": ["^build"], "outputs": ["coverage/**"] }
  }
}
```

`^build` memastikan shared dibangun sebelum consumer.

## Langkah 8 — Web Memanggil API

`apps/web/.env.local`:

```dotenv
NEXT_PUBLIC_API_URL=http://localhost:4000
```

Contoh fungsi:

```ts
import { orderInputSchema, type ApiResult } from "@santriverse/shared";

export async function createOrder(input: unknown): Promise<ApiResult<{ id: string }>> {
  const body = orderInputSchema.parse(input);
  const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/v1/orders`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  });
  return response.json();
}
```

## Langkah 9 — Build Root

```bash
npm install
npx turbo run lint build
```

**Output yang diharapkan:** shared build lebih dulu; web dan API build sukses; ringkasan task tanpa failure.

## Error Umum

| Error | Penyebab | Solusi |
|---|---|---|
| `Cannot find package @santriverse/shared` | Dependency/workspace install belum benar | Tambah dependency lalu `npm install` root |
| Type ditemukan, runtime gagal | Shared belum build/exports salah | Periksa `exports`, `dist`, `dependsOn` |
| API `dist` kosong | `noEmit` diwarisi | Override `noEmit: false` |
| Shared perubahan tidak rebuild | Pipeline tanpa `^build` | Tambah dependency pipeline |
| Web mengakses env `API_URL` undefined | Browser hanya expose `NEXT_PUBLIC_*` | Gunakan nama publik untuk base URL non-secret |

## Checklist

- [ ] Workspaces root mencakup `apps/*`, `packages/*`
- [ ] Web aktif pada 3000, API pada 4000
- [ ] Shared hanya berisi kode runtime-agnostic
- [ ] Web dan API mengimpor shared melalui package
- [ ] Tidak ada import source lintas app
- [ ] Turbo build memakai `dependsOn: ["^build"]`
- [ ] `npx turbo run lint build` sukses
- [ ] `.env.local` tidak di-commit

➡️ Lanjut ke **[04-preview.md](04-preview.md)**.
