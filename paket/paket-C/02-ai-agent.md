# 02 — Setup AI Agent: Antigravity (Paket C)

## Tujuan

Antigravity terpasang, bisa membaca folder project, mengubah file, dan
menjalankan perintah terminal. Agent sudah memahami batasan arsitektur cPanel.

---

## Sebelum Mulai

- [ ] Node.js LTS terinstal (`node -v`)
- [ ] Git terinstal (`git --version`)
- [ ] Folder project dari [01-pedoman.md](01-pedoman.md) sudah ada

---

## Langkah

### 1. Instal Antigravity

Panduan lengkap: [`../../docs/01-setup-ai-agent/01-antigravity.md`](../../docs/01-setup-ai-agent/01-antigravity.md)

Ringkasan:
1. Buka website/dokumentasi resmi Antigravity
2. Download installer sesuai OS
3. Instal dengan opsi standar
4. Login atau hubungkan provider model AI

> UI dan nama menu bisa berubah. Selalu ikuti dokumentasi resmi Antigravity.

### 2. Buka Folder Project

1. Buka Antigravity
2. Klik **Open Folder** → arahkan ke folder project kamu
3. Jangan buka seluruh drive — buka **folder project saja**

### 3. Tes Agent Dasar

Kirim prompt ini ke Antigravity:

```text
Baca folder ini. Buat file hello.txt berisi "Antigravity siap".
Setelah itu baca kembali file tersebut dan laporkan isinya.
```

**Berhasil kalau:** `hello.txt` muncul dan isinya tepat "Antigravity siap".

Tes terminal:

```text
Jalankan node -v dan git --version. Jangan ubah file apa pun.
Laporkan output mentahnya.
```

**Berhasil kalau:** agent menampilkan dua versi tanpa error.

### 4. Berikan Konteks Arsitektur cPanel

Kirim prompt ini agar agent paham batasan Paket C:

```text
Baca file PRD.md, ARCHITECTURE.md, dan AGENTS.md di folder ini.

Aturan penting untuk project ini:
1. Ini project Next.js dengan output: 'export' (static HTML)
2. DILARANG menggunakan getServerSideProps, getStaticProps dengan revalidate, atau middleware Next.js
3. Semua data fetching harus client-side (useEffect, SWR, atau fetch di browser)
4. Target deploy: cPanel shared hosting, bukan Vercel/VPS
5. Backend (kalau ada) menggunakan PHP sederhana atau Node kecil
6. Database: MySQL, bukan PostgreSQL
7. Tidak ada Docker, tidak ada CI/CD otomatis
8. Upload manual via File Manager cPanel

Konfirmasi kamu sudah memahami batasan ini.
```

**Berhasil kalau:** agent merangkum batasan dengan benar.

### 5. Hapus File Tes

```bash
rm hello.txt
```

---

## Cara Memberi Konteks ke Agent

Panduan lengkap: [`../../docs/01-setup-ai-agent/03-cara-memberi-konteks.md`](../../docs/01-setup-ai-agent/03-cara-memberi-konteks.md)

Untuk Paket C, konteks terpenting:
- **ARCHITECTURE.md** — agent harus tahu ini static export
- **AGENTS.md** — batasan apa yang boleh dan tidak boleh
- **DESIGN.md** — agar output sesuai desain yang diinginkan

---

## Alternatif AI Agent

Kalau Antigravity tidak cocok:

| Agent | Catatan |
|---|---|
| Cursor | Lebih kuat untuk editing, tapi perlu langganan |
| Windsurf | Mirip Cursor, alternatif gratis |
| Claude Code / Hermes | CLI-based, untuk yang nyaman terminal |

Detail: [`../../docs/01-setup-ai-agent/02-alternatif-ai-agent.md`](../../docs/01-setup-ai-agent/02-alternatif-ai-agent.md)

---

## Kegagalan Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Agent buat SSR code | Tidak dikasih konteks `output: 'export'` | Kirim ulang prompt langkah 4 |
| Agent tidak bisa jalankan terminal | Antigravity belum diberi izin | Buka settings, aktifkan terminal access |
| Agent buka file di luar project | Folder yang dibuka terlalu luas | Tutup, buka ulang dengan folder project saja |

---

## Checklist

- [ ] Antigravity terinstal dan bisa dibuka
- [ ] Folder project terbuka di Antigravity
- [ ] Agent bisa buat dan baca file
- [ ] Agent bisa jalankan perintah terminal
- [ ] Agent sudah diberi konteks arsitektur cPanel (static export, MySQL, upload manual)
