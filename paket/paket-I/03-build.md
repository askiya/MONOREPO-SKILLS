# 03 — Build Web, API, Worker, dan Data

## Tujuan

Membangun image terpisah untuk Next.js, NestJS/Fastify API, dan BullMQ worker yang terhubung ke PostgreSQL serta Redis.

## Prasyarat

Node.js 20+, npm, dan Docker Engine dengan Compose plugin.

## 1. Buat Struktur

```bash
mkdir -p proyek-multiservice/apps proyek-multiservice/packages/contracts proyek-multiservice/infra/traefik
cd proyek-multiservice
npm init -y
npx create-next-app@latest apps/web --ts --eslint --app --src-dir --use-npm
npx @nestjs/cli new apps/api --package-manager npm --skip-git
```

**Hasil yang diharapkan:** `apps/web` dan `apps/api` memiliki `package.json` sendiri.

Aktifkan adapter Fastify di API:

```bash
cd apps/api
npm install @nestjs/platform-fastify @nestjs/config @nestjs/bullmq bullmq ioredis pg
npm uninstall @nestjs/platform-express
cd ../..
```

Di `main.ts`, gunakan `FastifyAdapter`, bind `0.0.0.0`, port `4000`, dan endpoint `GET /health` yang menguji koneksi penting.

## 2. Tambah Worker BullMQ

```bash
mkdir -p apps/worker/src
cd apps/worker
npm init -y
npm install bullmq ioredis
npm install -D typescript tsx @types/node
cd ../..
```

Script minimum `apps/worker/src/index.ts`:

```ts
import { Worker } from "bullmq";

const connection = { host: process.env.REDIS_HOST ?? "redis", port: 6379 };
const worker = new Worker("jobs", async job => {
  console.log(JSON.stringify({ event: "job.started", id: job.id, name: job.name }));
  return { ok: true };
}, { connection });

worker.on("completed", job => console.log(JSON.stringify({ event: "job.completed", id: job.id })));
worker.on("failed", (job, error) => console.error(JSON.stringify({ event: "job.failed", id: job?.id, error: error.message })));
```

Tambahkan script `start` dan `build` di `apps/worker/package.json`. Payload produksi wajib tervalidasi dan handler wajib idempoten.

## 3. Konfigurasi Environment

Buat `.env.example`, bukan `.env` berisi secret nyata:

```dotenv
POSTGRES_DB=app
POSTGRES_USER=app
POSTGRES_PASSWORD=ganti-dengan-secret-kuat
DATABASE_URL=postgresql://app:ganti-dengan-secret-kuat@postgres:5432/app
REDIS_HOST=redis
REDIS_PORT=6379
NEXT_PUBLIC_API_URL=http://localhost:4000
```

Pastikan `.env` masuk `.gitignore`.

## 4. Buat Multi Dockerfile

`apps/web/Dockerfile` memakai multi-stage `deps`, `builder`, `runner`; aktifkan `output: "standalone"` di Next.js. Runner non-root menjalankan `.next/standalone/server.js` pada port `3000`.

`apps/api/Dockerfile`:

```dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci
FROM deps AS build
COPY . .
RUN npm run build
FROM node:20-alpine AS runner
ENV NODE_ENV=production
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=build /app/dist ./dist
USER node
EXPOSE 4000
CMD ["node", "dist/main.js"]
```

`apps/worker/Dockerfile` mengikuti pola sama, tetapi command akhir menjalankan hasil build worker dan tidak memakai `EXPOSE`.

Tambahkan `.dockerignore` per aplikasi:

```text
node_modules
.next
dist
.env*
.git
coverage
```

## 5. Buat Compose Dasar

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes: [postgres_data:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks: [backend]
  redis:
    image: redis:7-alpine
    command: ["redis-server", "--appendonly", "yes"]
    volumes: [redis_data:/data]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    networks: [backend]
  api:
    build: ./apps/api
    env_file: .env
    depends_on:
      postgres: { condition: service_healthy }
      redis: { condition: service_healthy }
    networks: [backend, proxy]
  worker:
    build: ./apps/worker
    env_file: .env
    depends_on:
      postgres: { condition: service_healthy }
      redis: { condition: service_healthy }
    networks: [backend]
  web:
    build: ./apps/web
    environment:
      NEXT_PUBLIC_API_URL: http://api:4000
    depends_on:
      api: { condition: service_started }
    networks: [proxy]
networks:
  proxy: {}
  backend: { internal: true }
volumes:
  postgres_data: {}
  redis_data: {}
```

Untuk preview browser, tambahkan override port di panduan berikut. Jangan publikasikan port data.

## 6. Build dan Validasi

```bash
cp .env.example .env
docker compose config --quiet
docker compose build
```

**Hasil yang diharapkan:** validasi senyap dengan exit code `0`; tiga image aplikasi selesai dibangun tanpa secret tercetak.

## Masalah Umum

| Gejala | Perbaikan |
|---|---|
| Next.js gagal di image runner | aktifkan `output: "standalone"` dan salin static/public |
| API hanya bisa diakses dalam container | bind Fastify ke `0.0.0.0` |
| worker keluar segera | pastikan proses `Worker` tetap aktif dan error startup tidak ditelan |
| `npm ci` gagal | commit lockfile yang cocok dengan tiap `package.json` |
| DB bisa diakses dari internet | hapus `ports` dari PostgreSQL dan Redis |

## Checklist

- [ ] Web, API, dan worker punya Dockerfile terpisah.
- [ ] Semua runtime berjalan sebagai user non-root.
- [ ] API memakai NestJS dengan Fastify.
- [ ] Worker memproses BullMQ dan handler dirancang idempoten.
- [ ] PostgreSQL dan Redis memiliki volume serta healthcheck.
- [ ] Jaringan `backend` internal dan tidak ada port data publik.
- [ ] `.env` diabaikan; `.env.example` tidak berisi secret nyata.
- [ ] `docker compose config --quiet` dan `docker compose build` lulus.
