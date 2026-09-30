# 02 — Legal Minimum Produk Digital

> Ini panduan operasional umum, bukan nasihat hukum. Untuk bisnis berisiko tinggi,
> data sensitif, atau regulasi khusus, konsultasikan praktisi hukum Indonesia.

## Halaman Minimum

1. **Privacy Policy**
   - data apa yang dikumpulkan,
   - tujuan penggunaan,
   - penyedia pihak ketiga (payment, analytics, email),
   - berapa lama disimpan,
   - cara user meminta akses/hapus data,
   - kontak pengelola.
2. **Terms of Service**
   - siapa boleh memakai,
   - aturan akun,
   - batas tanggung jawab,
   - larangan penyalahgunaan,
   - penghentian akun,
   - hukum/yurisdiksi yang berlaku.
3. **Refund Policy**
   - apakah produk digital bisa refund,
   - syarat dan batas waktu,
   - proses klaim,
   - pengecualian.
4. **Contact** — alamat email/WhatsApp bisnis yang aktif.

## Cookie Consent

Tidak semua cookie butuh banner. Bedakan:
- **esensial**: login/session/cart — perlu agar app berfungsi,
- **analytics/marketing**: tracking — minta consent sebelum aktif bila diwajibkan
  kebijakan/regulasi target pasar.

Jangan tampilkan banner palsu yang tombol "Tolak"-nya tidak berfungsi.

## Prinsip Data Minimum

- Jangan kumpulkan data yang tidak dipakai.
- Jangan minta KTP/tanggal lahir kalau tidak perlu.
- Password harus di-hash, bukan disimpan plain.
- Data payment sensitif ditangani provider payment, bukan server sendiri.
- Sediakan cara user minta hapus akun/data.

## Checkbox Persetujuan

Saat register/checkout:
```text
[ ] Saya menyetujui Syarat Layanan dan Kebijakan Privasi
```
Jangan dicentang default. Simpan timestamp + versi dokumen yang disetujui.

## Checklist Sebelum Jualan

- [ ] Privacy Policy tersedia dan sesuai app nyata
- [ ] Terms of Service tersedia
- [ ] Refund Policy jelas
- [ ] Kontak bisnis aktif
- [ ] Checkbox consent tidak pre-checked
- [ ] Tidak mengumpulkan data berlebihan
- [ ] Daftar third-party lengkap
- [ ] Tanggal update dokumen terlihat
