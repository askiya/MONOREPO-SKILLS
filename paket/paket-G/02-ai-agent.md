# G2 — Setup AI Agent: Cursor

> 🔵 MENENGAH · Target waktu: 45 menit · Prasyarat: [01-pedoman.md](01-pedoman.md) selesai

## Tujuan

Cursor siap bekerja di folder project, membaca semua dokumen, dan mengikuti aturan Railway tanpa menebak.

## Langkah 1 — Instal dan Buka Project

1. Unduh Cursor dari <https://www.cursor.com/> sesuai OS.
2. Instal, lalu pilih **File → Open Folder**.
3. Pilih folder root project, bukan folder `src/`.
4. Buka terminal Cursor dengan **Terminal → New Terminal**.

```bash
pwd
ls
```

**Output yang diharapkan:** lokasi root project dan daftar berisi `AGENTS.md` serta `docs/`.

## Langkah 2 — Lindungi Secret

Pastikan `.gitignore` berisi:

```gitignore
.env
.env.local
.env.*.local
node_modules/
.next/
```

Buat contoh aman:

```bash
printf 'DATABASE_URL="<FROM_RAILWAY>"\nSESSION_SECRET="<ISI_SENDIRI>"\nCRON_ENABLED="false"\n' > .env.example
```

**Output yang diharapkan:** `.env.example` ada, tetapi tidak memuat secret nyata.

## Langkah 3 — Tambah Aturan Cursor

Buat `.cursor/rules/project.mdc`:

```markdown
---
description: Aturan project Railway Fullstack
alwaysApply: true
---

- Baca AGENTS.md dan seluruh docs sebelum mengubah kode.
- Deploy target Railway, bukan Vercel.
- Gunakan Next.js App Router, TypeScript, Prisma, PostgreSQL.
- Jangan baca atau cetak nilai file .env.
- Jangan menambah dependency tanpa alasan.
- Kerjakan satu task TASKS.md per giliran.
- Setelah perubahan, jalankan lint, test terkait, dan build.
- Laporkan file yang berubah, perintah verifikasi, dan hasil nyata.
```

**Output yang diharapkan:** aturan muncul di **Cursor Settings → Rules** dan berstatus aktif.

## Langkah 4 — Uji Pemahaman Agent

Buka Agent (`Ctrl+I` atau `Cmd+I`), pilih model yang tersedia, lalu kirim:

```text
Baca AGENTS.md dan semua file dalam docs/. Jangan ubah file apa pun.
Ringkas: tujuan produk, maksimal 5 fitur MVP, stack, skema data,
alasan memilih Railway, variabel lingkungan, dan tiga aturan yang paling berisiko dilanggar.
Sebutkan konflik atau informasi yang belum jelas.
```

**Output yang diharapkan:** agent menyebut Railway, persistent process, Prisma/PostgreSQL, dan tidak membuat file.

Jika agent keliru, perbaiki dokumen—jangan menambal dengan chat panjang yang akan hilang.

## Langkah 5 — Prompt Implementasi Pertama

```text
Kerjakan hanya T01 dari docs/05-TASKS.md.
Sebelum mulai, baca AGENTS.md dan docs/04-ARCHITECTURE.md.
Jangan kerjakan task lain. Gunakan konfigurasi noninteraktif bila tersedia.
Sesudah selesai, jalankan pemeriksaan yang relevan dan laporkan:
1. file berubah,
2. keputusan penting,
3. output verifikasi,
4. blocker.
```

**Output yang diharapkan:** agent menginisialisasi Next.js sesuai arsitektur, lalu memberi hasil perintah nyata.

## Pola Kerja Aman

| Tahap | Prompt | Hasil wajib |
|---|---|---|
| Rencana | “Jelaskan rencana T05, jangan ubah file.” | Daftar file dan risiko |
| Eksekusi | “Kerjakan hanya T05.” | Diff kecil, fokus satu task |
| Verifikasi | “Jalankan lint, test terkait, build.” | Output perintah nyata |
| Review | “Review diff terhadap PRD dan AGENTS.md.” | Temuan atau status bersih |

## Error Umum

| Gejala | Penyebab | Perbaikan |
|---|---|---|
| Agent mengubah banyak fitur | Prompt terlalu lebar | Batasi satu ID task |
| Agent memilih Vercel | Dokumen tidak terbaca | Buka root project; tegaskan di rule |
| Agent mencetak `.env` | Instruksi keamanan kurang | Larang baca/cetak nilai secret |
| Agent mengaku build sukses tanpa output | Tidak diminta verifikasi | Minta perintah dan output nyata |
| Context penuh | Satu chat terlalu panjang | Mulai chat baru; dokumen tetap jadi sumber kebenaran |

## Checklist

- [ ] Cursor membuka root project
- [ ] `AGENTS.md` dan `docs/` terlihat
- [ ] `.env` diabaikan; `.env.example` hanya placeholder
- [ ] Rule Cursor aktif
- [ ] Agent merangkum stack dan alasan Railway dengan benar
- [ ] Agent tidak mengubah file saat diminta hanya membaca
- [ ] Prompt pertama dibatasi satu task

➡️ Lanjut ke **[03-build.md](03-build.md)**.
