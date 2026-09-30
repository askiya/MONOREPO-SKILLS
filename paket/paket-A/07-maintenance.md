# 07 — Maintenance: Monitoring, Backup, Update

## Tujuan

Aplikasi terpantau 24/7, database dibackup, dan dependency di-update berkala.
Kamu tidur nyenyak karena kalau web mati, kamu tahu duluan — bukan user.

## Sebelum Mulai

- Aplikasi sudah live di Vercel dengan domain custom ([`06-domain-ssl.md`](06-domain-ssl.md))

Referensi:
[`../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md`](../../docs/20-operasional-serah-terima/01-maintenance-bulanan.md)

---

## 1. Monitoring Uptime — UptimeRobot

### Setup

1. Daftar di [uptimerobot.com](https://uptimerobot.com) (free tier: 50 monitor)
2. **Add New Monitor**:
   - Monitor Type: **HTTP(s)**
   - Friendly Name: nama project
   - URL: `https://namaproject.com`
   - Monitoring Interval: 5 minutes
3. **Alert Contacts** → tambahkan email dan/atau Telegram
4. Save

### Verifikasi

- Status monitor: **Up** (hijau)
- Tes: matikan project sementara di Vercel → tunggu 5 menit → harus dapat
  notifikasi email/Telegram bahwa monitor **Down**
- Aktifkan kembali → harus dapat notifikasi **Up**

### Monitor tambahan (opsional)

| URL | Tujuan |
|---|---|
| `https://namaproject.com/api/health` | Cek API masih hidup |
| `https://namaproject.com/login` | Cek halaman auth tidak error |

Kalau mau endpoint `/api/health`, minta agent buatkan:
```
Buat API route GET /api/health yang return JSON { "status": "ok", "timestamp": ISO datetime }.
Tidak perlu auth. Harus selalu return 200 kecuali server benar-benar mati.
```

---

## 2. Analytics — Vercel Analytics

### Setup

1. Dashboard Vercel → project → **Analytics** → **Enable**
2. Vercel menambahkan script analytics otomatis (zero-config untuk Next.js)

### Yang dipantau

- **Web Vitals:** LCP, FID, CLS — ukuran kecepatan dan kestabilan
- **Visitors:** jumlah pengunjung unik per hari
- **Top Pages:** halaman paling sering diakses

> Free tier cukup untuk project kecil. Cek limit terbaru di dashboard Vercel.

---

## 3. Backup Database Neon

### Neon Point-in-Time Restore

Neon free tier menyediakan point-in-time restore dengan retention terbatas.
Cek limit terbaru di dashboard Neon.

### Backup manual berkala

Jalankan tiap minggu atau sebelum deploy besar:

```bash
pg_dump "DATABASE_URL_KAMU" > backup-$(date +%Y%m%d).sql
```

> Ganti `DATABASE_URL_KAMU` dengan connection string Neon.
> Kalau `pg_dump` belum ada: install PostgreSQL client tools.

Cek backup valid:
```bash
head -20 backup-*.sql
```
Harus menampilkan SQL CREATE TABLE statements.

### Tempat simpan backup

- **Minimum:** satu folder di komputer lokal
- **Lebih baik:** upload ke Google Drive / Cloudflare R2 / S3
- **Jangan:** simpan backup di repo GitHub (bisa berisi data user)

---

## 4. Update Dependencies

### Cek outdated packages

Jalankan tiap 2 minggu:
```bash
npm outdated
```

### Update minor/patch (aman)

```bash
npm update
```

### Update major (hati-hati)

Cek changelog dulu, update satu per satu:
```bash
npm install package-name@latest
```

Setelah update:
```bash
npm run lint
npm run build
```

Kalau build gagal → revert, baca changelog, perbaiki breaking changes.

### Dependabot / Renovate (opsional)

Buat `.github/dependabot.yml`:
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

GitHub akan otomatis membuat PR untuk update dependency.

---

## 5. Jadwal Maintenance

| Frekuensi | Aksi |
|---|---|
| **Harian** | Cek UptimeRobot — ada downtime? |
| **Mingguan** | `pg_dump` backup database |
| **2 minggu** | `npm outdated` + update minor/patch |
| **Bulanan** | Review Vercel Analytics, cek biaya, update major (kalau perlu) |
| **3 bulan** | Review SSL expiry (biasanya auto-renew), cek Neon free tier limit |

---

## 6. Kapan Harus Upgrade dari Free Tier

| Tanda | Aksi |
|---|---|
| Vercel function timeout sering | Upgrade Vercel plan atau pindah ke Paket E/F |
| Neon connection limit tercapai | Upgrade Neon plan atau pakai connection pooling |
| Butuh background job / cron | Pindah ke Paket G (Railway) atau Paket E (Coolify) |
| Traffic > 100K/bulan | Evaluasi biaya, mungkin VPS lebih hemat |

---

## Kesalahan Umum

| Kesalahan | Akibat | Solusi |
|---|---|---|
| Tidak pasang monitoring | Web mati 3 hari, baru tahu dari user marah | Setup UptimeRobot hari ini |
| Tidak backup database | Data hilang permanen | `pg_dump` minimal mingguan |
| Update semua package sekaligus | Build rusak, tidak tahu mana yang salah | Update satu per satu, tes setiap kali |
| Backup disimpan di repo publik | Data user bocor | Simpan di tempat privat, bukan Git |

---

## Checklist

- [ ] UptimeRobot monitor aktif, URL produksi dipantau
- [ ] Alert contact diset (email / Telegram)
- [ ] Tes down/up notification berhasil
- [ ] Vercel Analytics diaktifkan
- [ ] Backup database pertama sudah dibuat (`pg_dump`)
- [ ] Backup disimpan di tempat yang aman (bukan repo publik)
- [ ] `npm outdated` dijalankan, dependency terbaru
- [ ] Jadwal maintenance dicatat di kalender
