# G7 — Maintenance Railway

> 🔵 MENENGAH · Rutinitas: mingguan + bulanan

## Tujuan

Menjaga aplikasi sehat, database dapat dipulihkan, dan biaya Railway terkendali.

## 1. Logs: Diagnosis Pertama

Buka service aplikasi → **Deployments/Logs**. Dengan Railway CLI:

```bash
railway logs
```

Cari sinyal berikut:

| Sinyal | Arti | Tindakan |
|---|---|---|
| Deploy berulang | Crash loop | Baca error pertama, bukan baris terakhir |
| `P1001` / timeout DB | Koneksi database bermasalah | Cek service Postgres dan variable reference |
| Banyak respons 500 | Bug runtime | Catat endpoint, request ID, timestamp |
| `JavaScript heap out of memory` | RAM kurang/kebocoran | Profiling; batasi batch; naikkan resource jika terbukti |
| Cron jalan dua kali | Lebih dari satu replica | Pindah cron ke service tunggal/locking DB |

Jangan log password, token, cookie, isi `DATABASE_URL`, atau data pribadi lengkap.

## 2. Metrics: Pantau Tren

Railway service → **Metrics**. Pantau:

- CPU: lonjakan sesaat wajar; tinggi terus perlu investigasi.
- Memory: naik terus tanpa turun mengarah ke leak.
- Network egress: lonjakan dapat meningkatkan biaya.
- Restart/deploy: restart tak terencana harus dijelaskan.
- Disk database: pertumbuhan menentukan retensi dan backup.

Catat baseline setelah satu minggu. Alarm bermakna harus dibandingkan baseline, bukan angka tebak.

## 3. Backup PostgreSQL

### Backup manual terenkripsi/terlindungi

Ambil public connection string sementara dari Railway Postgres saat diperlukan. Jangan tampilkan nilainya di terminal history bersama perintah. Set sebagai environment variable sesi melalui prompt shell aman atau konfigurasi lokal yang diabaikan.

```bash
pg_dump --format=custom --no-owner --no-acl "$DATABASE_PUBLIC_URL" > "backup-$(date +%Y-%m-%d).dump"
```

**Output yang diharapkan:** perintah tanpa error dan file `.dump` berukuran lebih dari 0 byte.

Cek isi tanpa restore:

```bash
pg_restore --list "backup-YYYY-MM-DD.dump"
```

**Output yang diharapkan:** daftar tabel, sequence, constraint.

### Uji restore

Buat database lokal kosong:

```bash
createdb produk_restore_test
pg_restore --no-owner --no-acl --dbname=produk_restore_test "backup-YYYY-MM-DD.dump"
psql produk_restore_test -c '\dt'
```

**Output yang diharapkan:** tabel Prisma/Product/Order terlihat. Backup yang belum pernah diuji restore belum bisa disebut backup.

Hapus database uji setelah verifikasi:

```bash
dropdb produk_restore_test
```

Simpan backup di lokasi terenkripsi, terpisah dari Railway dan repo GitHub. Periksa fitur backup/snapshot bawaan sesuai plan Railway terbaru.

## 4. Update Aman

Urutan bulanan:

```bash
npm outdated
npm audit
npm run lint
npm run build
```

- Update patch/minor dulu dan baca changelog.
- Untuk major version, buat task migrasi terpisah.
- Jangan menjalankan `npm audit fix --force` tanpa review; dapat mematahkan API.
- Setelah deploy, smoke test health, list produk, dan create order.

## 5. Migrasi Database Aman

Prinsip expand–migrate–contract:

1. **Expand:** tambah kolom nullable/tabel baru; deploy kompatibel.
2. **Migrate:** isi data lama lewat job terbatas dan terukur.
3. **Contract:** hapus kolom lama hanya setelah semua kode berhenti memakainya.

Sebelum migrasi berisiko:

```bash
pg_dump --format=custom --no-owner --no-acl "$DATABASE_PUBLIC_URL" > pre-migration.dump
npx prisma migrate diff --from-url "$DATABASE_PUBLIC_URL" --to-schema-datamodel prisma/schema.prisma --script
```

Review SQL. Setelah backup dan review, deploy menjalankan:

```bash
npx prisma migrate deploy
```

## 6. Monitoring Biaya

Railway → **Usage/Billing** (nama menu dapat berubah). Setiap minggu catat:

| Komponen | Yang dicatat |
|---|---|
| Web service | RAM-hours, CPU, egress |
| PostgreSQL | RAM, storage, egress |
| Total | proyeksi akhir bulan |

Tindakan hemat:

- Hindari polling frontend setiap detik; gunakan interval wajar atau WebSocket.
- Batasi query, pagination, dan ukuran response.
- Optimalkan image lewat CDN/object storage, bukan DB.
- Batasi worker concurrency.
- Matikan service staging saat tidak dipakai bila kebijakan Railway mendukung.
- Set spending alert/usage limit bila tersedia pada akun saat ini.

Harga dan fitur berubah. Periksa halaman pricing Railway sebelum menyepakati biaya dengan klien.

## 7. Jadwal Operasional

### Mingguan

- [ ] Baca error log 7 hari terakhir
- [ ] Cek restart/crash
- [ ] Cek CPU, memory, egress, storage
- [ ] Cek proyeksi biaya
- [ ] Jalankan smoke test produksi

### Bulanan

- [ ] Buat backup dan verifikasi daftar isinya
- [ ] Uji restore ke database terpisah
- [ ] Jalankan `npm outdated` dan `npm audit`
- [ ] Review akses GitHub, Railway, Cloudflare
- [ ] Review secret yang perlu dirotasi
- [ ] Update catatan insiden dan baseline metrics

## Runbook Insiden Singkat

1. Catat waktu, gejala, dan deploy terakhir.
2. Cek `/api/health`, Railway status, application logs, lalu database logs.
3. Jika deploy baru penyebabnya, rollback ke deployment sehat.
4. Jika data berisiko, hentikan write path sebelum eksperimen.
5. Pulihkan dari backup hanya setelah penyebab dan target restore jelas.
6. Setelah pulih, tulis akar masalah dan pencegahan.

## Checklist Kelulusan

- [ ] Dapat menemukan error request pada logs
- [ ] Baseline CPU/memory tercatat
- [ ] `pg_dump` berhasil
- [ ] `pg_restore --list` berhasil
- [ ] Restore uji berhasil dan tabel terbaca
- [ ] Backup disimpan di luar repo dan luar Railway
- [ ] Proyeksi biaya bulanan tercatat
- [ ] Runbook insiden diketahui operator
- [ ] Jadwal mingguan dan bulanan punya penanggung jawab

✅ **Paket G selesai.** Kembali ke [README Paket G](README.md) dan kumpulkan bukti kelulusan.
