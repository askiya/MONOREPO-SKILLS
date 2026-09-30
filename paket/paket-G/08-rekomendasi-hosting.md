# G8 — Rekomendasi Hosting Railway Fullstack

> 🔵 MENENGAH · Stack: Next.js + Railway PostgreSQL + Railway Deploy

## Tujuan

Memilih plan Railway atau alternatif PaaS yang sesuai untuk aplikasi fullstack Paket G, memahami komponen biaya, dan memasang domain produksi melalui Cloudflare.

## Prasyarat

- Aplikasi Next.js lulus `npm run build`.
- Repo tersedia di GitHub dan tidak menyimpan `.env` atau secret.
- Sudah memperkirakan RAM, CPU, penyimpanan database, dan trafik keluar.
- Sudah membaca [`05-deploy.md`](05-deploy.md), [`06-domain-ssl.md`](06-domain-ssl.md), dan [`07-maintenance.md`](07-maintenance.md).

> Harga, kredit, kuota, free tier, dan tampilan dashboard dapat berubah. Verifikasi angka terbaru di [dokumentasi harga Railway](https://railway.com/pricing) sebelum memilih plan. Angka di bawah adalah titik awal perencanaan, bukan jaminan tagihan.

## 1. Rekomendasi Utama: Railway

Railway cocok bila ingin menjalankan Next.js, PostgreSQL, cron, worker, atau WebSocket tanpa mengelola VPS, Docker daemon, Nginx, dan Certbot sendiri.

Kelebihan utama:

- Deploy otomatis dari GitHub.
- PostgreSQL, MySQL, Redis, dan MongoDB tersedia sebagai service.
- Private networking antarservice dalam satu project.
- Environment variable dan reference variable dikelola dari dashboard.
- Log, metrics, custom domain, volume, dan rollback deployment tersedia.
- Paid plan tidak memakai cold start seperti layanan free yang tidur saat tidak aktif.

Tetap ada tanggung jawab pengguna: migrasi database, desain backup, kontrol biaya, keamanan aplikasi, rotasi secret, observability, dan pengujian restore.

## 2. Plan Railway

| Plan | Biaya/kredit awal | Cocok untuk | Catatan |
|---|---:|---|---|
| **Free Trial** | Kredit US$5 sekali, berlaku sekitar 30 hari | Evaluasi dan belajar | Berakhir saat waktu atau kredit habis; cek syarat akun terbaru |
| **Free** | Kredit sekitar US$1/bulan | Demo sangat kecil | Sangat terbatas; jangan mengandalkan kapasitas ini untuk produksi |
| **Hobby** | US$5/bulan, termasuk US$5 usage credit | Project pribadi dan produksi kecil | Pemakaian di atas kredit ditagihkan sesuai usage |
| **Pro** | US$20/bulan, termasuk US$20 usage credit | Tim dan workload produksi lebih serius | Periksa fitur tim, limit, dan dukungan terbaru |

Biaya dasar plan bukan selalu batas maksimal tagihan. Railway menggunakan model usage-based: tagihan meningkat saat aplikasi memakai resource melebihi kredit yang termasuk.

## 3. Komponen Biaya Railway

Gunakan angka berikut sebagai perkiraan awal:

| Resource | Perkiraan harga |
|---|---:|
| RAM | US$10/GB/bulan |
| CPU | US$20/vCPU/bulan |
| Network egress | US$0,05/GB |
| Volume | US$0,15/GB/bulan |

Contoh perhitungan konseptual:

```text
biaya aplikasi + biaya PostgreSQL + volume + egress - kredit plan
```

Tagihan nyata bergantung pada rata-rata pemakaian dan durasi service. Jangan mengalikan limit maksimum seolah resource selalu terpakai penuh. Gunakan halaman usage Railway sebagai sumber tagihan aktual.

### Langkah mengendalikan biaya

1. Mulai dengan satu service aplikasi dan satu PostgreSQL.
2. Pantau **Usage** dan **Metrics** setiap hari selama minggu pertama.
3. Pasang budget alert atau spending limit bila tersedia pada plan saat ini.
4. Hapus environment, volume, dan service eksperimen yang tidak digunakan.
5. Hindari menyimpan file upload pengguna di filesystem aplikasi; gunakan object storage.
6. Optimalkan query dan connection pool sebelum menaikkan resource database.
7. Pantau egress untuk response besar, image, video, dan download file.

## 4. PostgreSQL di Railway

Railway PostgreSQL cocok untuk Paket G karena provisioning cepat dan aplikasi dapat memakai private network. Untuk aplikasi kecil, penggunaan database dapat berada sekitar US$1–4/bulan, tetapi angka ini berubah mengikuti RAM, CPU, volume, backup, dan aktivitas nyata.

Gunakan reference variable dari service PostgreSQL:

```text
DATABASE_URL=${{Postgres.DATABASE_URL}}
```

Nama service dan sintaks yang ditampilkan dashboard dapat berbeda. Pilih reference dari UI agar nilainya tidak disalin manual.

Aturan produksi:

- Gunakan private URL untuk aplikasi dalam project Railway yang sama.
- Jangan membuka public URL database kecuali untuk tugas administrasi sementara.
- Jalankan `npx prisma migrate deploy`, bukan `prisma migrate dev`.
- Buat backup di lokasi terpisah dari Railway.
- Uji restore ke database non-produksi.
- Periksa fitur backup/snapshot yang tersedia pada plan terbaru.
- Pantau ukuran volume dan jumlah koneksi.

## 5. Alternatif Managed Database

| Database | Kelebihan | Perhatian | Cocok ketika |
|---|---|---|---|
| **Railway PostgreSQL** | Satu project, private networking, setup cepat | Usage-based; backup tetap harus diverifikasi | Ingin operasional paling sederhana di Railway |
| **Neon** | Free tier; autoscaling/serverless PostgreSQL; branching | Latensi dan cold behavior harus diuji dari region aplikasi | Ingin free tier atau branching database |
| **Supabase** | Free tier; PostgreSQL plus auth, storage, dan API | Jangan memakai fitur tambahan tanpa kebutuhan; pahami limit free tier | Aplikasi juga membutuhkan auth/storage Supabase |

Jika database berada di Neon atau Supabase sementara aplikasi berada di Railway, koneksi melewati jaringan publik/TLS. Pilih region yang dekat, wajibkan SSL, dan gunakan connection pooler bila direkomendasikan provider.

## 6. Perbandingan Railway dan Alternatif PaaS

| Platform | Harga awal | Database | Cold start | Kelebihan | Perhatian |
|---|---:|---|---|---|---|
| **Railway** | Trial/free terbatas; Hobby US$5/bulan | PostgreSQL, MySQL, Redis, MongoDB | Tidak ada cold start pada paid plan | UX sederhana, deploy GitHub, private network | Usage dapat naik; pantau resource |
| **Render** | Web Starter sekitar US$7/bulan | Managed PostgreSQL mulai sekitar US$7/bulan | Free web service dapat tidur; bangun sekitar 30–50 detik | Mudah, blueprint, managed services | Free tier tidak cocok untuk respons produksi konsisten |
| **Fly.io** | Usage-based | Bisa menjalankan Postgres/cluster sesuai produk terkini | Bergantung konfigurasi machine | Banyak region, kontrol runtime dan jaringan lebih tinggi | Operasional lebih teknis; biaya perlu dihitung dari usage |
| **DigitalOcean App Platform** | Mulai sekitar US$5/bulan | Managed database terpisah | Bergantung kelas layanan | Dokumentasi baik, ekosistem DO | Database dan resource tambahan meningkatkan total biaya |

### Railway vs Render

Pilih **Railway** bila:

- Ingin aplikasi dan database dalam satu project dengan private reference.
- Membutuhkan worker, cron, atau WebSocket tanpa cold start pada paid plan.
- Lebih nyaman dengan billing usage-based.

Pilih **Render** bila:

- Tim sudah memakai ekosistem Render.
- Blueprint dan pemisahan layanan Render sesuai workflow.
- Cold start free tier 30–50 detik dapat diterima untuk demo, bukan produksi yang perlu respons konsisten.

### Railway vs Fly.io

Pilih **Fly.io** bila butuh deployment lebih dekat ke banyak region dan siap mengelola konfigurasi machine, volume, jaringan, dan database dengan lebih teknis. Untuk pemula PaaS, Railway biasanya lebih cepat dipahami.

### Railway vs DigitalOcean App Platform

Pilih DigitalOcean bila sudah memakai Spaces, managed database, atau resource DigitalOcean lain. Hitung total aplikasi dan database; harga aplikasi awal bukan total stack fullstack.

## 7. Pilihan Berdasarkan Skenario

| Skenario | Rekomendasi |
|---|---|
| Belajar selama kurang dari 30 hari | Railway Free Trial, dengan pemantauan kredit |
| Demo sangat kecil dan boleh terbatas | Railway Free atau Render free |
| Produksi kecil tanpa cold start | Railway Hobby |
| Tim, beberapa service, penggunaan lebih besar | Railway Pro setelah estimasi usage |
| Perlu free PostgreSQL terpisah | Railway + Neon/Supabase free tier |
| Perlu banyak region dan kontrol runtime | Fly.io |
| Sudah memakai ekosistem DigitalOcean | DigitalOcean App Platform + managed database |
| Perlu biaya bulanan sangat tetap | Bandingkan paket fixed-price atau VPS; Railway usage-based mungkin kurang cocok |

## 8. Langkah Memilih Plan Railway

### Langkah 1 — Deploy staging

Gunakan Free Trial atau plan yang berlaku untuk staging. Hubungkan GitHub, tambah PostgreSQL, dan jalankan migrasi sesuai [`05-deploy.md`](05-deploy.md).

### Langkah 2 — Ukur resource nyata

Selama minimal beberapa hari, pantau:

- Rata-rata dan puncak RAM aplikasi.
- Rata-rata dan puncak CPU.
- RAM, CPU, volume, dan koneksi PostgreSQL.
- Egress per hari.
- Waktu build dan frekuensi deploy.
- Jumlah restart dan respons 5xx.

### Langkah 3 — Proyeksikan tagihan

Gunakan halaman **Usage** Railway, bukan angka tebak. Proyeksikan traffic normal dan lonjakan kampanye. Sisakan margin anggaran.

### Langkah 4 — Pilih plan

- **Hobby** untuk individu atau produksi kecil yang muat dalam pola penggunaan rendah.
- **Pro** bila fitur tim, limit, atau usage yang lebih besar memang dibutuhkan.
- Jangan memilih Pro hanya agar terlihat “production”. Pilih berdasarkan fitur dan resource yang terukur.

### Langkah 5 — Pasang kontrol

Aktifkan alert biaya, uptime monitor, backup eksternal, dan prosedur rollback sebelum mengarahkan domain produksi.

## 9. Domain dan DNS Cloudflare

Railway menyediakan domain bawaan untuk verifikasi. Untuk produksi, gunakan custom domain dengan DNS Cloudflare.

1. Di service aplikasi Railway, buka **Settings → Networking**.
2. Tambahkan custom domain, misalnya `app.domainmu.com`.
3. Railway menampilkan target DNS yang harus dibuat.
4. Buka Cloudflare → **DNS**.
5. Buat record persis sesuai instruksi Railway, biasanya `CNAME`.
6. Awali dengan status **DNS only** bila validasi atau penerbitan sertifikat bermasalah.
7. Tunggu domain berstatus aktif di Railway.
8. Uji HTTPS:

```bash
curl -I https://app.domainmu.com
```

9. Jika ingin mengaktifkan proxy Cloudflare, lakukan setelah HTTPS Railway valid lalu uji login, callback OAuth, webhook, upload, dan WebSocket.

Jangan membuat record `A` ke IP sementara bila Railway meminta `CNAME`. Jangan menerbitkan sertifikat manual dengan Certbot; TLS custom domain ditangani Railway.

Rujukan lengkap: [`06-domain-ssl.md`](06-domain-ssl.md).

## 10. Peringatan Produksi

- **Railway paid tidak cold start.** Aplikasi berbayar tetap dapat restart saat deploy, crash, atau maintenance; sediakan health check dan retry yang aman.
- **Render free dapat cold start sekitar 30–50 detik.** Jangan gunakan untuk endpoint produksi yang membutuhkan respons konsisten, webhook sensitif waktu, atau job yang harus selalu aktif.
- Kredit plan bukan resource gratis tanpa batas. Pemakaian di atas kredit dapat menambah tagihan.
- Free Trial berakhir. Siapkan plan pembayaran atau jalur migrasi sebelum demo penting.
- Volume tidak sama dengan backup. Simpan `pg_dump` di lokasi eksternal dan uji restore.
- Jangan menyimpan upload pengguna pada filesystem ephemeral aplikasi.
- Jangan memasukkan `DATABASE_URL`, token, atau secret ke GitHub, log, screenshot, atau variable `NEXT_PUBLIC_*`.
- Jangan menggunakan public database URL dari aplikasi bila private networking tersedia.
- Pastikan cron hanya berjalan pada satu replica atau memakai database lock.
- Aktifkan MFA pada Railway, GitHub, Cloudflare, dan provider database.
- Hapus preview environment dan volume lama agar biaya tidak terus berjalan.

## 11. Verifikasi Sebelum Produksi

Uji domain dan health endpoint:

```bash
curl -fsS https://app.domainmu.com/api/health
```

**Hasil yang diharapkan:** HTTP 200 dan status aplikasi/database sehat.

Verifikasi deployment:

1. Push perubahan kecil ke branch deployment.
2. Pastikan Railway melakukan auto deploy.
3. Pastikan build dan pre-deploy migration sukses.
4. Pastikan deployment lama diganti tanpa data hilang.
5. Jalankan smoke test login, baca data, tulis data uji, dan logout.
6. Periksa log: tidak ada secret, crash loop, atau error koneksi database.
7. Periksa Usage: biaya sesuai proyeksi.

## Checklist Akhir

- [ ] Harga dan kuota Railway terbaru sudah diperiksa di situs resmi.
- [ ] Plan dipilih berdasarkan usage nyata, bukan hanya biaya dasar.
- [ ] Aplikasi dan PostgreSQL berada di region yang sesuai.
- [ ] Aplikasi memakai private reference ke Railway PostgreSQL.
- [ ] Migrasi produksi memakai `prisma migrate deploy` atau perintah setara.
- [ ] Backup database berada di luar Railway.
- [ ] Restore backup sudah diuji.
- [ ] Usage RAM, CPU, egress, dan volume sudah dipantau.
- [ ] Alert biaya atau spending control sudah dipasang jika tersedia.
- [ ] Paid plan dipakai bila produksi tidak boleh mengalami cold start.
- [ ] Render free tidak dipakai untuk workload sensitif terhadap cold start.
- [ ] Custom domain aktif melalui Cloudflare dan Railway.
- [ ] HTTPS dan health endpoint mengembalikan HTTP 200.
- [ ] Auto deploy dari GitHub sudah diuji.
- [ ] MFA aktif pada semua akun infrastruktur.
- [ ] Tidak ada `.env`, secret, dump database, atau token dalam Git.
