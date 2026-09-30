# 08 — Rekomendasi Hosting Paket D

> ⚠️ **Harga dan fitur berubah sewaktu-waktu.** Angka di bawah adalah patokan awal, bukan penawaran tetap. Periksa halaman harga resmi sebelum membeli.

Paket D memakai **Next.js di Vercel** serta **Supabase untuk PostgreSQL, Auth, Storage, dan API**. Kombinasi ini cocok untuk MVP dan aplikasi SaaS yang ingin cepat online tanpa mengelola server.

## Arsitektur yang Direkomendasikan

```text
Pengguna
   │ HTTPS
   ▼
Cloudflare DNS
   │
   ▼
Vercel — Next.js
   │ Supabase SDK / HTTPS
   ▼
Supabase
   ├── PostgreSQL
   ├── Auth
   ├── Storage
   └── Realtime
```

Gunakan region Vercel dan Supabase yang berdekatan dengan mayoritas pengguna. Untuk pasar Indonesia, region Asia seperti Singapura biasanya memberi latensi lebih rendah daripada Eropa atau Amerika.

## 1. Frontend: Vercel

### Hobby — gratis

Cocok untuk belajar, prototipe, dan proyek pribadi nonkomersial.

- Deploy otomatis dari GitHub.
- Preview deployment untuk setiap branch atau pull request.
- Domain `*.vercel.app`, custom domain, dan HTTPS otomatis.
- Penggunaan tunduk pada batas kuota dan ketentuan nonkomersial Vercel.

> Jangan memakai Hobby untuk proyek komersial hanya karena trafik masih kecil. Gunakan paket yang sesuai dengan ketentuan Vercel.

### Pro — mulai US$20 per pengguna per bulan

Cocok untuk produk komersial, tim, dan aplikasi produksi.

- Kuota serta kemampuan operasional lebih besar daripada Hobby.
- Fitur kolaborasi dan observabilitas lebih lengkap.
- Biaya dapat bertambah karena jumlah anggota tim dan pemakaian di luar kuota.

## 2. Backend, Database, Auth, dan Storage: Supabase

### Free — US$0

Patokan kemampuan:

- Maksimal 2 proyek aktif.
- Database 500 MB.
- Storage 1 GB.
- Auth hingga 50.000 MAU.
- Proyek dapat dijeda setelah sekitar satu minggu tidak aktif.
- Tidak menyediakan backup terkelola yang layak dijadikan strategi pemulihan produksi.

Cocok untuk belajar, demo, dan MVP awal tanpa data penting. Jangan menganggap paket gratis sebagai tempat aman satu-satunya untuk data produksi.

### Pro — mulai US$25 per bulan

Patokan kemampuan:

- Database 8 GB.
- Storage 100 GB.
- Auth hingga 100.000 MAU.
- Backup harian.
- Point-in-time recovery (PITR) 7 hari sesuai fitur atau biaya yang berlaku pada paket saat pembelian.

Cocok untuk MVP komersial dan aplikasi produksi kecil sampai menengah. Periksa rincian biaya organisasi, compute, egress, backup, dan tambahan penggunaan karena total tagihan dapat melebihi harga dasar.

### Team — mulai US$599 per bulan

Cocok untuk organisasi yang memerlukan kontrol tim, kepatuhan, dan dukungan lebih kuat. Paket ini umumnya terlalu mahal untuk MVP kecil.

### Self-hosted Supabase

Supabase dapat dijalankan sendiri melalui Docker atau Coolify. Perangkat lunaknya gratis dan biaya utama berasal dari VPS, object storage, backup, email, monitoring, serta waktu operasional.

Pilih self-hosted hanya jika tim siap menangani:

- update dan patch keamanan;
- backup PostgreSQL dan uji restore;
- SMTP untuk email autentikasi;
- kapasitas disk, RAM, dan koneksi;
- monitoring, log, serta respons insiden.

Self-hosted mengurangi ketergantungan pada layanan managed, tetapi memindahkan seluruh tanggung jawab operasional ke pemilik server.

## 3. Alternatif PostgreSQL Managed

### Neon

Paket Free menjadi alternatif bila aplikasi hanya membutuhkan PostgreSQL, bukan Auth dan Storage Supabase.

- Hingga 100 proyek sesuai batas paket yang berlaku.
- Compute dapat scale-to-zero saat tidak dipakai.
- Storage sekitar 0,5 GB pada paket gratis.
- Cocok untuk preview database, eksperimen, dan aplikasi kecil.

Jika memakai Neon, sediakan layanan auth dan object storage secara terpisah.

### Railway

Railway menawarkan PostgreSQL dengan tagihan berbasis pemakaian. Pengalaman deploy sederhana, tetapi biaya mengikuti resource aktif, volume, dan trafik. Tetapkan spending limit atau notifikasi biaya bila tersedia.

### Kapan Tetap Memilih Supabase

Tetap gunakan Supabase bila aplikasi membutuhkan Auth, Storage, RLS, dan Realtime dalam satu layanan. Memisahkan database hanya masuk akal bila ada kebutuhan biaya, region, performa, atau arsitektur yang jelas.

## 4. Domain dan DNS: Cloudflare

