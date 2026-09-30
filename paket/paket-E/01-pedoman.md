# 01 — Pedoman Perencanaan Paket E

## Tujuan

Menghasilkan enam dokumen yang menjadi kontrak kerja manusia, AI agent, aplikasi, dan operasi VPS sebelum kode dibuat.

## Prasyarat

- Masalah pengguna dan target bisnis sudah diketahui.
- Repo project tersedia.
- Template ada di [`../../templates/`](../../templates/).

## 1. Salin Dokumen

Dari root project aplikasi:

```bash
cp templates/PRD.md PRD.md
cp templates/SDLC.md SDLC.md
cp templates/DESIGN.md DESIGN.md
cp templates/ARCHITECTURE.md ARCHITECTURE.md
cp templates/TASKS.md TASKS.md
cp templates/AGENTS.md AGENTS.md
```

**Hasil yang diharapkan:** enam file ada di root dan tidak kosong.

```bash
wc -l PRD.md SDLC.md DESIGN.md ARCHITECTURE.md TASKS.md AGENTS.md
```

## 2. Isi PRD

Wajib tulis:

- masalah dan pengguna utama;
- tiga alur terpenting;
- fitur masuk dan tidak masuk versi pertama;
- data yang disimpan;
- kriteria terima terukur;
- risiko privasi, pembayaran, dan kehilangan data.

Contoh kriteria terima: “Pengguna terautentikasi dapat membuat produk, memuat ulang halaman, dan melihat data yang sama dari PostgreSQL.” Hindari “fitur berjalan baik”.

## 3. Tetapkan SDLC

Gunakan tahap berikut:

| Tahap | Bukti lulus |
|---|---|
| Rencana | PRD dan arsitektur disetujui |
| Build | fitur sesuai TASKS |
| Verifikasi | lint, test, build lulus |
| Staging | smoke test dengan database staging |
| Produksi | deploy sehat dan migrasi sukses |
| Operasi | monitoring dan backup aktif |

Tentukan branch produksi (`main`), cara review, rollback, dan siapa yang boleh mengubah env produksi.

## 4. Buat DESIGN

Catat viewport, warna, tipografi, komponen, state loading/kosong/error/sukses, aksesibilitas keyboard, dan responsif. Sertakan rancangan halaman error dan konfirmasi tindakan destruktif.

## 5. Buat ARCHITECTURE

```text
Browser → Cloudflare → Coolify proxy → Next.js standalone
                                      ├── Route Handler / Server Action
                                      └── Prisma → PostgreSQL internal
GitHub → webhook → Coolify build/deploy
```

Catat keputusan:

- Next.js satu service, port `3000`;
- `output: "standalone"`;
- PostgreSQL resource terpisah di Coolify;
- `DATABASE_URL` hanya lewat env;
- migrasi produksi memakai `prisma migrate deploy`;
- backup database keluar VPS;
- health endpoint tidak membuka secret.

## 6. Pecah TASKS

Setiap tugas maksimal satu hasil yang dapat diuji. Urutan minimum:

1. fondasi Next.js dan quality gate;
2. skema Prisma dan migrasi;
3. autentikasi/otorisasi bila diperlukan;
4. fitur inti per alur PRD;
5. state UI dan validasi input;
6. test;
7. health endpoint;
8. konfigurasi standalone;
9. deploy staging;
10. backup, domain, produksi.

Format:

```markdown
- [ ] E-01 — Tambah model Product
  - Selesai jika: migrasi tercatat dan test create/read lulus
  - File: prisma/schema.prisma
  - Risiko: perubahan skema produksi
```

## 7. Atur AGENTS

Tulis perintah resmi project, batas direktori, konvensi, larangan membaca/menulis `.env`, aturan migrasi, dan definisi selesai. AI agent tidak boleh:

- menghapus data atau volume;
- memakai `prisma db push` untuk produksi;
- menaruh secret dalam kode/log;
- deploy tanpa quality gate dan backup sebelum migrasi berisiko.

## 8. Review Konsistensi

```bash
npm run lint
npm test
npm run build
```

**Hasil yang diharapkan:** semua exit code `0`; bila script test belum ada, buat test minimal sebelum fase deploy, bukan menganggapnya lulus.

## Kegagalan Umum

| Gejala | Perbaikan |
|---|---|
| Scope terus bertambah | Pindahkan ke “tidak masuk versi pertama” di PRD |
| TASKS terlalu besar | Pecah per hasil teruji |
| Diagram berbeda dari implementasi | Jadikan ARCHITECTURE sumber keputusan dan perbarui saat keputusan berubah |
| Agent mengubah infra | Tegaskan batas izin di AGENTS |

## Checklist

- [ ] PRD punya scope dan kriteria terima.
- [ ] SDLC punya gerbang staging, produksi, rollback.
- [ ] DESIGN mencakup semua state dan aksesibilitas.
- [ ] ARCHITECTURE menyebut standalone, Coolify, Prisma, PostgreSQL.
- [ ] TASKS kecil, berurutan, dan dapat diuji.
- [ ] AGENTS melindungi secret, data, dan produksi.
- [ ] Enam dokumen tidak saling bertentangan.
