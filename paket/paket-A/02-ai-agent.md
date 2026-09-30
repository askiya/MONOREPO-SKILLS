# 02 — Setup AI Agent: Antigravity

## Tujuan

Antigravity terpasang, terhubung ke folder project, dan sudah diberi konteks
6 dokumen perencanaan. Agent siap menerima perintah build.

## Sebelum Mulai

- 6 dokumen perencanaan sudah selesai (lihat [`01-pedoman.md`](01-pedoman.md))
- Node.js LTS terinstal (`node -v`)
- Git terinstal (`git --version`)

---

## Langkah-Langkah

### 1. Instal Antigravity

Ikuti panduan lengkap di:
[`../../docs/01-setup-ai-agent/01-antigravity.md`](../../docs/01-setup-ai-agent/01-antigravity.md)

Ringkasan:
1. Buka website resmi Antigravity
2. Download installer sesuai OS
3. Instal dengan opsi standar
4. Buka aplikasi, login atau hubungkan provider model AI

> ⚠️ UI dan langkah instalasi bisa berubah. Selalu cocokkan dengan dokumentasi
> resmi Antigravity. Jangan unduh dari link tidak resmi.

### 2. Buka folder project

1. Di Antigravity, pilih **Open Folder**
2. Arahkan ke folder project yang sudah berisi 6 dokumen
3. **Jangan** buka seluruh drive — arahkan ke folder project saja

Verifikasi: Antigravity menampilkan file tree project kamu di sidebar.

### 3. Beri konteks ke agent

Agent perlu membaca 6 dokumen perencanaan supaya tahu apa yang harus dibangun.

Kirim pesan ini ke Antigravity:

```
Baca semua dokumen berikut dan pahami sebagai konteks project:

1. PRD.md — requirements dan fitur
2. SDLC.md — alur kerja development
3. DESIGN.md — desain visual
4. ARCHITECTURE.md — stack dan struktur folder
5. TASKS.md — daftar task yang harus dikerjakan
6. AGENTS.md — aturan yang harus kamu ikuti

Setelah baca, konfirmasi: sebutkan nama project, stack yang dipakai, dan
3 fitur utama MVP.
```

**Expected output:** Agent membalas dengan nama project, stack (Next.js +
Prisma + Neon + NextAuth + Vercel), dan 3 fitur dari PRD.

Kalau agent menyebut stack yang berbeda → cek ARCHITECTURE.md, pastikan isinya
sesuai Paket A.

### 4. Tes kemampuan dasar agent

Kirim perintah sederhana untuk memastikan agent bisa bekerja:

```
Buatkan file README.md untuk project ini berdasarkan PRD.md.
```

Verifikasi:
- Agent membuat file `README.md`
- Isinya relevan dengan PRD
- File muncul di folder project

---

## Cara Memberi Konteks (Detail)

Untuk panduan lengkap cara memberi konteks ke agent, baca:
[`../../docs/01-setup-ai-agent/03-cara-memberi-konteks.md`](../../docs/01-setup-ai-agent/03-cara-memberi-konteks.md)

Tips utama:
- Masukkan semua 6 dokumen **sebelum** mulai build
- Kalau agent "lupa", kirim ulang dokumen yang relevan
- Jangan beri instruksi yang bertentangan dengan AGENTS.md

---

## Alternatif Agent

Antigravity adalah agent default untuk pemula karena GUI-nya mudah. Kalau
kamu lebih nyaman dengan CLI, lihat alternatif di:
[`../../docs/01-setup-ai-agent/02-alternatif-ai-agent.md`](../../docs/01-setup-ai-agent/02-alternatif-ai-agent.md)

---

## Kesalahan Umum

| Kesalahan | Akibat | Solusi |
|---|---|---|
| Tidak beri konteks 6 dokumen | Agent bikin project dari imajinasi sendiri | Kirim semua dokumen sebelum build |
| Buka seluruh drive di Antigravity | Agent bingung, baca file yang salah | Buka folder project saja |
| Pakai agent tanpa AGENTS.md | Agent install package sembarangan | Tulis aturan di AGENTS.md |
| Langsung suruh build tanpa tes | Tidak tahu agent bisa kerja atau belum | Tes buat README dulu |

---

## Checklist

- [ ] Antigravity terinstal dan bisa dibuka
- [ ] Provider model AI terhubung
- [ ] Folder project dibuka di Antigravity
- [ ] 6 dokumen perencanaan sudah dibaca oleh agent
- [ ] Agent bisa menyebutkan nama project, stack, dan 3 fitur MVP
- [ ] Tes buat README.md berhasil
