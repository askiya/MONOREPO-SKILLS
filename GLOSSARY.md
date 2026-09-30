# Kamus Istilah Pemula

Kalau mentor/AI agent bilang kata yang tidak kamu paham, cari di sini dulu.

| Istilah | Arti sederhana | Contoh |
|---|---|---|
| **Frontend** | Bagian aplikasi yang dilihat dan diklik pengguna | halaman login, tombol, kartu produk |
| **Backend** | Bagian server yang memproses data dan aturan bisnis | cek password, simpan pesanan |
| **API** | Jalur komunikasi antar-aplikasi/frontend-backend | frontend meminta daftar produk lewat API |
| **Endpoint** | Satu alamat + method spesifik dalam API | `GET /api/products` |
| **HTTP Method** | Jenis tindakan request | GET baca, POST buat, PATCH ubah, DELETE hapus |
| **Status Code** | Angka hasil request | 200 sukses, 404 tidak ditemukan, 500 error server |
| **Database (DB)** | Tempat data tersimpan terstruktur | tabel users, products, orders |
| **PostgreSQL/MySQL** | Jenis database relasional | PostgreSQL dipakai project kelas |
| **ORM** | Alat mengakses DB lewat kode tanpa banyak SQL mentah | Prisma |
| **Schema** | Bentuk/aturan data di DB | Product punya title, price, status |
| **Migration** | Catatan perubahan struktur DB | menambah kolom `paidAt` |
| **Seed** | Data contoh yang dimasukkan otomatis | 6 produk dummy untuk dev |
| **Environment Variable** | Nilai konfigurasi dari luar kode | `DATABASE_URL`, `NEXTAUTH_SECRET` |
| **Credential** | Informasi untuk membuktikan akses | username, password, API key |
| **Secret** | Credential rahasia yang tidak boleh masuk Git/browser | DB password, signing key |
| **`.env`** | File lokal tempat environment variable rahasia | `.env` masuk `.gitignore` |
| **Localhost** | Komputer sendiri sebagai server sementara | `http://localhost:3000` |
| **Port** | Nomor pintu layanan di komputer/server | Next.js default port 3000 |
| **Domain** | Nama mudah dibaca untuk alamat website | `santriverse.com` |
| **DNS** | Sistem yang mengarahkan domain ke server | domain diarahkan ke IP VPS |
| **Nameserver** | Server yang memegang catatan DNS domain | nameserver Cloudflare |
| **A Record** | DNS record domain/subdomain menuju alamat IPv4 | `api` → `43.159.x.x` |
| **CNAME** | DNS record alias menuju nama domain lain | `www` → `domain.com` |
| **MX Record** | DNS record tujuan email domain | email masuk menuju Google Workspace |
| **SSL/TLS** | Enkripsi koneksi website | gembok HTTPS di browser |
| **HTTPS** | HTTP yang dienkripsi SSL/TLS | `https://domain.com` |
| **Hosting** | Komputer/layanan tempat website berjalan online | cPanel, Vercel, VPS |
| **Shared Hosting** | Satu server dipakai banyak pelanggan | paket hosting cPanel murah |
| **VPS** | Server virtual yang kamu kelola lebih bebas | instal Docker + Coolify sendiri |
| **cPanel** | Panel visual untuk kelola shared hosting | upload file, DB, SSL, email |
| **Cloudflare** | Layanan DNS, CDN, proxy, dan security | mengelola record A/CNAME |
| **CDN** | Jaringan server penyaji asset dari lokasi dekat user | gambar/CSS lebih cepat |
| **Build** | Proses mengubah source code menjadi versi siap jalan | `npm run build` menghasilkan `.next` |
| **Deploy** | Mengirim dan menjalankan app di hosting | push GitHub lalu deploy Vercel |
| **Staging** | Lingkungan online untuk tes sebelum produksi | `staging.domain.com` |
| **Production** | Lingkungan asli yang dipakai pengguna | `domain.com` |
| **Repository (Repo)** | Folder project yang dilacak Git | repo GitHub `MONOREPO-SKILLS` |
| **Git** | Sistem pencatat versi/perubahan kode | melihat diff dan rollback |
| **Commit** | Satu checkpoint perubahan | `feat(auth): add login` |
| **Push** | Mengirim commit lokal ke remote/GitHub | `git push origin main` |
| **Pull** | Mengambil perubahan remote | `git pull` |
| **Branch** | Jalur pengembangan terpisah | `main`, `develop`, `feature/login` |
| **Merge** | Menggabungkan branch/perubahan | feature digabung ke main |
| **Pull Request (PR)** | Permintaan review sebelum merge | PR fitur login |
| **Diff** | Perbandingan isi file sebelum/sesudah | baris hijau ditambah, merah dihapus |
| **Dependency** | Package/library yang dipakai project | React, Prisma |
| **Package Manager** | Alat install dependency | npm, pnpm, yarn |
| **Framework** | Kerangka kerja aplikasi | Next.js, Laravel, FastAPI |
| **Runtime** | Lingkungan yang menjalankan kode | Node.js |
| **Serverless** | Platform menjalankan function tanpa kamu kelola server | Vercel Functions |
| **Container** | Paket app + runtime yang konsisten | Docker container |
| **Docker** | Alat membuat dan menjalankan container | app + PostgreSQL via Compose |
| **Reverse Proxy** | Server depan yang meneruskan request ke app | Nginx ke app port 3000 |
| **Webhook** | Request otomatis dari layanan luar saat event terjadi | Xendit memberi tahu pembayaran sukses |
| **Signature Webhook** | Bukti webhook benar dari provider | HMAC/signing token diverifikasi server |
| **Authentication** | Mengecek siapa pengguna | login email/password |
| **Authorization** | Mengecek apa yang boleh dilakukan | hanya admin boleh CRUD produk |
| **Session** | Catatan login pengguna | cookie session setelah login |
| **Cookie** | Data kecil yang disimpan browser | session ID |
| **Hash** | Hasil transformasi satu arah | password disimpan sebagai bcrypt hash |
| **Encryption** | Data disandikan dan bisa dibuka dengan key | koneksi HTTPS |
| **Backup** | Salinan data untuk pemulihan | dump DB harian |
| **Restore** | Mengembalikan data dari backup | restore dump ke DB uji |
| **Rollback** | Kembali ke versi app sebelumnya | deploy commit lama saat rilis rusak |
| **Log** | Catatan kejadian dari app/server | error runtime, request ID |
| **Monitoring** | Pemantauan kesehatan app | alert saat website mati |
| **CI** | Test/build otomatis saat kode di-push | GitHub Actions menjalankan lint/test |
| **SEO** | Optimasi agar halaman mudah ditemukan mesin pencari | title, sitemap, robots |
| **Responsive** | UI menyesuaikan ukuran layar | grid 1 kolom HP, 3 desktop |
| **PRD** | Dokumen kebutuhan produk | fitur, persona, scope, metrik |
| **SDLC** | Tahapan pengembangan software | planning → build → test → deploy |
| **Tech Stack** | Kumpulan teknologi project | Next.js + PostgreSQL + Vercel |
| **MVP** | Versi minimum yang sudah memberi nilai | login + katalog + checkout, tanpa referral |
| **Scope** | Batas pekerjaan yang disepakati | fitur MVP masuk, referral belum |
