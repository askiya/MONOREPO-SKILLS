# 05 — Deploy GitHub ke Cloudflare dan Coolify

## Tujuan

Satu repository GitHub menjadi sumber dua deployment: frontend ke Cloudflare,
backend ke Coolify. PostgreSQL berjalan sebagai resource terpisah di Coolify.

## Urutan Deploy

```text
1. PostgreSQL Coolify
2. Backend Coolify
3. Uji URL backend /up
4. Frontend Cloudflare dengan URL backend
5. Uji integrasi dari domain frontend
```

## 1. GitHub

- Repository private.
- Branch `develop` untuk staging, `main` untuk production bila workflow tim
  membutuhkannya.
- CI harus hijau sebelum merge/deploy.
- `.env` tidak boleh tracked.

## 2. PostgreSQL di Coolify

1. Coolify → project → environment target.
2. Add Resource → PostgreSQL.
3. Pilih versi stabil yang didukung.
4. Aktifkan persistent volume.
5. Batasi akses publik; backend mengakses lewat network internal jika memungkinkan.
6. Catat connection string melalui secret/environment Coolify, bukan chat atau Git.
7. Konfigurasikan backup eksternal sebelum production.

Verifikasi dari backend container, bukan laptop publik:

```text
migration command exit code 0
health endpoint database = healthy
```

## 3. Backend di Coolify

1. Add Resource → Public Repository/Private Repository sesuai akses GitHub.
2. Pilih repo dan branch backend.
3. Root/Base Directory: folder backend, misal `/apps/api`.
4. Build pack/Dockerfile sesuai project.
5. Isi environment variables di Coolify:
   - `DATABASE_URL`
   - `AUTH_SECRET`
   - `FRONTEND_ORIGIN`
   - variable provider lain
6. Set health check path: `/up`.
7. Deploy.

**Jangan** memasukkan variable server secret sebagai build arg frontend.

Verifikasi:

```bash
curl -i https://api.domainmu.com/up
```

Expected: HTTP 200 + JSON status ok.

## 4. Frontend di Cloudflare

1. Cloudflare Dashboard → Workers & Pages → Create → Pages/Workers.
2. Connect GitHub repository.
3. Pilih branch frontend.
4. Root directory: folder frontend, misal `apps/web`.
5. Isi build command/output sesuai framework.
6. Tambahkan:

```text
NEXT_PUBLIC_API_URL=https://api.domainmu.com
```

7. Deploy.

Expected: URL preview Cloudflare tampil dan bisa memanggil API.

## 5. Pipeline Staging vs Production

| Environment | Frontend | Backend | Database |
|---|---|---|---|
| Staging | branch `develop` / preview | Coolify staging | PostgreSQL staging |
| Production | branch `main` | Coolify production | PostgreSQL production |

DB dan secret **harus terpisah**. Jangan membuat staging memakai DB produksi.

## Rollback

- Frontend: rollback deployment dari dashboard Cloudflare.
- Backend: redeploy image/commit terakhir yang diketahui sehat di Coolify.
- Database: migration harus backward-compatible; restore hanya jika benar-benar
  perlu dan sudah punya backup.

## Checklist

- [ ] GitHub menjadi source of truth
- [ ] PostgreSQL punya volume + backup
- [ ] Backend `/up` 200 dari internet
- [ ] Frontend Cloudflare build sukses
- [ ] Frontend memakai URL API production/staging yang benar
- [ ] Staging dan production terpisah
- [ ] Rollback frontend/backend pernah disimulasikan
