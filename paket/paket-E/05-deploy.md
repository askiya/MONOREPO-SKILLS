# 05 — Deploy: VPS + Coolify

## Tujuan

Aplikasi berjalan di VPS melalui Coolify, terhubung ke PostgreSQL yang dikelola Coolify, dan melakukan deploy otomatis setiap kali kamu push ke GitHub.

## Prasyarat

- Bab `04-preview.md` lulus (lint, test, build semuanya hijau).
- Kode sudah ada di repositori GitHub.
- Kartu pembayaran atau saldo untuk membeli VPS.

> Rujukan utama: [`../../docs/10-deploy-vps/02-coolify.md`](../../docs/10-deploy-vps/02-coolify.md)
> Hardening VPS: [`../../docs/10-deploy-vps/01-setup-vps.md`](../../docs/10-deploy-vps/01-setup-vps.md)

## 1. Beli VPS

| Provider | Catatan |
|---|---|
| Hetzner | Harga per resource paling ringan, region Eropa dan AS |
| DigitalOcean | Dokumentasi paling banyak, antarmuka ramah pemula |
| Tencent Cloud | Region Asia (Singapura, Jakarta), latensi baik untuk pengguna Indonesia |
| Vultr, Linode | Alternatif dengan region beragam |
| Biznet, IDCloudHost | Server Indonesia, latensi terendah untuk pasar lokal |

Spesifikasi minimum realistis untuk Coolify + satu app Next.js + PostgreSQL:

- 2 vCPU
- 4 GB RAM
- 40–60 GB SSD
- Ubuntu LTS

> Coolify sendiri memakai RAM cukup besar. VPS 1 GB atau 2 GB RAM akan gagal saat build Next.js karena kehabisan memori. Jangan berhemat di titik ini.

Harga berubah cepat. Cek langsung di situs provider; jangan percaya angka di tutorial mana pun.

## 2. Amankan VPS Sebelum Instal Apa Pun

Bagian ini mengubah cara kamu mengakses server. Jika langkahnya dibalik, kamu bisa terkunci dari VPS sendiri dan harus membangun ulang dari nol. Kerjakan berurutan, dan jangan menutup sesi SSH pertama sampai sesi kedua terbukti berhasil.

**Langkah 1.** Login sebagai root menggunakan IP yang diberikan provider.

```bash
ssh root@IP_SERVER
```

**Langkah 2.** Perbarui seluruh paket sistem.

```bash
apt update && apt upgrade -y
```

**Langkah 3.** Buat user non-root dan beri hak sudo.

```bash
adduser deploy
usermod -aG sudo deploy
```

**Langkah 4.** Dari komputer lokal, buat SSH key dan kirim ke server.

```bash
ssh-keygen -t ed25519 -C "deploy@project"
ssh-copy-id deploy@IP_SERVER
```

**Langkah 5.** Buka terminal baru — jangan tutup terminal pertama — lalu buktikan login dengan key berhasil.

```bash
ssh deploy@IP_SERVER
```

Jangan melanjutkan ke langkah 6 sebelum login ini berhasil tanpa diminta password. Jika masih diminta password, perbaiki dulu penempatan key.

**Langkah 6.** Setelah login key terbukti bekerja, matikan login root dan login password. Edit `/etc/ssh/sshd_config` dan pastikan dua baris berikut bernilai `no`:

```
PermitRootLogin no
PasswordAuthentication no
```

Muat ulang layanan SSH:

```bash
sudo systemctl restart ssh
```

Verifikasi kembali dari terminal ketiga bahwa `ssh deploy@IP_SERVER` masih berhasil. Selama verifikasi ini belum lulus, biarkan sesi lama tetap terbuka sebagai jalur penyelamat.

**Langkah 7.** Izinkan port SSH terlebih dahulu, baru aktifkan firewall. Mengaktifkan UFW sebelum mengizinkan SSH akan langsung memutus koneksi kamu.

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

**Langkah 8.** Pasang Fail2ban untuk memblokir percobaan login brute force.

```bash
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
```

