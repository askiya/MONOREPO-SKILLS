# Paket A — Vercel Starter

> 🟢 **PEMULA** · Estimasi: 2 hari · Biaya: **GRATIS**

Paket default kelas. Cocok untuk member yang baru pertama kali bikin web app
dengan login dan database, ingin cepat online tanpa ribet server.

---

## Stack Diagram

```text
┌─────────────────────────────────────────────────────────────┐
│                        BROWSER                              │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTPS
┌──────────────────────────▼──────────────────────────────────┐
│                     VERCEL (CDN)                            │
│  ┌───────────────┐   ┌────────────────────────────────┐    │
│  │  Static Files  │   │  Serverless Functions (API)    │    │
│  │  (Next.js SSG) │   │  NextAuth + API Routes         │    │
│  └───────────────┘   └──────────────┬─────────────────┘    │
└─────────────────────────────────────┼───────────────────────┘
                                      │ TCP/SSL
                       ┌──────────────▼──────────────┐
                       │     NEON PostgreSQL (free)   │
                       │     + Prisma ORM             │
                       └─────────────────────────────┘

DNS: Cloudflare → CNAME → cname.vercel-dns.com
```

---

## Kapan Pakai Paket Ini

- Baru pertama kali bikin web app sampai deploy
- Butuh login (auth) + database
- Mau gratis 100% (free tier Vercel + Neon)
- Project pribadi, MVP, atau portofolio interaktif
- Tim 1–2 orang

## Kapan JANGAN Pakai

- Landing page statis tanpa login → pakai **Paket B**
- Butuh cron job, background worker, atau WebSocket → pakai **Paket G (Railway)**
- Klien minta cPanel → pakai **Paket C**
- Butuh kontrol penuh server → pakai **Paket E atau F**
- Traffic tinggi / multi-service → pakai **Paket I**

---

## Prasyarat

| Kebutuhan | Cara Cek |
|---|---|
| Node.js LTS | `node -v` → harus ≥ 18 |
| Git | `git --version` |
| Akun GitHub | [github.com](https://github.com) |
| Akun Vercel | [vercel.com](https://vercel.com) — login pakai GitHub |
| Akun Neon | [neon.tech](https://neon.tech) — free tier |
| Akun Cloudflare | [cloudflare.com](https://cloudflare.com) |
| Antigravity terinstal | Baca `02-ai-agent.md` |
| Domain (opsional) | Beli di Niagahoster / Cloudflare Registrar |

Kalau belum ada satupun, ikuti [`../../docs/00-mulai-dari-sini/02-prasyarat-alat.md`](../../docs/00-mulai-dari-sini/02-prasyarat-alat.md) dulu.

---

## Alur Lengkap

```text
1. Pedoman ──► 2. AI Agent ──► 3. Build ──► 4. Preview ──► 5. Deploy ──► 6. Domain ──► 7. Maintenance
   (docs)       (setup)        (code)      (test)        (Vercel)     (DNS/SSL)    (monitoring)
```

| Fase | File | Waktu |
|---|---|---|
| 1. Pedoman — tulis 6 dokumen perencanaan | [`01-pedoman.md`](01-pedoman.md) | 2–4 jam |
| 2. AI Agent — instal & konfigurasi Antigravity | [`02-ai-agent.md`](02-ai-agent.md) | 30 menit |
| 3. Build — Next.js + Prisma + NextAuth | [`03-build.md`](03-build.md) | 4–8 jam |
| 4. Preview — localhost, lint, test | [`04-preview.md`](04-preview.md) | 1–2 jam |
| 5. Deploy — push ke Vercel | [`05-deploy.md`](05-deploy.md) | 30–60 menit |
| 6. Domain & SSL — Cloudflare DNS | [`06-domain-ssl.md`](06-domain-ssl.md) | 30 menit |
| 7. Maintenance — monitoring & backup | [`07-maintenance.md`](07-maintenance.md) | 30 menit |
| 8. Rekomendasi Hosting — opsi provider | [`08-rekomendasi-hosting.md`](08-rekomendasi-hosting.md) | referensi |

---

## Video Tutorial

> 🎬 Video belum tersedia. Akan ditambahkan.

Saat video sudah ada, link akan muncul di setiap file fase.

---

## Checklist Selesai

- [ ] 6 dokumen perencanaan selesai (PRD, SDLC, DESIGN, ARCHITECTURE, TASKS, AGENTS)
- [ ] Antigravity terinstal, bisa baca folder project
- [ ] Next.js jalan di localhost tanpa error
- [ ] Prisma terhubung ke Neon, `npx prisma db push` sukses
- [ ] NextAuth login berfungsi di localhost
- [ ] `npm run lint` — 0 error
- [ ] `npm run build` — sukses tanpa error
- [ ] Deploy di Vercel — halaman terbuka dari URL `.vercel.app`
- [ ] Domain custom mengarah ke Vercel (HTTPS aktif, gembok hijau)
- [ ] UptimeRobot memantau URL produksi
- [ ] Backup database Neon diaktifkan

Kalau semua centang, Paket A selesai. Lanjut ke project berikutnya atau naik ke
paket yang lebih tinggi.

---

## Referensi

| Topik | Lokasi |
|---|---|
| Panduan lengkap docs/ | [`../../docs/`](../../docs/) |
| Template kosong | [`../../templates/`](../../templates/) |
| Contoh terisi | [`../../examples/toko-produk-digital/`](../../examples/toko-produk-digital/) |
| Config Vercel | [`../../deployment-examples/vercel/`](../../deployment-examples/vercel/) |
| Troubleshooting | [`../../docs/15-troubleshooting/`](../../docs/15-troubleshooting/) |
