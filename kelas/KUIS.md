# Kuis Pemahaman Risiko

Kuis ini bukan tentang menghafal. Ini memastikan kamu paham **kenapa** aturan ada,
bukan cuma mengikuti langkah.

Jawab di kepala / tulis jawaban sendiri sebelum lihat jawaban di bawah.

---

## Setelah Fase 2 — Dokumen

1. **Kenapa PRD ditulis sebelum kode?**
2. **Apa yang terjadi kalau agent tidak diberi ARCHITECTURE.md?**
3. **Apa bedanya task yang baik dan task yang buruk? Beri contoh.**
4. **Kenapa "Batasan (Out of Scope)" penting ditulis?**
5. **Kenapa definisi selesai harus bisa dites, bukan "sudah bagus"?**

---

## Setelah Fase 4 — Localhost

6. **Kenapa state loading, empty, dan error wajib ada?**
7. **Apa bedanya validasi client dan validasi server? Kenapa keduanya perlu?**
8. **Kenapa tombol yang tidak berfungsi harus dihapus, bukan dibiarkan?**
9. **Apa yang harus dicek sebelum menyebut halaman "selesai"?**
10. **Kenapa responsive diuji di 375px, bukan langsung desktop saja?**

---

## Setelah Fase 6 — Testing & Deploy

11. **Kenapa `.env` tidak boleh masuk Git, padahal repo private?**
12. **Apa beda `401` dan `403`?**
13. **Kenapa redirect "payment success" di browser bukan bukti pembayaran?**
14. **Apa beda staging dan production?**
15. **Kenapa backup database harus pernah di-restore, bukan cuma ada file-nya?**

---

## Setelah Fase 8 — Production

16. **Kenapa webhook payment harus idempotent?**
17. **Apa yang harus dilakukan pertama kalau secret terlanjur ter-push ke GitHub?**
18. **Kenapa monitoring uptime penting meskipun situs sudah jalan?**
19. **Apa bedanya hash dan enkripsi? Mana untuk password?**
20. **Kalau hosting diretas, kenapa menghapus file aneh saja tidak cukup?**

---

## Kunci Jawaban Singkat

<details>
<summary>Klik untuk buka</summary>

1. Tanpa PRD, agent menebak fitur. Revisi berputar-putar.
2. Agent akan memilih stack dan folder sendiri, bisa berbeda setiap sesi.
3. Buruk: "bikin auth". Baik: "POST /api/auth/register, validasi email+password,
   test 3 kasus". Task buruk tidak punya definisi selesai.
4. Supaya agent (dan kamu) tahu batas. Tanpa scope, semua fitur "sekalian aja".
5. "Sudah bagus" itu opini. "npm run build sukses" itu fakta.
6. User melihat layar kosong/error tidak tertangani → kira app rusak.
7. Client: UX cepat. Server: keamanan (request bisa dikirim langsung tanpa browser).
8. Kontrol mati merusak kepercayaan user. UI harus jujur.
9. Jalan, tidak ada error console, state loading/empty/error, responsive 375px, build sukses.
10. Mayoritas akses dari HP. Desktop bagus tapi HP potong = cacat.
11. Repo bisa bocor, kolaborator bisa berubah, Git history menyimpan selamanya.
12. 401 = siapa kamu (belum login). 403 = boleh tidak (sudah login, role salah).
13. User bisa memanipulasi URL. Sumber kebenaran: webhook tervalidasi dari server.
14. Staging = uji coba; production = nyata. DB, secret, dan URL harus terpisah.
15. File backup bisa corrupt, format salah, atau kosong. Restore drill satu-satunya bukti.
16. Provider bisa mengirim webhook lebih dari sekali. Tanpa idempotency, akses/order dobel.
17. Cabut/rotate secret **segera**. Menghapus file saja tidak menghapus dari Git history.
18. Website bisa mati kapan saja: disk penuh, sertifikat expired, container restart.
    Monitoring memberi tahu sebelum user complain.
19. Hash satu arah (tidak bisa dikembalikan), untuk password. Enkripsi dua arah (bisa
    di-decrypt dengan key), untuk data yang perlu dibaca kembali.
20. Penyerang bisa menyisipkan backdoor di file lain, cron, forwarder email, akun DB.
    Harus restore dari backup bersih + patch kerentanan.

</details>

## Cara Pakai

- Jawab sendiri dulu **sebelum** buka kunci jawaban.
- Kalau jawaban kamu beda tapi tetap benar, bagus — artinya paham konteks.
- Kalau salah 3+ soal dalam satu bagian, baca ulang bab terkait.
- Tidak ada nilai. Ini untuk diri sendiri.