**Langkah 9.** Tambahkan swap agar build tidak gagal saat RAM puncak.

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## 3. Instal Coolify

Buka dokumentasi resmi Coolify dan jalankan installer versi terkini. Perintah installer dan port panel dapat berubah antar versi, jadi ambil perintahnya dari dokumentasi resmi, bukan dari catatan lama.

Setelah installer selesai, akses panel pada URL dan port yang ditampilkan installer, lalu **segera buat akun admin pertama**. Panel Coolify yang belum punya admin dapat diklaim oleh siapa pun yang menemukan IP kamu.

Amankan panel:

- gunakan subdomain khusus, misalnya `panel.domain-kamu.com`, dan aktifkan HTTPS untuk panel;
- gunakan password panjang dan unik;
- aktifkan 2FA bila versi kamu mendukungnya;
- batasi akses panel dari IP tertentu bila memungkinkan.

## 4. Hubungkan GitHub

1. Panel Coolify → **Sources** → tambah **GitHub App**.
2. Ikuti proses otorisasi ke akun atau organisasi GitHub kamu.
3. Saat memilih repository access, pilih **Only select repositories** dan beri akses hanya ke repo project ini. Jangan berikan akses ke seluruh akun.
4. Selesaikan instalasi App dan pastikan status source menjadi terhubung.

## 5. Buat PostgreSQL di Coolify

Buat database sebelum aplikasi, supaya credential-nya sudah siap saat mengisi env aplikasi.

1. Buka project di Coolify → **Add Resource** → **Database** → **PostgreSQL**.
2. Pilih versi PostgreSQL yang sama dengan yang kamu pakai lokal (misalnya 16) agar perilaku migrasi konsisten.
3. Deploy resource database.
4. Setelah berjalan, buka tab credential dan catat host internal, port, username, password, dan nama database.
5. Jangan mengaktifkan akses publik untuk database. Aplikasi dan database berada di server yang sama, jadi cukup lewat jaringan internal Coolify.

Connection string yang akan dipakai aplikasi berbentuk:

```
postgresql://USERNAME:PASSWORD@HOST_INTERNAL:5432/NAMA_DATABASE
```

Ambil setiap nilai dari panel Coolify. Jangan menuliskan credential ini ke file mana pun di repo.

## 6. Tambah Resource Aplikasi dari GitHub

1. Project → **Add Resource** → **Application**.
2. Pilih source GitHub yang tadi dihubungkan.
3. Pilih repository dan branch. Gunakan `main` untuk produksi. Jika ingin staging, buat resource kedua dengan branch `develop` dan database terpisah.
4. Build Pack: pilih **Nixpacks** untuk app Node standar, atau **Dockerfile** bila repo kamu menyertakan Dockerfile sendiri.
5. Port aplikasi: `3000`.
6. Jangan tekan Deploy dulu. Isi environment variables terlebih dahulu, karena build Next.js membutuhkan sebagian env pada waktu build.

## 7. Isi Environment Variables di Coolify

Buka tab **Environment Variables** pada resource aplikasi dan tambahkan satu per satu:

| Key | Value | Catatan |
|---|---|---|
| `DATABASE_URL` | connection string dari langkah 5 | wajib |
| `NODE_ENV` | `production` | wajib |
| `NEXTAUTH_SECRET` | `<ISI_SENDIRI>` | bila memakai autentikasi |
| `NEXTAUTH_URL` | `https://domain-kamu.com` | bila memakai autentikasi |
| `NEXT_PUBLIC_APP_URL` | `https://domain-kamu.com` | bila dipakai di client |

Aturan penting:

- Variable berawalan `NEXT_PUBLIC_` ikut terbundel ke JavaScript browser dan dapat dibaca siapa pun. Jangan pernah menaruh secret di sana.
- Variable yang dipakai saat build harus ditandai sebagai build-time variable bila panel kamu memisahkan build dan runtime. Jika tidak, build bisa lulus tetapi aplikasi gagal saat start.
- Secret hanya hidup di panel Coolify. Tidak di repo, tidak di chat, tidak di screenshot.

Untuk membangkitkan secret acak:

