# 03 — Build Frontend, Backend, dan Database

## Tujuan

Membangun tiga bagian terpisah tetapi terintegrasi melalui kontrak API.

## Urutan Aman

```text
1. Shared schema/contract
2. Backend + database
3. Test endpoint
4. Frontend memakai endpoint
5. Integration test
```

Jangan membangun UI penuh sebelum endpoint kritis punya kontrak jelas.

## 1. Backend

Prompt Hermes:

```text
Kerjakan BE-001 saja. Buat endpoint health GET /up yang mengembalikan
{ "status": "ok" }. Gunakan struktur backend yang sudah ada.
Tambah test dan jalankan test/lint/build backend. Jangan commit.
```

Expected:

```text
GET http://localhost:BACKEND_PORT/up
200 {"status":"ok"}
```

## 2. PostgreSQL Lokal

Untuk development, gunakan PostgreSQL lokal/Docker atau database development
terpisah. Jangan pernah memakai database produksi.

Contoh Docker development:

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: app_dev
      POSTGRES_USER: app
      POSTGRES_PASSWORD: local_only_change_me
    ports:
      - "127.0.0.1:5432:5432"
    volumes:
      - postgres_dev:/var/lib/postgresql/data
volumes:
  postgres_dev:
```

Nilai di atas hanya lokal. Jangan pakai di production.

Jalankan:

```bash
docker compose up -d db
docker compose ps
```

Expected: service `db` status `Up`/`running`.

## 3. Frontend

Frontend hanya mengetahui base URL API:

```bash
NEXT_PUBLIC_API_URL=http://localhost:BACKEND_PORT
```

Prompt Hermes:

```text
Kerjakan FE-001 saja. Buat API client yang membaca NEXT_PUBLIC_API_URL.
Jangan hardcode localhost di source. Buat halaman status yang memanggil GET /up,
dengan loading, success, dan error state. Jalankan test/lint/build frontend.
```

## 4. CORS Lokal

Backend mengizinkan origin frontend development saja:

```text
http://localhost:FRONTEND_PORT
```

Jangan memakai origin `*` untuk endpoint dengan credential/cookie.

## 5. Gate

Jalankan dari root:

```bash
npm run test
npm run lint
npm run build
```

Kalau monorepo memakai command berbeda, catat command per app di AGENTS.md.

## Hasil yang Benar

- API `/up` merespons 200.
- Frontend menampilkan status API.
- Network tab tidak menunjukkan CORS error.
- Tidak ada `DATABASE_URL` di bundle frontend.
- Build frontend dan backend exit code 0.

## Checklist

- [ ] Backend health endpoint punya test
- [ ] Database development terpisah dari produksi
- [ ] Frontend membaca URL API dari env
- [ ] CORS dibatasi ke origin development
- [ ] Loading/success/error state terlihat
- [ ] Test, lint, build hijau
