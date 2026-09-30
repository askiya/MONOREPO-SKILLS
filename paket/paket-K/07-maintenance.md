# 07 — Maintenance Santriverse Hybrid

## Tujuan

Menjaga empat lapisan tetap sehat: Cloudflare, GitHub/CI, Coolify/backend, dan
PostgreSQL.

## Monitoring Minimum

| Komponen | Yang dipantau | Alert |
|---|---|---|
| Frontend | `https://app.domainmu.com` | non-200 / timeout |
| Backend | `https://api.domainmu.com/up` | non-200 / latency tinggi |
| Coolify | container restart, CPU, RAM, disk | threshold + crash loop |
| PostgreSQL | koneksi, storage, backup | gagal backup / disk tinggi |
| GitHub Actions | test/lint/build | workflow gagal |
| Domain/SSL | expiry, DNS | sertifikat hampir expired |

## Backup PostgreSQL

Backup wajib:
- otomatis setiap hari,
- disimpan di lokasi eksternal, bukan hanya disk VPS yang sama,
- terenkripsi,
- retensi harian/mingguan,
- restore drill ke database uji minimal per kuartal.

Backup tanpa restore drill belum terbukti.

## Maintenance Bulanan

1. Cek Coolify dan OS update; baca release notes sebelum upgrade besar.
2. Cek disk VPS dan image/container lama.
3. Cek dependency frontend/backend (`npm outdated`, advisory).
4. Jalankan full gate pada branch maintenance.
5. Cek domain, SSL, dan billing provider.
6. Uji login, alur utama, webhook payment, dan email transaksional.
7. Verifikasi backup terbaru tidak 0 byte.
8. Restore satu backup ke environment uji.

## Log dan Privacy

- Jangan log password, token, cookie, atau connection string.
- Gunakan request/correlation ID agar request frontend→backend bisa ditelusuri.
- Pisahkan error teknis di log dari pesan aman untuk user.
- Tetapkan retensi log; jangan simpan selamanya tanpa alasan.

## Respons Insiden

Urutan saat API/DB bermasalah:

1. Cegah kerusakan meluas (maintenance mode / stop worker berbahaya).
2. Simpan bukti: timestamp, log, deployment/commit terakhir.
3. Rollback backend ke commit sehat bila perubahan kode penyebabnya.
4. Kalau secret bocor: cabut/rotate segera; menghapus dari Git tidak cukup.
5. Kalau DB rusak: jangan panik restore production langsung — uji backup di DB
   terpisah, baru jalankan recovery terkontrol.
6. Tulis postmortem: root cause, dampak, fix, pencegahan.

## Cost Check

Per bulan, cek:
- biaya VPS,
- bandwidth Cloudflare (jika layanan berbayar),
- storage backup,
- domain,
- API/provider eksternal.

Jangan optimasi biaya dari perkiraan. Gunakan metrik penggunaan nyata.

## Checklist

- [ ] Uptime monitor frontend + backend aktif
- [ ] Alert menuju channel yang dibaca
- [ ] Backup harian eksternal aktif
- [ ] Restore drill pernah berhasil
- [ ] Disk/CPU/RAM VPS dipantau
- [ ] CI wajib hijau sebelum production
- [ ] Dependency/security advisory direview bulanan
- [ ] Runbook insiden tersedia
- [ ] Biaya aktual direview bulanan