```bash
openssl rand -base64 32
```

## 8. Deploy Pertama

Tekan **Deploy** dan baca log build sampai selesai. Jangan menutup halaman log; di situlah semua informasi kegagalan berada.

**Hasil yang diharapkan:** log menunjukkan install dependency sukses, `next build` sukses, container start, dan status resource menjadi running. Setelah itu URL sementara dari Coolify dapat dibuka.

Verifikasi dari komputer lokal:

```bash
curl -s https://URL_SEMENTARA_COOLIFY/api/health
```

**Hasil yang diharapkan:**

```json
{"status":"ok"}
```

Jika mengembalikan `{"status":"error"}` dengan HTTP 503, aplikasi hidup tetapi `DATABASE_URL` salah atau database belum siap.

## 9. Jalankan Migrasi Produksi

Migrasi produksi harus memakai `prisma migrate deploy`, bukan `prisma migrate dev` dan bukan `prisma db push`. Dua perintah terakhir dapat mengubah atau menghapus skema tanpa riwayat migrasi.

Cara paling aman: tambahkan migrasi ke tahap start aplikasi, misalnya pada script start:

```json
{
  "scripts": {
    "start": "prisma migrate deploy && node server.js"
  }
}
```

Alternatifnya, jalankan sekali lewat terminal resource di panel Coolify:

```bash
npx prisma migrate deploy
```

Sebelum menjalankan migrasi yang mengubah atau menghapus kolom pada database yang sudah berisi data, ambil backup terlebih dahulu (lihat `07-maintenance.md`). Migrasi yang menghapus kolom tidak dapat dibatalkan tanpa backup.

## 10. Aktifkan Auto Deploy

1. Buka resource aplikasi → pengaturan **Webhooks** atau **Auto Deploy**.
2. Aktifkan auto-deploy untuk branch yang dipilih.
3. Uji dengan membuat commit kecil dan push ke `main`.

```bash
git commit --allow-empty -m "test auto deploy"
git push origin main
```

**Hasil yang diharapkan:** Coolify memulai build baru secara otomatis dalam beberapa detik tanpa kamu menekan tombol apa pun.

## Membaca Kegagalan Deploy

Baca log, jangan menebak. Pola kegagalan dan artinya:

| Log berhenti di | Artinya | Tindakan |
|---|---|---|
| install dependency | lockfile, registry, atau versi Node tidak cocok | commit lockfile, set versi Node di `package.json` engines |
| `next build` | error TypeScript, import, atau env build-time hilang | jalankan `npm run build` lokal, tambahkan env build-time |
| build sukses, container restart berulang | env runtime kurang, port salah, atau crash saat start | cek `DATABASE_URL`, pastikan port `3000` |
| container jalan, URL tidak bisa diakses | domain/proxy belum benar | lanjut ke `06-domain-ssl.md` |
| OOM / proses terbunuh saat build | RAM VPS tidak cukup | tambah swap atau naikkan ukuran VPS |

Jangan mengganti secret atau memaksa push berulang tanpa bukti dari log. Setiap deploy gagal yang tidak dibaca lognya akan terulang.

## Checklist

- [ ] VPS Ubuntu LTS dengan minimum 2 vCPU / 4 GB RAM aktif.
- [ ] User non-root dengan SSH key terbukti bisa login.
- [ ] Login root dan password dimatikan **setelah** key diverifikasi.
- [ ] UFW aktif dengan OpenSSH, 80, dan 443 diizinkan.
- [ ] Fail2ban aktif dan swap tersedia.
- [ ] Coolify terpasang, punya akun admin, dan panel diamankan.
- [ ] GitHub source terhubung dengan akses hanya ke repo yang diperlukan.
- [ ] PostgreSQL berjalan di Coolify dan tidak terbuka ke publik.
- [ ] Environment variables terisi di panel, bukan di repo.
- [ ] Deploy pertama sukses dan health endpoint mengembalikan `ok`.
- [ ] Migrasi produksi dijalankan dengan `prisma migrate deploy`.
- [ ] Auto deploy terbukti berjalan dari push ke `main`.
