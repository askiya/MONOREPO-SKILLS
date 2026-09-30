# 02 — Deploy dengan Coolify

## Tujuan

Deploy dari GitHub ke VPS sendiri lewat panel, dengan HTTPS otomatis.

## Kenapa Coolify

Self-hosted PaaS: build dari repo, kelola env, domain, SSL, database, dan log
dalam satu panel. Cocok ketika ingin pengalaman mirip Vercel tetapi di VPS
sendiri.

## Instalasi

Prasyarat: VPS sudah diamankan sesuai bab sebelumnya.

Ikuti installer resmi dari dokumentasi Coolify pada versi terkini. Setelah
instalasi selesai, buka panel di port yang ditunjukkan installer dan buat akun
admin pertama segera. Jangan biarkan panel tanpa akun admin.

Amankan panel:

- gunakan domain + HTTPS untuk panel,
- password kuat dan unik,
- aktifkan 2FA bila tersedia,
- batasi akses bila memungkinkan.

## Hubungkan GitHub

1. Panel Coolify → Sources → tambah GitHub App.
2. Beri akses hanya ke repo yang diperlukan.
3. Selesaikan proses otorisasi.

## Deploy Aplikasi

1. Project → Add Resource → Application.
2. Pilih repo dan branch (`main` untuk produksi, `develop` untuk staging).
3. Build pack: Nixpacks untuk app Node umum, atau Dockerfile bila ada.
4. Port aplikasi sesuai app (`3000` untuk Next.js default).
5. Isi Environment Variables dari `.env` produksi.
6. Set domain, aktifkan SSL otomatis.
7. Deploy dan baca log build sampai selesai.

## Database di Coolify

1. Add Resource → PostgreSQL.
2. Catat kredensial dari panel.
3. Hubungkan lewat jaringan internal bila app satu server.
4. Aktifkan backup terjadwal.
5. Uji restore secara berkala.

## Deploy Otomatis

Aktifkan webhook/auto-deploy agar push ke branch memicu build. Gunakan branch
terpisah untuk staging dan produksi.

## Membaca Kegagalan Deploy

Baca log mentah, jangan menebak:

1. Gagal saat install dependency → masalah lockfile/registry/versi Node.
2. Gagal saat build → error TypeScript/konfigurasi, uji `npm run build` lokal.
3. Build sukses, container restart → env kurang, port salah, atau crash runtime.
4. Domain tidak bisa diakses → DNS belum benar atau port/proxy salah.

Jangan mengganti secret atau memaksa push tanpa bukti dari log.

## Checklist

- [ ] Panel Coolify aman dan punya admin
- [ ] Repo terhubung dengan akses terbatas
- [ ] App deploy sukses dan bisa diakses
- [ ] HTTPS aktif
- [ ] Database punya backup terjadwal
- [ ] Prosedur baca log dipahami
