# AGENTS.md — Aturan untuk AI Agent di Repo Ini

Repo ini adalah **repo pedoman/dokumentasi**, bukan aplikasi.

## Tujuan

Menyediakan pedoman monorepo untuk kelas vibe coding Santriverse: dari instalasi
AI agent sampai deploy produksi.

## Aturan Menulis

- Bahasa Indonesia, gaya instruktif, langsung ke langkah.
- Semua perintah terminal harus bisa dijalankan apa adanya (copy-paste).
- Setiap dokumen wajib punya: tujuan, prasyarat, langkah bernomor, checklist akhir.
- Jangan tulis secret, API key, token, atau password asli. Pakai placeholder
  `<ISI_SENDIRI>`.
- Nama file dokumen: huruf kecil, pakai tanda hubung, diawali nomor urut.
- Jangan hapus dokumen lama tanpa diminta; tandai `DEPRECATED` di judul.

## Struktur

```
docs/<nomor>-<topik>/<nomor>-<judul>.md
templates/<NAMA>.md
prompts/<fase>.md
skills/<nama-skill>/SKILL.md
checklists/<nama>.md
```

## Definisi Selesai

Sebuah dokumen dianggap selesai kalau orang awam bisa mengikutinya tanpa
bertanya, dan checklist di akhir bisa dicentang semua.
