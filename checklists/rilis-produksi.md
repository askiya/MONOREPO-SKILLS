# Checklist Rilis Produksi

## Kode

- [ ] `git status` dipahami; tidak ada file liar
- [ ] Diff hanya berisi scope rilis
- [ ] Tidak ada debug log, TODO kritis, fitur placeholder mati
- [ ] Lockfile ikut update bila dependency berubah

## Kualitas

- [ ] Test otomatis semua lulus
- [ ] Lint 0 error
- [ ] Build produksi sukses
- [ ] Manual QA alur kritis selesai
- [ ] Mobile 375px dan desktop diuji

## Keamanan

- [ ] Tidak ada secret di Git/diff/build output
- [ ] Environment produksi terisi
- [ ] Auth/role diuji 401 dan 403
- [ ] Input API divalidasi server
- [ ] Rate limit endpoint sensitif
- [ ] HTTPS aktif

## Database

- [ ] Migrasi ditinjau
- [ ] Perubahan destructive dipisah / disetujui
- [ ] Backup terbaru tersedia
- [ ] Restore pernah diuji
- [ ] Rollback schema dan app jelas

## Infrastruktur

- [ ] Domain dan DNS benar
- [ ] Health endpoint sehat
- [ ] Log runtime bisa diakses
- [ ] Monitoring/alert minimum aktif
- [ ] Kapasitas disk/RAM cukup

## Payment (bila ada)

- [ ] Sandbox lulus end-to-end
- [ ] Webhook signature diverifikasi
- [ ] Nominal/reference dicocokkan
- [ ] Idempotency diuji
- [ ] Transaksi kecil produksi berhasil

## Setelah Deploy

- [ ] URL produksi bisa diakses jaringan luar
- [ ] Register/login berhasil
- [ ] Alur utama berhasil
- [ ] Tidak ada error 5xx baru di log
- [ ] Versi/commit yang rilis dicatat
- [ ] Rollback tidak diperlukan / sudah dilakukan
