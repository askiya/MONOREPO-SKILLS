# 07 — Maintenance: Supabase Monitoring, Backup & Vercel Analytics (Paket D)

## Tujuan

Aplikasi terpantau, database terbackup, quota tidak terlewati, dan update
bisa dilakukan tanpa merusak data production.

---

## Sebelum Mulai

- [ ] Aplikasi online dengan custom domain + HTTPS ([06-domain-ssl.md](06-domain-ssl.md))

---

## 1. Monitoring Supabase Dashboard

### Database

Supabase Dashboard → **Database** → **Reports**:

| Metrik | Yang dicek | Waspada kalau |
|---|---|---|
| Database Size | Ukuran total DB | Mendekati quota plan |
| Connections | Koneksi aktif | Mendekati connection limit |
| Query Performance | Query lambat | Query > 1 detik berulang |
| Table Size | Tabel terbesar | Satu tabel tumbuh tidak wajar |

### Auth

Supabase Dashboard → **Authentication** → **Users**:
- Jumlah user aktif
- User yang belum confirm email
- Login gagal berulang (indikasi brute force)

### Storage

Supabase Dashboard → **Storage**:
- Total storage used
- File terbesar
- File orphan (user sudah dihapus tapi file masih ada)

### API Logs

Supabase Dashboard → **Logs** → pilih service:

| Log | Kapan dicek |
|---|---|
| API | Request error 400/500 |
| Postgres | Query error, deadlock |
| Auth | Login gagal, token invalid |
| Storage | Upload gagal, permission denied |
| Edge Functions | Function error/timeout (kalau pakai) |

---

## 2. Backup Database

### Opsi A: Dashboard Backup (Plan Berbayar)

Supabase Dashboard → **Database** → **Backups**:
- Plan berbayar punya automated daily backups
- Cek retention period di plan kamu (bisa berubah)
- Test restore di project staging secara berkala

### Opsi B: Manual Backup via CLI (Semua Plan)

Install Supabase CLI:

```bash
npm install -g supabase
```

Login:

```bash
supabase login
```

Dump schema + data:

```bash
# Ambil connection string dari Supabase → Settings → Database
supabase db dump --db-url "<CONNECTION_STRING>" -f backup-$(date +%Y-%m-%d).sql
```

**Output yang benar:** file `backup-YYYY-MM-DD.sql` muncul.

> ⚠️ Connection string mengandung password. Jangan simpan di script atau Git.
> Masukkan via environment variable untuk automation.

### Opsi C: pg_dump Langsung

```bash
pg_dump "<CONNECTION_STRING>" \
  --format=custom \
  --file=backup-$(date +%Y-%m-%d).dump
```

### Jadwal Backup

| Frekuensi | Kapan |
|---|---|
| Mingguan | MVP aktif dengan data user |
| Harian | Sudah ada transaksi/data penting |
| Sebelum migrasi | Selalu — tanpa pengecualian |
| Sebelum deploy besar | Kalau ada perubahan schema |

Simpan backup di **tempat terpisah dari Supabase**:
- Cloud storage (Google Drive, S3, R2)
- Komputer lokal terenkripsi
- Minimal 3 versi terakhir

### Test Restore

Backup yang belum pernah direstore = belum terbukti berfungsi.

```bash
# Restore ke database staging, JANGAN production
pg_restore \
  --dbname="<STAGING_CONNECTION_STRING>" \
  --clean \
  --if-exists \
  backup-YYYY-MM-DD.dump
```

> ⚠️ `--clean` menghapus object lama sebelum restore. Hanya jalankan ke
> database staging/tes. Jangan ke production tanpa konfirmasi dan backup terbaru.

---

## 3. Vercel Analytics & Logs

### Aktifkan Analytics

1. Vercel Dashboard → Project → **Analytics**
2. Klik **Enable** (cek plan dan harga terbaru)
3. Install package kalau diminta:

```bash
npm install @vercel/analytics
```

Tambahkan ke root layout:

```tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="id">
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

Pantau:
- Page views
- Unique visitors
- Top pages
- Referrer
- Country

### Speed Insights (Opsional)

```bash
npm install @vercel/speed-insights
```

Tambahkan `<SpeedInsights />` di root layout.

### Function Logs

Vercel Dashboard → Project → **Logs**:
- Filter status 4xx/5xx
- Filter function name
- Cari error berulang

> Jangan log password, token, anon key, service role key, atau data pribadi user.

---

## 4. Update & Deploy

### Alur Update Aman

```text
1. Buat branch fitur
2. Edit + test lokal
3. npm run lint && npx tsc --noEmit && npm run build
4. Push → Vercel Preview Deployment
5. Test preview URL
6. Merge ke main → Vercel Production Deployment
7. Test production
8. Monitor logs 15 menit setelah deploy
```

### Migrasi Database

Sebelum perubahan schema:

1. Backup database production
2. Tulis migration SQL (jangan edit tabel manual tanpa catatan)
3. Test migration di local/staging
4. Jalankan ke production
5. Verifikasi RLS tetap aktif

Contoh migration:

```sql
-- supabase/migrations/20260930_add_product_status.sql
ALTER TABLE public.products
ADD COLUMN status TEXT NOT NULL DEFAULT 'draft'
CHECK (status IN ('draft', 'published', 'archived'));

-- RLS policy existing tetap berlaku karena policy di level tabel
```

Apply via CLI:

```bash
supabase db push
```

> Jalankan hanya setelah `supabase link --project-ref <ref>` dan backup selesai.

---

## 5. Security Check Bulanan

### RLS Audit

Jalankan di Supabase SQL Editor:

```sql
-- Cari tabel public tanpa RLS
SELECT schemaname, tablename, rowsecurity
FROM pg_tables
WHERE schemaname = 'public'
  AND rowsecurity = false;
```

**Output yang benar:** 0 rows (semua tabel punya RLS).

Cek policy:

```sql
SELECT schemaname, tablename, policyname, cmd, qual, with_check
FROM pg_policies
WHERE schemaname = 'public'
ORDER BY tablename, policyname;
```

Pastikan setiap tabel punya policy sesuai matriks di ARCHITECTURE.md.

### Secret Rotation

Rotate key kalau:
- Key pernah masuk Git
- Anggota tim keluar
- Ada aktivitas mencurigakan
- Sesuai kebijakan keamanan perusahaan (misal tiap 90 hari)

Setelah rotate:
1. Update env var di Vercel
2. Redeploy
3. Test login + database + storage
4. Hapus key lama

---

## 6. Quota & Biaya

Cek dashboard Supabase dan Vercel tiap bulan:

| Service | Metrik quota |
|---|---|
| Supabase | Database size, bandwidth, storage, MAU, Edge Function invocations |
| Vercel | Bandwidth, function invocations, build minutes, image optimization |

> Limit free tier dan harga cepat berubah. Jangan hardcode angka. Cek halaman
> pricing resmi Supabase dan Vercel saat melakukan review bulanan.

Setup billing alert kalau provider mendukung.

---

## 7. Uptime Monitoring

Pakai layanan eksternal (jangan hanya percaya dashboard Vercel):

1. Daftar [UptimeRobot](https://uptimerobot.com)
2. Tambah monitor HTTP(s): `https://domainmu.com`
3. Interval 5 menit
4. Alert ke email/Telegram
5. Tambah monitor untuk endpoint health kalau ada

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Database penuh | Data/log tumbuh tanpa kontrol | Hapus data lama, upgrade plan, buat retention policy |
| User tidak bisa login | Supabase Auth down atau URL config salah | Cek status.supabase.com + logs auth |
| Deploy baru rusak | Tidak test preview dulu | Rollback di Vercel → Deployments → Promote deployment lama |
| Data bocor antar user | RLS policy salah/tidak aktif | Audit RLS, fix policy segera, cek log akses |
| Backup gagal restore | Backup korup / versi mismatch | Test restore bulanan ke staging |
| Biaya naik tiba-tiba | Traffic/bot abuse | Cek bandwidth, aktifkan rate limiting/Cloudflare WAF |

---

## Checklist Maintenance Bulanan

- [ ] Cek Supabase database size + connections
- [ ] Cek Auth users (login gagal, unconfirmed users)
- [ ] Cek Storage usage + file orphan
- [ ] Cek Supabase Logs untuk error berulang
- [ ] Backup database ke lokasi terpisah
- [ ] Test restore backup terbaru ke staging (minimal per kuartal)
- [ ] Jalankan audit tabel tanpa RLS (harus 0 rows)
- [ ] Review semua RLS policies
- [ ] Cek Vercel Analytics + Function Logs
- [ ] Cek quota Supabase + Vercel
- [ ] Cek uptime monitor
- [ ] Update dependency security patch
- [ ] Cek domain expiry
