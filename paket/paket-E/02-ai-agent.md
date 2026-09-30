# 02 — Setup AI Agent (Antigravity)

## Tujuan

Antigravity terpasang, konteks proyek termuat, dan agent siap menerima tugas build sesuai AGENTS.md.

## Prasyarat

- Repo aplikasi sudah berisi enam dokumen perencanaan.
- Koneksi internet stabil.

## 1. Pasang Antigravity

Buka [Antigravity](https://antigravity.dev) dan ikuti proses instalasi sesuai platform yang kamu pakai. Antigravity berbasis GUI sehingga cocok untuk level menengah yang belum terbiasa CLI agent.

## 2. Buka Proyek

Buka folder proyek dari panel Antigravity. Pastikan agent bisa membaca seluruh file di root project.

## 3. Muat Konteks

Beri agent akses ke dokumen perencanaan:

```text
Baca file berikut dan jadikan panduan kerja:
- PRD.md
- ARCHITECTURE.md
- TASKS.md
- AGENTS.md
```

Verifikasi agent dapat merangkum scope dan arsitektur dengan benar.

## 4. Atur Batasan

Pastikan `AGENTS.md` sudah berisi:

- agent tidak boleh membaca/mencetak `.env`;
- agent tidak boleh menghapus data atau volume;
- migrasi produksi hanya dengan `prisma migrate deploy` dan harus setelah backup;
- output standalone wajib di `next.config.js`;
- semua tugas wajib menyertakan verifikasi (`npm run lint`, `npm test`, `npm run build`).

## 5. Mulai Tugas Pertama

Berikan tugas pertama dari TASKS.md:

```text
Kerjakan tugas E-01 dari TASKS.md.
Jalankan lint dan test setelah selesai.
Laporkan file yang diubah dan hasil test.
```

**Hasil yang diharapkan:** agent membuat perubahan sesuai scope tugas; lint dan test lulus.

## Alternatif Agent

| Agent | Kapan pakai |
|---|---|
| Antigravity | Default paket ini. GUI. |
| Cursor | Sudah berlangganan Cursor dan lebih nyaman IDE. |
| Hermes Agent | Suka CLI dan butuh orkestrasi multi-langkah. |
| Claude Code | Nyaman terminal dan bisa membaca output mentah. |

Ganti agent kapan saja tanpa mengubah arsitektur; yang berubah hanya cara memberi perintah.

## Kegagalan Umum

| Gejala | Perbaikan |
|---|---|
| Agent mengubah file di luar scope tugas | Perbaiki `AGENTS.md`, batasi direktori |
| Agent mencetak isi `.env` | Tegaskan larangan di AGENTS; jangan masukkan `.env` ke konteks |
| Hasil build rusak setelah tugas agent | Jalankan `npm run build` sebelum commit; rollback bila gagal |
| Agent bingung arsitektur | Pastikan ARCHITECTURE.md terkini |

## Checklist

- [ ] Antigravity terpasang dan bisa membuka proyek.
- [ ] Agent membaca dokumen perencanaan dengan benar.
- [ ] AGENTS.md membatasi secret, data, dan deploy.
- [ ] Tugas pertama selesai, lint dan test lulus.
