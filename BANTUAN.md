# Cara Meminta Bantuan

Kalau buntu, kirim pertanyaan dengan format ini. Tanpa format ini, mentor
harus menebak dan jawaban akan lambat.

## Format Wajib

```text
OS         : [Windows 11 / macOS / Ubuntu ...]
Agent      : [Antigravity / Cursor / Claude Code / ...]
Bab/file   : [docs/04-build-frontend/01-...]
Task       : [T-021]

TUJUAN     : [apa yang sedang dikerjakan]

COMMAND    : [perintah yang dijalankan]

OUTPUT ERROR LENGKAP:
[tempel seluruh output, sensor password/API key]

EXPECTED   : [apa yang harusnya terjadi]
ACTUAL     : [apa yang terjadi]

SCREENSHOT : [lampirkan kalau ada]
LINK REPO  : [URL GitHub commit/branch]

SUDAH DICOBA:
1. [langkah yang sudah dicoba]
2. [langkah lain]
```

## Aturan

1. **Jangan** kirim password, API key, token, atau secret. Sensor jadi `<HIDDEN>`.
2. **Jangan** hanya bilang "error bang" atau "gabisa". Tempel output.
3. **Jangan** screenshot error yang terpotong. Kalau bisa, tempel teks, bukan
   gambar — teks bisa dicari, gambar tidak.
4. **Jangan** kirim seluruh kode; hanya file + baris yang relevan.
5. **Jangan** minta solusi tanpa tempel error. Mentor bukan cenayang.
6. **Do** cek GLOSSARY.md kalau tidak paham istilah.
7. **Do** cek `docs/15-troubleshooting/` sebelum bertanya.
8. **Do** sebut nomor bab dan task — mentor langsung tahu konteks.

## Contoh Buruk

```text
bang error nih gabisa build, tolong benerin dong
```

Mentor harus menebak: OS apa, agent apa, error apa, sudah coba apa? Lama.

## Contoh Baik

```text
OS         : Windows 11
Agent      : Antigravity
Bab/file   : docs/06-database/01-postgresql-prisma.md
Task       : T-002

TUJUAN     : koneksi Prisma ke Neon PostgreSQL

COMMAND    : npx prisma migrate dev --name init

OUTPUT ERROR LENGKAP:
Error: P1001: Can't reach database server at `ep-cool-branch-123.us-east-2.aws.neon.tech:5432`
Please make sure your database server is running at `ep-cool-branch-123.us-east-2.aws.neon.tech:5432`.

EXPECTED   : migrasi berhasil, tabel terbuat
ACTUAL     : error koneksi

LINK REPO  : https://github.com/saya/project/commit/abc123

SUDAH DICOBA:
1. Cek DATABASE_URL di .env — sudah diisi, tanpa spasi
2. Buka Neon dashboard — database aktif
3. Coba ping host — timeout
4. Ganti ?sslmode=require → masih gagal
```

Mentor langsung bisa fokus: kemungkinan IP/firewall/SSL, bukan nebak semua.

## Tempat Bertanya

1. **GitHub Issues** repo kelas (kalau diaktifkan mentor)
   - pakai issue template yang tersedia
   - pertanyaan lama bisa dicari
2. **Grup kelas** (WhatsApp/Discord/dll)
   - format tetap sama
   - jangan kirim voice note panjang untuk error teknis
3. **Langsung ke mentor** — gunakan sebagai opsi terakhir

## Checklist Sebelum Kirim Pertanyaan

- [ ] Saya sudah cek GLOSSARY.md untuk istilah yang belum paham
- [ ] Saya sudah cek troubleshooting docs/15
- [ ] Saya sudah tempel output error **lengkap**
- [ ] Saya sudah sensor secret/password
- [ ] Saya menyebut nomor bab dan task
