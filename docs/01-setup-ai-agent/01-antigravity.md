# 01 — Instalasi Antigravity (Agent Utama Kelas)

> Catatan versi: UI dan langkah instalasi Antigravity bisa berubah. Selalu cocokkan
> dengan dokumentasi resmi Antigravity pada saat kelas. Jangan unduh installer
> dari link tidak resmi.

## Tujuan

Antigravity terpasang, bisa membuka folder project, membaca file, mengubah file,
dan menjalankan perintah terminal.

## Sebelum Mulai

Pastikan sudah ada:

- Node.js LTS (`node -v`)
- Git (`git --version`)
- Akun/provider model AI yang didukung Antigravity
- Folder project kosong untuk latihan

## Instalasi

1. Buka website/dokumentasi resmi Antigravity.
2. Pilih installer sesuai OS.
3. Instal dengan opsi standar.
4. Buka aplikasi, login atau hubungkan provider model.
5. Pilih **Open Folder**, arahkan ke folder latihan — jangan buka seluruh drive.

Karena distribusi dan nama menu dapat berubah, sumber resmi menang atas screenshot
materi ini. Jangan ikuti tutorial lama untuk memasukkan API key ke file kode.

## Tes Agent

Buat folder kosong, buka di Antigravity, lalu kirim:

```text
Baca folder ini. Buat file hello.txt berisi "Antigravity siap".
Setelah itu baca kembali file tersebut dan laporkan isinya.
```

Berhasil kalau `hello.txt` muncul dan isinya tepat.

Tes terminal:

```text
Jalankan node -v dan git --version. Jangan ubah file apa pun.
Laporkan output mentahnya.
```

Berhasil kalau agent menampilkan dua versi tanpa error.

## Mode Izin

Gunakan kebijakan ini:

| Aksi | Keputusan |
|---|---|
| Baca file project | boleh otomatis |
| Tulis file dalam project | boleh, tetap baca diff |
| Jalankan lint/test/build | boleh otomatis |
| Install dependency | minta konfirmasi |
| Hapus banyak file | wajib konfirmasi |
| Git commit/push | wajib konfirmasi |
| Deploy produksi | wajib konfirmasi |
| Perintah DB produksi | wajib konfirmasi |

Jangan beri akses satu drive penuh kalau project cuma satu folder.

## Pasang Dokumen Konteks

Copy template berikut ke root project:

- `AGENTS.md`
- `PRD.md`
- `SDLC.md`
- `DESIGN.md`
- `ARCHITECTURE.md`
- `TASKS.md`

Lalu prompt pertama:

```text
Baca AGENTS.md, PRD.md, SDLC.md, DESIGN.md, ARCHITECTURE.md, dan TASKS.md.
Jangan menulis kode. Ringkas tujuan produk, stack, aturan, risiko, dan 5 task
pertama. Tunjukkan konflik antar-dokumen kalau ada.
```

Kalau rangkumannya salah, perbaiki dokumen. Jangan lanjut build.

## Troubleshooting

### Agent tidak bisa melihat file
- Pastikan folder yang benar dibuka.
- Tutup/buka ulang workspace.
- Pastikan OS tidak memblokir izin folder.

### Terminal bilang `node: command not found`
- Tutup Antigravity sepenuhnya setelah install Node.
- Buka lagi supaya PATH dimuat ulang.
- Cek `node -v` di terminal OS.

### Agent mengubah file di luar task
- Klik reject/undo.
- Kecilkan task.
- Tambah larangan spesifik ke `AGENTS.md`.

## Checklist

- [ ] Antigravity terpasang dari sumber resmi
- [ ] Folder project berhasil dibuka
- [ ] Agent bisa membuat dan membaca file
- [ ] Agent bisa menjalankan `node -v` dan `git --version`
- [ ] Mode izin tidak membolehkan deploy/hapus/push otomatis
- [ ] Enam dokumen konteks sudah masuk root project
