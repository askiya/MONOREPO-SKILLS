# G5 — Deploy Next.js dan PostgreSQL ke Railway

> 🔵 MENENGAH · Target waktu: 1–2 jam

## Tujuan

Repo GitHub terhubung ke Railway, service PostgreSQL tersedia, migrasi produksi berjalan, dan aplikasi online.

> Tampilan dashboard dan harga Railway dapat berubah. Cocokkan label terbaru di dokumentasi resmi: <https://docs.railway.com/>.

## Prasyarat

- Repo aplikasi sudah tersedia di GitHub.
- `npm run build` lulus lokal.
- Akun Railway aktif.
- Tidak ada `.env` atau secret dalam repo.

## Langkah 1 — Buat Project dan Hubungkan GitHub

1. Buka <https://railway.app/> dan masuk.
2. Pilih **New Project**.
3. Pilih **Deploy from GitHub repo**.
4. Otorisasi Railway hanya ke repo yang dibutuhkan.
5. Pilih repo Paket G.

**Hasil yang diharapkan:** satu service aplikasi muncul dan build mulai. Build boleh gagal karena database belum ada.

## Langkah 2 — Tambah PostgreSQL

1. Di canvas project, pilih **New** / **Add Service**.
2. Pilih **Database → PostgreSQL**.
3. Tunggu status service aktif.

**Hasil yang diharapkan:** project memiliki dua service: aplikasi dan PostgreSQL.

Jangan menyalin public database URL bila aplikasi berada di project yang sama. Gunakan private reference Railway agar jaringan lebih cepat dan tidak keluar internet.

## Langkah 3 — Pasang Environment Variables

Buka service aplikasi → **Variables**. Tambahkan:

| Nama | Nilai |
|---|---|
| `DATABASE_URL` | Reference variable ke `DATABASE_URL` service PostgreSQL |
| `SESSION_SECRET` | Nilai acak minimal 32 karakter |
| `CRON_ENABLED` | `true` hanya jika job cron sudah aman |
| `NODE_ENV` | biasanya diatur Railway; jangan override bila tidak perlu |

Untuk membuat secret lokal tanpa membagikan hasilnya:

```bash
openssl rand -base64 48
```

Masukkan output langsung ke Railway, jangan ke chat, source code, atau screenshot.

**Hasil yang diharapkan:** redeploy terpicu setelah variable berubah.

## Langkah 4 — Atur Build dan Start

Railway biasanya mendeteksi Node otomatis. Jika perlu, buka pengaturan service:

```text
Build Command: npm run build
Start Command: npm start
Healthcheck Path: /api/health
```

Jangan isi port tetap. Railway menyuntikkan `PORT` saat runtime.

**Hasil yang diharapkan:** fase build menjalankan `prisma generate` lalu `next build`; runtime menunjukkan server siap.

## Langkah 5 — Jalankan Migrasi Produksi

Pilihan paling aman: set **Pre-deploy Command**:

```bash
npx prisma migrate deploy
```

Jika fitur itu tidak tersedia pada plan/UI saat ini, jalankan satu kali melalui Railway CLI setelah login:

```bash
npm install -g @railway/cli
railway login
railway link
railway run npx prisma migrate deploy
```

**Output yang diharapkan:** `All migrations have been successfully applied` atau `No pending migrations`.

Jangan gunakan `prisma migrate dev` atau `prisma db push` di produksi.

## Langkah 6 — Generate Public Domain

1. Service aplikasi → **Settings → Networking**.
2. Pilih **Generate Domain**.
3. Buka URL `https://...up.railway.app`.

Uji:

```bash
curl -i https://NAMA-SERVICE.up.railway.app/api/health
```

**Output yang diharapkan:** HTTP 200 dan database `ok`.

## Langkah 7 — Smoke Test Produksi

- Buka halaman produk.
- Buat satu pesanan uji.
- Pastikan data muncul pada DB produksi.
- Uji URL tidak valid.
- Periksa log setelah setiap request.

Hapus data uji bila tidak dibutuhkan.

## Langkah 8 — Atur Deploy Otomatis

Pastikan source service menunjuk branch produksi (umumnya `main`). Setiap push ke branch itu memicu build baru. Untuk project serius, gunakan branch staging/service terpisah sebelum produksi.

## Error Umum

| Log/gejala | Penyebab | Perbaikan |
|---|---|---|
| `P1001` | `DATABASE_URL` salah/tidak direferensikan | Hubungkan variable service Postgres |
| `P2021` | Migrasi belum diterapkan | Jalankan `prisma migrate deploy` |
| `Application failed to respond` | Start command atau port salah | Pakai `npm start`; jangan hardcode port |
| `prisma: command not found` | Prisma hanya di devDependencies dan production install membuangnya | Pastikan migration command punya binary Prisma |
| Health check 503 | DB belum siap/migrasi gagal | Baca deploy log dan Postgres log |
| Crash loop setelah cron aktif | Job melempar unhandled error | Bungkus job dengan error handling; matikan `CRON_ENABLED` sementara |

## Checklist

- [ ] Project Railway dibuat dari repo GitHub yang benar
- [ ] Dua service aktif: aplikasi + PostgreSQL
- [ ] `DATABASE_URL` memakai reference, bukan hardcode
- [ ] Secret hanya ada di Railway Variables
- [ ] Build dan start command benar
- [ ] `prisma migrate deploy` sukses
- [ ] Health check Railway berstatus sehat
- [ ] URL `*.up.railway.app` memberi HTTPS dan HTTP 200
- [ ] Create/read data produksi bekerja
- [ ] Log tidak menunjukkan crash loop

➡️ Lanjut ke **[06-domain-ssl.md](06-domain-ssl.md)**.
