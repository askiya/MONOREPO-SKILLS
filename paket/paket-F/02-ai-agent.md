# 02 — Setup AI Agent (CLI-Based)

## Tujuan

Agent CLI (Claude Code atau Hermes Agent) siap di terminal, konteks proyek termuat, dan batas kerja ditegakkan melalui AGENTS.md.

## Prasyarat

- Repo dan enam dokumen perencanaan sudah tersedia.
- Terminal dan shell Bash/Zsh siap.

## 1. Pilih Agent

| Agent | Instalasi | Catatan |
|---|---|---|
| Claude Code | `npm install -g @anthropic-ai/claude-code` | Perlu API key atau langganan |
| Hermes Agent | Ikuti [dokumentasi Hermes](https://hermes-agent.nousresearch.com/docs) | Multi-tool, orkestrasi kuat |

Pilih salah satu. Keduanya CLI-based dan cocok untuk paket lanjutan yang banyak bekerja di terminal, Docker, dan server.

## 2. Pasang dan Verifikasi

### Claude Code

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

### Hermes Agent

Ikuti petunjuk instalasi resmi. Verifikasi:

```bash
hermes --version
```

**Hasil yang diharapkan:** versi tercetak tanpa error.

## 3. Muat Konteks

Buka terminal di root project, lalu berikan perintah agent:

```text
Baca file berikut dan jadikan panduan:
- PRD.md
- ARCHITECTURE.md
- TASKS.md
- AGENTS.md
```

Agent harus bisa merangkum:

- ada tiga service (web, API, DB);
- web = Next.js standalone, API = Express/Fastify, DB = PostgreSQL;
- deploy = Docker Compose + Nginx + Certbot;
- semua build harus standalone/multi-stage.

## 4. Atur Batasan

Pastikan `AGENTS.md` mencakup:

- Jangan membaca/cetak/commit `.env`.
- Jangan menghapus volume Docker (`docker compose down -v` terlarang).
- Jangan membuka port `5432` ke publik.
- Jangan mengubah firewall, SSH, atau Nginx produksi tanpa persetujuan.
- Migrasi produksi hanya setelah backup.
- Setiap tugas diakhiri `npm run lint`, `npm test`, `npm run build`.

## 5. Tugas Pertama

```text
Kerjakan tugas F-01 dari TASKS.md.
Verifikasi: jalankan lint dan test.
Laporkan file yang dibuat/diubah dan output test.
```

**Hasil yang diharapkan:** perubahan sesuai scope, lint/test lulus.

## 6. Alur Kerja Terminal

```text
1. Baca tugas dari TASKS → beri konteks ke agent.
2. Agent mengerjakan → review diff.
3. Jalankan quality gate.
4. Commit bila lulus.
5. Ulangi tugas berikutnya.
```

Jangan beri semua tugas sekaligus. Satu tugas, verifikasi, lalu lanjut.

## Kegagalan Umum

| Gejala | Perbaikan |
|---|---|
| Agent membuat Dockerfile tapi tidak multi-stage | Tegaskan di AGENTS: "Dockerfile wajib multi-stage, user non-root" |
| Agent menulis DB URL langsung di kode | Cek hasil, tegaskan hanya `env("DATABASE_URL")` |
| Agent mengubah config Nginx | Batasi scope tugas ke direktori app, bukan infra |
| Output terlalu panjang | Gunakan `--max-tokens` atau batasi cakupan tugas |

## Checklist

- [ ] CLI agent terpasang dan bisa dipanggil dari terminal.
- [ ] Agent membaca dan memahami dokumen perencanaan.
- [ ] AGENTS.md membatasi secret, volume, port, firewall.
- [ ] Tugas pertama selesai dan quality gate lulus.
