# 02 — Prasyarat Alat

## Tujuan

Menyiapkan laptop supaya semua bab berikutnya jalan tanpa drama.

## Minimal Spesifikasi

- RAM 8 GB (16 GB nyaman)
- Penyimpanan kosong 20 GB
- Koneksi internet stabil (AI agent bolak-balik ke API)
- OS: Windows 10/11, macOS, atau Linux

## Wajib Dipasang

| Alat | Fungsi | Cek versi |
|---|---|---|
| Node.js LTS (20/22) | menjalankan Next.js & backend JS | `node -v` |
| npm (ikut Node) | package manager | `npm -v` |
| Git | versi kode | `git --version` |
| AI Agent (Antigravity) | yang ngoding | lihat `docs/01` |
| Browser Chrome/Edge | preview + DevTools | — |

Opsional tapi sangat berguna:

| Alat | Fungsi |
|---|---|
| Docker Desktop | database lokal & deploy VPS |
| pnpm | package manager lebih cepat |
| DBeaver / pgAdmin | lihat isi database pakai GUI |
| Postman / Thunder Client | tes API |

## Pasang Node.js

### Windows
1. Buka https://nodejs.org
2. Unduh versi **LTS**.
3. Jalankan installer, centang "Add to PATH", Next sampai selesai.
4. Buka terminal **baru**, jalankan:

```bash
node -v
npm -v
```

Kalau keluar angka versi, berhasil.

### macOS
```bash
brew install node@22
node -v
```

### Linux (Ubuntu/Debian)
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
node -v
```

## Pasang Git & Set Identitas

```bash
git --version
git config --global user.name "Nama Kamu"
git config --global user.email "email@kamu.com"
git config --global init.defaultBranch main
```

## Akun yang Perlu Dibuat

1. **GitHub** — wajib. Tempat kode.
2. **Vercel** — deploy gratis frontend.
3. **Cloudflare** — DNS + Pages gratis.
4. **Supabase / Neon** — PostgreSQL gratis untuk latihan.
5. (Nanti) penyedia VPS + Xendit/Lynk.id kalau sudah jualan.

Semua daftar pakai email yang sama supaya tidak bingung.

## Struktur Folder Kerja

Bikin satu folder induk, jangan menyebar di Desktop:

```
~/projects/
  nama-project-1/
  nama-project-2/
```

Windows: `C:\projects\nama-project`. Hindari folder dengan spasi dan karakter
aneh kalau bisa — sebagian tool CLI masih rewel dengan spasi.

## Verifikasi Akhir

Jalankan ini; semua harus keluar versi, bukan error:

```bash
node -v && npm -v && git --version
```

## Checklist

- [ ] `node -v` dan `npm -v` keluar versi
- [ ] `git --version` keluar versi
- [ ] Identitas git sudah diset
- [ ] Akun GitHub aktif
- [ ] Folder kerja sudah dibuat
