# 01 — Checklist Maintenance Bulanan

Website tidak selesai saat deploy. Jalankan checklist ini sebulan sekali.

## Keamanan
- [ ] Cek update dependency: `npm outdated` / Dependabot
- [ ] Review security advisory, update yang relevan
- [ ] Cek akun admin yang masih aktif, hapus yang tidak perlu
- [ ] Cek log login mencurigakan
- [ ] Rotasi secret yang terindikasi bocor (bukan rotasi tanpa sebab)
- [ ] Cek 2FA pemilik/admin aktif

## Infrastruktur
- [ ] Cek disk, RAM, CPU, bandwidth
- [ ] Cek domain expiry (auto-renew + payment valid)
- [ ] Cek SSL expiry / auto-renew
- [ ] Cek uptime 30 hari
- [ ] Cek health endpoint
- [ ] Cek cron/background job terakhir sukses

## Database & Backup
- [ ] Backup otomatis masih jalan
- [ ] Ukuran backup masuk akal (bukan 0 byte)
- [ ] Download satu backup ke lokasi terpisah
- [ ] Restore drill ke DB uji (minimal per kuartal)
- [ ] Cek pertumbuhan DB / tabel terbesar

## Aplikasi
- [ ] Jalankan lint + test + build di branch maintenance
- [ ] Uji register/login/logout
- [ ] Uji alur utama (checkout kalau ada)
- [ ] Cek email transaksional sampai inbox
- [ ] Cek broken link / 404 utama
- [ ] Cek Lighthouse Mobile

## Bisnis
- [ ] Harga produk benar
- [ ] Link WhatsApp/contact aktif
- [ ] Policy/refund masih sesuai proses nyata
- [ ] Akun payment aktif, settlement normal

## Log

Buat catatan:
```text
Tanggal:
Pemeriksa:
Masalah ditemukan:
Tindakan:
Next review:
```

Masalah yang tidak langsung diperbaiki masuk issue tracker, bukan diingat di kepala.
