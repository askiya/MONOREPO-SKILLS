# 04 — Preview dan Testing Lokal

## Tujuan

Membuktikan frontend, backend, dan database bekerja bersama sebelum deploy.

## Jalankan Semua Service

Terminal 1 — database:

```bash
docker compose up -d db
```

Terminal 2 — backend:

```bash
npm run dev --workspace=apps/api
```

Terminal 3 — frontend:

```bash
npm run dev --workspace=apps/web
```

Sesuaikan command dengan struktur project. Command resmi harus dicatat di
AGENTS.md; jangan menebak.

## Verifikasi Berurutan

1. Database status running.
2. Backend `/up` merespons 200.
3. Backend endpoint data merespons JSON valid.
4. Frontend terbuka tanpa error console.
5. Frontend berhasil memanggil backend.
6. Create/update dari frontend benar-benar tersimpan di DB.
7. Unauthorized request ditolak.

## Cek CORS

Buka DevTools → Network. Request API harus:

```text
Status: 200/201 sesuai operasi
Access-Control-Allow-Origin: http://localhost:FRONTEND_PORT
```

Gejala gagal:

```text
blocked by CORS policy
```

Perbaiki `FRONTEND_ORIGIN` backend; jangan mematikan CORS.

## Testing Minimum

| Layer | Wajib |
|---|---|
| Frontend | render, loading/error state, form validation |
| Backend | validation, auth/role, endpoint success/failure |
| Database | unique/foreign key, migration, transaction kritis |
| Integrasi | frontend→API→DB pada alur utama |
| Security | secret tidak di client, CORS terbatas |

Jalankan gate per app:

```bash
npm run test --workspace=apps/api
npm run lint --workspace=apps/api
npm run build --workspace=apps/api
npm run test --workspace=apps/web
npm run lint --workspace=apps/web
npm run build --workspace=apps/web
```

Expected: semua command exit code 0.

## Bukti Kelulusan

- screenshot frontend 375px dan 1280px,
- output `/up`,
- output gate frontend dan backend,
- DevTools Network request sukses,
- commit URL GitHub setelah mendapat izin.

## Checklist

- [ ] Semua service jalan lokal
- [ ] CORS tidak memakai `*` untuk credential
- [ ] Alur utama bekerja end-to-end
- [ ] Unauthorized request ditolak
- [ ] Test/lint/build frontend hijau
- [ ] Test/lint/build backend hijau
- [ ] Tidak ada error merah di browser console
