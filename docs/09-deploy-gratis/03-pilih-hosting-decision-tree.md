# 03 — Pilih Hosting: Decision Tree

## Pohon Keputusan

```text
Apakah website statis? (HTML/CSS/JS, tidak butuh server/database)
├── Ya → Cloudflare Pages / GitHub Pages / Netlify
│         biaya: GRATIS
│
└── Tidak
    ├── Next.js sederhana (SSR, API routes ringan)?
    │     → Vercel
    │       biaya: GRATIS (hobby) → MURAH saat naik
    │
    ├── PHP / WordPress / CMS?
    │     → cPanel shared hosting
    │       biaya: MURAH (bulanan)
    │
    ├── Node.js kecil DAN hosting mendukung Setup Node.js App?
    │     → cPanel Node
    │       biaya: MURAH
    │       catatan: RAM kecil, build di lokal
    │
    ├── Butuh Docker / worker / WebSocket / cron berat?
    │     → VPS + Coolify (atau Docker manual)
    │       biaya: MENENGAH + PERLU OPERASIONAL SERVER
    │
    ├── Butuh backend Python/Go terpisah?
    │     → Render / Railway / Cloud Run (backend)
    │       + Vercel (frontend)
    │       biaya: GRATIS tier → MURAH
    │
    └── Belum tahu / masih eksperimen?
          → Vercel staging dulu, putuskan nanti
            biaya: GRATIS
```

## Kategori Biaya

Angka rupiah cepat basi. Pakai kategori:

| Kategori | Arti | Contoh |
|---|---|---|
| **GRATIS** | tier gratis cukup untuk belajar & staging | Cloudflare Pages, Vercel Hobby, Neon free |
| **MURAH** | biaya bulanan kecil, sekali bayar tinggal pakai | shared hosting cPanel, domain |
| **MENENGAH** | biaya bulanan tetap, kapasitas jelas | VPS 2vCPU/4GB, managed DB |
| **PERLU OPERASIONAL SERVER** | biaya + waktu kamu untuk patch, backup, monitoring, insiden | VPS tanpa panel, Kubernetes |

Kategori terakhir sering dilupakan pemula. VPS "murah" jadi mahal kalau kamu
menghabiskan 10 jam/bulan mengurusnya.

## Pertanyaan Penyaring

Jawab sebelum memilih:

1. **Apakah app butuh proses yang hidup terus?** (WebSocket, worker, queue)
   Ya → bukan serverless, bukan shared hosting.
2. **Apakah ada upload file user?**
   Ya → butuh object storage, bukan disk container.
3. **Apakah butuh database?**
   Ya → pilih managed DB, jangan pasang di server yang sama tanpa backup.
4. **Berapa lama request terlama?**
   Lebih dari batas platform serverless → pakai container/VPS.
5. **Siapa yang akan maintain 6 bulan lagi?**
   Kalau cuma kamu dan waktumu sedikit → hindari VPS manual.
6. **Kalau situs mati jam 2 pagi, siapa yang bangun?**
   Tidak ada → pilih platform managed.

## Kombinasi yang Umum Dipakai

| Skenario | Kombinasi |
|---|---|
| Landing page kelas | Cloudflare Pages + domain Cloudflare |
| MVP SaaS | Vercel + Neon + Cloudflare DNS |
| Toko digital Indonesia | Vercel + Neon + Xendit + Cloudflare |
| Client WordPress | cPanel + AutoSSL + Cloudflare |
| Produk skala naik | VPS Coolify + managed PostgreSQL + R2 |
| Backend AI/Python | Vercel (FE) + Render/Cloud Run (BE) + managed DB |

## Kapan Pindah Platform

Pindah dari gratis ke berbayar/VPS kalau:

- kena limit build/bandwidth secara rutin,
- butuh proses background yang tidak didukung,
- cold start mengganggu pengguna nyata,
- butuh kontrol jaringan/IP tetap,
- biaya platform managed melebihi biaya VPS + waktumu.

Jangan pindah karena "VPS lebih keren". Pindah karena ada masalah nyata.

## Checklist

- [ ] Saya tahu app saya statis atau butuh server
- [ ] Saya tahu butuh database atau tidak
- [ ] Saya tahu ada upload file atau tidak
- [ ] Saya sudah pilih kategori biaya yang sanggup saya tanggung
- [ ] Saya tahu siapa yang maintain
- [ ] Pilihan hosting saya tertulis di `ARCHITECTURE.md`
