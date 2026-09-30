# Latihan Praktik: Sesi 1 — Setup Environment

## Target Praktik
Komputer siap coding: Node.js, Git, GitHub, AI Agent terinstal.

## Estimasi Waktu
60 menit.

## Yang Harus Disiapkan
- laptop dengan internet,
- akun email,
- kemauan membaca error.

## Langkah

### 1. Instal Node.js

1. Buka https://nodejs.org — unduh versi LTS (bukan Current).
2. Instal (Windows: centang "Add to PATH").
3. Buka terminal (Command Prompt / Terminal):
   ```bash
   node --version
   npm --version
   ```

**Hasil yang benar:**
```text
v20.x.x   (atau LTS terbaru)
10.x.x
```

**Gagal kalau:** `'node' is not recognized` → Node tidak masuk PATH. Instal ulang,
centang "Add to PATH".

### 2. Instal Git

1. Buka https://git-scm.com — unduh installer.
2. Instal dengan default.
3. Tes:
   ```bash
   git --version
   ```

**Hasil yang benar:**
```text
git version 2.x.x
```

### 3. Buat Akun GitHub

1. Buka https://github.com — Sign Up.
2. Verifikasi email.
3. Buat repo baru "latihan-pertama" (private).
4. Clone ke lokal:
   ```bash
   git clone https://github.com/USERNAMEMU/latihan-pertama.git
   cd latihan-pertama
   ```
5. Buat file test:
   ```bash
   echo "# Latihan Pertama" > README.md
   git add README.md
   git commit -m "docs: first commit"
   git push
   ```

**Hasil yang benar:** refresh repo di browser → README.md tampil.

**Gagal kalau:** "Authentication failed" → buat Personal Access Token:
Settings → Developer settings → Personal access tokens → Tokens (classic).

### 4. Instal AI Agent

1. Baca `docs/01-setup-ai-agent/01-antigravity.md` (atau agent lain).
2. Instal sesuai panduan.
3. Buka agent, arahkan ke folder `latihan-pertama`.
4. Kirim prompt:
   ```text
   Buat file hello.js yang isinya console.log("Halo Santriverse")
   ```
5. Jalankan:
   ```bash
   node hello.js
   ```

**Hasil yang benar:**
```text
Halo Santriverse
```

## Cara Verifikasi
- [ ] `node --version` → versi LTS
- [ ] `git --version` → versi 2.x
- [ ] repo GitHub ada + README.md terlihat
- [ ] `hello.js` dibuat AI agent + jalan di terminal

## Error yang Sering Terjadi

| Gejala | Penyebab | Solusi |
|---|---|---|
| `'node' is not recognized` | PATH | Instal ulang, centang PATH |
| `git: command not found` | belum instal / PATH | Instal ulang Git |
| `Authentication failed` | password / token | Buat Personal Access Token |
| Agent tidak mengerti | folder salah | Arahkan agent ke folder project |

## Tugas Mandiri

Buat repo baru "bio-saya" di GitHub. Buat `index.html` lewat AI agent yang berisi:
- nama,
- asal kota,
- kenapa ikut kelas ini.

Push ke GitHub. Buka tab "Code" di GitHub. HTML harus tampil.

## Bukti Kelulusan

Kirim ke mentor:
1. screenshot terminal `node --version` + `git --version`,
2. link repo GitHub "latihan-pertama" (undang mentor kalau private),
3. link repo "bio-saya",
4. screenshot agent berhasil membuat file.
