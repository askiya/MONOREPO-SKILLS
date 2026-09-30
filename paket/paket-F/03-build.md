# 03 — Build: Next.js + Express/Fastify + PostgreSQL + Docker

## Tujuan

Web (Next.js standalone), API (Express atau Fastify), dan database (PostgreSQL) berjalan sebagai tiga service terpisah, masing-masing dengan Dockerfile multi-stage, siap di-compose.

## Prasyarat

- Node.js LTS, Docker, dan Docker Compose terpasang.
- Dokumen perencanaan sudah lengkap.

> Rujukan config siap pakai: [`../../deployment-examples/docker/`](../../deployment-examples/docker/)

## 1. Struktur Workspace

```text
project/
├── apps/
│   ├── web/          ← Next.js
│   └── api/          ← Express atau Fastify
├── packages/
│   └── db/           ← Prisma schema + client
├── docker-compose.yml
├── .env              ← tidak di-commit
└── .dockerignore
```

Pisahkan tanggung jawab: web hanya rendering, API hanya logika bisnis dan data, DB hanya penyimpanan.

## 2. API Service

Pilih Express atau Fastify. Contoh minimal Express:

```bash
mkdir -p apps/api && cd apps/api
npm init -y
npm install express cors dotenv
npm install -D typescript @types/express @types/node tsx
npx tsc --init
```

File `apps/api/src/index.ts`:

```ts
import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

app.get("/health", (_req, res) => {
  res.json({ status: "ok" });
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, () => console.log(`API listening on ${PORT}`));
```

Tambah script build dan start di `package.json`:

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

## 3. Web (Next.js Standalone)

```bash
cd apps/web
npx create-next-app@latest . --typescript --app --tailwind --eslint
```

Edit `next.config.js`:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: "standalone",
};

module.exports = nextConfig;
```

Web memanggil API lewat environment variable:

```ts
const API_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:4000";
```

## 4. Database dan Prisma

```bash
mkdir -p packages/db && cd packages/db
npm init -y
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

Edit `packages/db/prisma/schema.prisma` sesuai PRD. Contoh:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Product {
  id    String @id @default(cuid())
  name  String
  price Int
}
```

API mengimpor Prisma client dari `packages/db`.

## 5. Dockerfile Web (Multi-Stage)

File `apps/web/Dockerfile`:

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 -S app && adduser -S app -u 1001
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
USER app
EXPOSE 3000
CMD ["node", "server.js"]
```

## 6. Dockerfile API (Multi-Stage)

File `apps/api/Dockerfile`:

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 -S app && adduser -S app -u 1001
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
USER app
EXPOSE 4000
CMD ["node", "dist/index.js"]
```

## 7. Docker Compose

File `docker-compose.yml` di root:

```yaml
services:
  web:
    build:
      context: ./apps/web
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:3000"
    env_file: .env
    depends_on:
      - api

  api:
    build:
      context: ./apps/api
    restart: unless-stopped
    ports:
      - "127.0.0.1:4000:4000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

Port web dan API hanya di-bind ke `127.0.0.1`. Database **tidak** diekspos ke publik.

## 8. File `.dockerignore`

```
node_modules
.next
.env
.git
```

## 9. File `.env` Lokal

```
DATABASE_URL=postgresql://dev:dev@db:5432/myapp
POSTGRES_USER=dev
POSTGRES_PASSWORD=dev
POSTGRES_DB=myapp
NEXT_PUBLIC_API_URL=http://localhost:4000
PORT=4000
```

> Jangan commit `.env`. Pastikan ada di `.gitignore`.

## 10. Migrasi

Jalankan migrasi dari host (dev) atau dari container API:

```bash
npx prisma migrate dev --name init --schema packages/db/prisma/schema.prisma
```

**Hasil yang diharapkan:** migrasi tercatat, tabel tercipta.

## 11. Build dan Verifikasi

```bash
npm run lint --workspaces
npm test --workspaces
docker compose build
```

**Hasil yang diharapkan:**

- Lint lulus.
- Test lulus.
- Image terbangun tanpa error.
- Image web memakai standalone (ukuran kecil).
- Image API memakai user non-root.

## Kegagalan Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| `Cannot find module '@prisma/client'` | Generate belum dijalankan | `npx prisma generate` |
| Build web besar (>1 GB) | `output: "standalone"` belum diset | Edit `next.config.js` |
| API tidak bisa connect ke DB | Hostname salah di container | Pakai `db` (nama service), bukan `localhost` |
| Container restart terus | Env kurang atau port salah | `docker compose logs api` |
| CORS error di browser | CORS belum diset di API | Tambah `cors()` middleware |

## Checklist

- [ ] Web: `output: "standalone"`, image multi-stage, user non-root.
- [ ] API: build TypeScript, image multi-stage, user non-root.
- [ ] Prisma schema sesuai PRD, migrasi tercatat.
- [ ] `docker compose build` berhasil.
- [ ] Port DB tidak dipublish ke luar.
- [ ] `.env` tidak di-commit.
- [ ] Lint dan test lulus.
