# 02 — Migrasi, Backup, dan Restore

## Tujuan

Perubahan skema aman dan data bisa dipulihkan ketika gagal.

## Migrasi Aman

Perubahan berisiko dilakukan bertahap:

1. Tambah kolom baru sebagai nullable/default aman.
2. Deploy kode yang menulis kolom lama + baru bila perlu.
3. Backfill data.
4. Deploy kode yang membaca kolom baru.
5. Baru hapus kolom lama pada rilis berikutnya.

Jangan rename/drop kolom besar dalam satu deploy tanpa rencana rollback.

## Backup PostgreSQL

Backup custom format:

```bash
pg_dump --format=custom --no-owner --file=backup.dump "$DATABASE_URL"
```

Restore ke database kosong:

```bash
pg_restore --clean --if-exists --no-owner --dbname="$RESTORE_DATABASE_URL" backup.dump
```

Perintah restore dapat menimpa data target. Pastikan URL target benar dan bukan
produksi sebelum menjalankan.

## Bukti Backup yang Benar

File backup ada belum berarti bisa dipakai. Minimal bulanan:

1. Buat database uji kosong.
2. Restore backup terbaru ke DB uji.
3. Jalankan query jumlah tabel/record penting.
4. Buka aplikasi dengan DB uji.
5. Catat tanggal dan hasil restore drill.

## Kebijakan Minimum

- Backup otomatis harian.
- Simpan 7 harian + 4 mingguan.
- Salinan di lokasi berbeda dari VPS.
- Enkripsi storage backup.
- Jangan commit dump DB ke Git.

## Checklist

- [ ] Migrasi destruktif dipisah beberapa rilis
- [ ] Backup otomatis aktif
- [ ] Backup di luar VPS
- [ ] Restore pernah diuji nyata
- [ ] Dump DB tidak masuk repo
