# PEDOMAN MONOREPO — Kelas Vibe Coding Santriverse

Kumpulan pedoman, template, prompt, dan skill untuk membangun produk digital
**dari nol sampai online** dengan cara **vibe coding** (nyuruh AI agent yang ngoding,
kita yang mikir produk, arsitektur, dan review).

Repo ini bukan teori. Ini urutan kerja yang dipakai beneran: pasang AI agent →
tulis dokumen (PRD, SDLC, DESIGN.md) → suruh agent build frontend + backend →
preview di localhost → testing → deploy gratisan → naik ke VPS/cPanel → domain +
pembayaran.

---

## Cara Pakai Repo Ini

1. Clone atau download repo ini.
2. Baca `docs/00-mulai-dari-sini/`.
3. Ikuti nomor folder `docs/` secara berurutan. Jangan lompat sebelum checklist
   di akhir tiap bab centang semua.
4. Copy isi `templates/` ke project kamu, isi, lalu masukkan ke AI agent.
5. Copy prompt dari `prompts/` saat butuh — jangan ngetik prompt dari nol.

> Aturan emas kelas ini: **AI agent tidak boleh menebak.** Kalau agent tidak
> punya dokumen, dia akan mengarang arsitektur. Dokumen dulu, kode belakangan.

---

## Peta Isi

| Folder | Isi |
|---|---|
| `docs/00-mulai-dari-sini/` | Filosofi vibe coding, alur besar, prasyarat laptop |
| `docs/01-setup-ai-agent/` | Antigravity (utama), Cursor, Claude Code, Codex, Hermes |
| `docs/02-dokumen-perencanaan/` | PRD, SDLC, DESIGN.md, ARCHITECTURE, TASKS, AGENTS.md |
| `docs/03-tech-stack/` | Pilihan framework & kapan dipakai |
| `docs/04-build-frontend/` | Next.js dari 0 lewat AI agent |
| `docs/05-build-backend/` | API, auth, database, struktur service |
| `docs/06-database/` | PostgreSQL, Prisma, migrasi, seed, backup |
| `docs/07-preview-localhost/` | Menjalankan & debug lokal |
| `docs/08-testing/` | Manual QA, unit test, e2e, gate sebelum deploy |
| `docs/09-deploy-gratis/` | Vercel, Netlify, Cloudflare Pages, Render, Railway, Fly |
| `docs/10-deploy-vps/` | VPS, Docker, Coolify, Nginx, SSL, backup |
| `docs/11-hosting-cpanel/` | Shared hosting cPanel: static, PHP, Node |
| `docs/12-domain-dns-cloudflare/` | Domain, DNS, SSL, subdomain staging |
| `docs/13-payment/` | Xendit, Lynk.id, webhook, aktivasi otomatis |
| `docs/14-keamanan-secret/` | .env, rotasi kunci, jangan bocor ke GitHub |
| `docs/15-troubleshooting/` | Error yang paling sering muncul + obatnya |
| `templates/` | File siap isi (PRD, SDLC, DESIGN.md, dst) |
| `prompts/` | Prompt siap tempel per fase |
| `skills/` | Skill AI agent (format Hermes/Claude) |
| `checklists/` | Checklist rilis, keamanan, performa |
| `kelas/` | Silabus kelas per sesi |

---

## Alur Besar (hafalkan ini)

```
IDE PRODUK
   ↓
PRD.md          apa yang dibangun & untuk siapa
   ↓
SDLC.md         tahapan kerja & definisi selesai
   ↓
DESIGN.md       warna, font, spacing, komponen
   ↓
ARCHITECTURE.md folder, stack, kontrak API
   ↓
TASKS.md        pecahan kerja kecil-kecil
   ↓
AI AGENT build  frontend + backend
   ↓
localhost       lihat, pakai, catat bug
   ↓
testing         gate: lint + build + test lolos
   ↓
deploy gratis   biar bisa dilihat orang (staging)
   ↓
VPS / cPanel    produksi + domain sendiri
```

---

## Prinsip Kelas

1. **Dokumen dulu.** Prompt tanpa dokumen = kode sampah yang rapi.
2. **Satu tugas satu commit.** Jangan suruh agent kerjakan 10 hal sekaligus.
3. **Baca diff.** Kalau kamu tidak paham perubahannya, jangan merge.
4. **Secret tidak pernah masuk Git.** Selamanya.
5. **Kalau belum jalan di localhost, jangan deploy.**
6. **Deploy gratisan dulu, VPS belakangan.** Hemat uang, cepat validasi.

---

## Lisensi & Kredit

Materi kelas Santriverse. Bebas dipakai murid kelas untuk project pribadi
maupun klien. Jangan dijual ulang sebagai produk terpisah.
