# 03 — Alur Kerja Lengkap (Peta Kelas)

## Tujuan

Melihat seluruh perjalanan dari ide sampai produk online, supaya tahu sedang
berada di mana.

## Sepuluh Fase

| Fase | Nama | Output | Bab |
|---|---|---|---|
| 1 | Pasang AI agent | Agent bisa baca/tulis folder project | `docs/01` |
| 2 | Tulis dokumen | PRD, SDLC, DESIGN.md, ARCHITECTURE, TASKS | `docs/02` |
| 3 | Pilih stack | Keputusan framework tertulis | `docs/03` |
| 4 | Build frontend | Halaman jalan di localhost | `docs/04` |
| 5 | Build backend | API + auth jalan | `docs/05` |
| 6 | Database | Skema + migrasi + seed | `docs/06` |
| 7 | Preview lokal | Bisa diklik sendiri, bug tercatat | `docs/07` |
| 8 | Testing | lint + build + test hijau | `docs/08` |
| 9 | Deploy gratis | URL publik bisa dibagikan | `docs/09` |
| 10 | Produksi | VPS/cPanel + domain + pembayaran | `docs/10`–`13` |

## Aturan Perpindahan Fase (Gate)

Jangan pindah fase sebelum gate terpenuhi:

- **Fase 2 → 3**: PRD dibaca ulang keesokan hari dan masih masuk akal.
- **Fase 3 → 4**: stack ditulis di `ARCHITECTURE.md`, bukan cuma di kepala.
- **Fase 4 → 5**: halaman utama tampil tanpa error merah di console browser.
- **Fase 6 → 7**: `npm run build` sukses.
- **Fase 8 → 9**: lint, build, dan test lolos semua.
- **Fase 9 → 10**: URL staging sudah dicoba minimal 3 orang selain kamu.

## Ritme Harian yang Sehat

```
1. Buka TASKS.md, pilih SATU task.
2. Tempel prompt dari prompts/ ke AI agent.
3. Agent kerja → baca diff.
4. Jalankan di localhost → coba sendiri.
5. Lolos? commit. Gagal? kasih agent pesan error apa adanya.
6. Centang task, tutup laptop.
```

Satu task selesai per sesi jauh lebih baik daripada lima task setengah jadi.

## Estimasi Waktu Realistis (pemula)

| Fase | Waktu |
|---|---|
| Pasang agent | 1 jam |
| Dokumen | 3–6 jam |
| Frontend dasar | 1–2 hari |
| Backend + DB | 2–4 hari |
| Testing | 1 hari |
| Deploy gratis | 1–2 jam |
| VPS + domain | 3–5 jam |

Kalau lebih lama, wajar. Yang tidak wajar: 3 minggu tanpa satu pun halaman jalan.

## Checklist

- [ ] Saya tahu sedang di fase berapa
- [ ] Saya tahu gate untuk pindah fase
- [ ] Saya sanggup satu task per sesi
