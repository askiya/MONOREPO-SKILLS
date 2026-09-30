# H7 — Maintenance Monorepo Turborepo

> 🟠 LANJUTAN · Rutinitas mingguan + bulanan

## Tujuan

Menjaga dependency sehat, shared tetap stabil, Turborepo cache efektif, dan biaya dua platform terkendali.

## 1. Dependency Management Monorepo

### Update dari root

```bash
npm outdated
npm audit
```

Output menunjukkan semua workspace sekaligus.

Untuk update satu workspace:

```bash
npm update --workspace @santriverse/api
```

Untuk update root dev dependency:

```bash
npm update turbo
```

### Aturan update

| Jenis | Aman langsung? | Langkah |
|---|---|---|
| Patch (`1.2.3→1.2.4`) | Biasanya aman | Update, build, test, deploy |
| Minor (`1.2→1.3`) | Baca changelog | Update, build, test terkait |
| Major (`1→2`) | Tidak; buat task | Baca migration guide, branch terpisah, test menyeluruh |

Jangan `npm audit fix --force` tanpa review—dapat mematahkan API.

### Shared dependency version

Paket yang dipakai di banyak workspace (contoh: `zod`) idealnya satu versi. Cek:

```bash
npm ls zod
```

**Output yang diharapkan:** satu versi. Jika banyak versi, selaraskan.

## 2. Update Shared Package

Perubahan `packages/shared` berdampak semua consumer. Urutan:

1. Buat perubahan backward-compatible di shared.
2. Build shared:
   ```bash
   npx turbo run build --filter=@santriverse/shared
   ```
3. Typecheck consumer:
   ```bash
   npx turbo run lint --filter=@santriverse/web --filter=@santriverse/api
   ```
4. Test consumer:
   ```bash
   npx turbo run test --filter=@santriverse/web --filter=@santriverse/api
   ```
5. Build root:
   ```bash
   npx turbo run build
   ```

Jika perubahan breaking:
- Buat versi baru type/schema, biarkan lama tetap ada.
- Migrasi consumer satu per satu.
- Hapus type lama hanya setelah semua consumer berpindah.

## 3. Turborepo Cache

### Local cache

Cache tersimpan di `node_modules/.cache/turbo`. Task yang input tidak berubah menampilkan `FULL TURBO` atau `cache hit`.

Jika cache tidak pernah hit, periksa:
- `outputs` di `turbo.json` sudah mencakup semua artifact build.
- Environment variable yang memengaruhi output terdaftar di `env`/`globalEnv`.

### Remote cache (opsional)

Turbo mendukung remote cache melalui Vercel atau custom server. Berguna untuk CI dan tim banyak developer. Setup:

```bash
npx turbo login
npx turbo link
```

Ikuti dokumentasi Turborepo terbaru: <https://turbo.build/repo/docs/core-concepts/remote-caching>.

### Bersihkan cache

```bash
npx turbo run build --force
```

Membangun ulang semuanya tanpa cache. Gunakan saat curiga cache corrupt.

## 4. Monitoring Biaya (Dua Platform)

Catat mingguan:

| Platform | Komponen | Yang dicatat |
|---|---|---|
| Vercel | Web | Bandwidth, function invocation, build minutes |
| Railway/Render | API | RAM-hours, CPU, egress, storage DB |
| Total | — | Proyeksi akhir bulan gabungan |

Tindakan hemat:
- Web: optimalkan image, lazy load, cache header, ISR/SSG di mana bisa.
- API: pagination response, batasi batch size, matikan staging saat tidak dipakai.
- Shared: tidak menambah biaya langsung, tetapi bloat shared memperlambat build.

Harga platform berubah. Cek pricing page sebelum berkomitmen ke klien.

## 5. Monitoring Operasional

### Log

- Vercel: **Deployments → Logs** atau `vercel logs`.
- Railway: **Deployments → Logs** atau `railway logs`.

Cari pola:
- Error 500 berulang.
- CORS rejection.
- Timeout API.
- Cold start lambat (Vercel serverless).

### Health check

```bash
curl -i https://api.domainmu.com/health
curl -I https://app.domainmu.com
```

Pertimbangkan uptime monitor eksternal (UptimeRobot, Better Stack, dsb.) untuk alerting.

## 6. Backup Database

Sama seperti Paket G. Jika API memakai PostgreSQL di Railway:

```bash
pg_dump --format=custom --no-owner --no-acl "$DATABASE_PUBLIC_URL" > "backup-$(date +%Y-%m-%d).dump"
pg_restore --list "backup-YYYY-MM-DD.dump"
```

Uji restore ke database terpisah minimal bulanan.

## 7. Jadwal Operasional

### Mingguan

- [ ] Baca log error web dan API
- [ ] Cek health kedua domain
- [ ] Cek billing Vercel dan Railway/Render
- [ ] Jalankan `npx turbo run lint build` dari root

### Bulanan

- [ ] `npm outdated` + `npm audit`
- [ ] Update patch/minor dependency
- [ ] Backup database + uji restore
- [ ] Review shared package—apakah ada kode yang seharusnya pindah ke app?
- [ ] Review Turborepo cache hit rate
- [ ] Review akses GitHub, Vercel, Railway/Render, Cloudflare
- [ ] Rotasi secret jika diperlukan

### Kuartalan

- [ ] Evaluasi: apakah monorepo masih memberi nilai? Atau sudah waktunya split?
- [ ] Review major version dependency
- [ ] Performance audit web (Lighthouse/CWV)

## Runbook Insiden Monorepo

1. Identifikasi service mana yang bermasalah: web, API, atau keduanya.
2. Cek health API dahulu—web bergantung pada API.
3. Jika API sehat dan web gagal: masalah Vercel/build/env web.
4. Jika API down: cek Railway/Render logs, database, env vars.
5. Jika keduanya down setelah deploy shared: rollback deploy API, lalu web.
6. Backup database sebelum eksperimen perbaikan data.
7. Dokumentasikan akar masalah dan pencegahan.

## Checklist Kelulusan

- [ ] `npm outdated` terakhir dijalankan dan dicatat
- [ ] Shared update diuji pada dua consumer
- [ ] Cache Turborepo terverifikasi (ada cache hit)
- [ ] Backup database berhasil dan restore diuji
- [ ] Biaya bulanan dua platform tercatat
- [ ] Jadwal operasional punya penanggung jawab
- [ ] Runbook insiden dipahami operator

✅ **Paket H selesai.** Kembali ke [README Paket H](README.md) dan kumpulkan bukti kelulusan.