Gunakan Cloudflare sebagai registrar bila domain tersedia atau sebagai pengelola DNS.

1. Beli atau tambahkan domain ke Cloudflare.
2. Tambahkan custom domain di Vercel.
3. Buat record DNS sesuai nilai yang diberikan Vercel.
4. Tunggu status domain Vercel menjadi valid.
5. Pastikan HTTPS aktif.
6. Perbarui `Site URL` dan daftar redirect URL pada Supabase Auth.

Untuk konfigurasi lengkap, ikuti [`06-domain-ssl.md`](06-domain-ssl.md).

## Tabel Perbandingan

### Kombinasi Paket

| Skenario | Frontend | Backend/DB | Perkiraan biaya dasar | Cocok untuk |
|---|---|---|---:|---|
| Belajar | Vercel Hobby | Supabase Free | US$0 | Latihan dan demo |
| MVP pribadi | Vercel Hobby* | Supabase Free | US$0 | Prototipe nonkomersial |
| MVP komersial | Vercel Pro | Supabase Pro | Mulai US$45/bulan | Produk dengan pengguna nyata |
| DB terpisah | Vercel Pro | Neon atau Railway | Berdasarkan paket/pemakaian | Kebutuhan PostgreSQL tanpa seluruh fitur Supabase |
| Self-hosted | Vercel atau VPS | Supabase di VPS | Biaya VPS + operasional | Tim yang mampu merawat server |

\* Vercel Hobby hanya untuk penggunaan yang diizinkan oleh ketentuan paket Hobby.

### Supabase Managed vs Self-hosted

| Aspek | Supabase Managed | Supabase Self-hosted |
|---|---|---|
| Setup | Cepat | Lebih rumit |
| Update platform | Dikelola Supabase | Dikelola sendiri |
| Backup | Tergantung paket | Harus dibuat sendiri |
| Kontrol server | Terbatas | Penuh |
| Beban operasional | Rendah | Tinggi |
| Biaya awal | Gratis atau mulai US$25 | Mulai dari biaya VPS |
| Cocok | MVP dan tim kecil | Tim berpengalaman atau kebutuhan khusus |

## Rekomendasi Praktis

- **Belajar atau demo:** Vercel Hobby + Supabase Free.
- **MVP komersial:** Vercel Pro + Supabase Pro.
- **Aplikasi belum memakai Auth/Storage:** bandingkan Supabase dengan Neon.
- **Perlu kontrol penuh atau penempatan data khusus:** pertimbangkan self-hosted setelah backup dan monitoring siap.
- **Trafik mulai tumbuh:** ukur database, egress, storage, MAU, dan function usage sebelum menaikkan paket.

## Peringatan Biaya dan Operasional

- Harga dasar bukan total tagihan. Egress, compute, storage, image optimization, function execution, dan anggota tim dapat menambah biaya.
- Free tier dapat berubah, dijeda, atau dibatasi. Jangan menjanjikan layanan produksi berdasarkan kuota gratis.
- Jangan menaruh `SUPABASE_SERVICE_ROLE_KEY` di variable `NEXT_PUBLIC_*`, browser, repository, atau log.
- Aktifkan RLS pada semua tabel yang dapat diakses client. RLS mati dapat membuka data antarpengguna.
- Pisahkan proyek staging dan production setelah aplikasi menyimpan data pengguna nyata.
- Backup harus berada di lokasi berbeda dan wajib diuji dengan proses restore.
- Pilih region sebelum produksi. Pemindahan region setelah data tumbuh memerlukan migrasi.

## Link Resmi

- [Harga Vercel](https://vercel.com/pricing)
- [Dokumentasi batas penggunaan Vercel](https://vercel.com/docs/limits)
- [Harga Supabase](https://supabase.com/pricing)
- [Dokumentasi platform Supabase](https://supabase.com/docs/guides/platform)
- [Panduan self-hosting Supabase](https://supabase.com/docs/guides/self-hosting)
- [Harga Neon](https://neon.com/pricing)
- [Harga Railway](https://railway.com/pricing)
- [Cloudflare DNS](https://developers.cloudflare.com/dns/)

## Checklist

- [ ] Penggunaan komersial atau nonkomersial sudah ditentukan.
- [ ] Paket Vercel sesuai dengan ketentuan penggunaan proyek.
- [ ] Paket Supabase dipilih berdasarkan DB, storage, MAU, egress, dan backup.
- [ ] Region Vercel dan Supabase dipilih sedekat mungkin.
- [ ] Anggaran bulanan dan batas pemakaian terdokumentasi.
- [ ] Notifikasi penggunaan atau biaya diaktifkan bila tersedia.
- [ ] Domain dikelola melalui Cloudflare dan terhubung ke Vercel.
- [ ] HTTPS aktif dan redirect Supabase Auth memakai domain production.
- [ ] RLS aktif serta diuji untuk pengguna berbeda.
- [ ] Secret hanya tersimpan di dashboard deployment atau secret manager.
- [ ] Proyek staging dan production dipisahkan bila sudah ada data nyata.
- [ ] Backup tersedia dan satu restore uji berhasil.
- [ ] Harga serta kuota diperiksa ulang di link resmi sebelum membeli.
