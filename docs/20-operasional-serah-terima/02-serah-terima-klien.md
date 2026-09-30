# 02 — Serah Terima Project ke Klien

Tujuan: klien bisa melanjutkan operasi tanpa bergantung pada laptop/developer lama.

## Aset yang Diserahkan

- [ ] Repo GitHub dipindah ke akun/organization klien
- [ ] Domain berada di akun klien
- [ ] Hosting/cloud berada di akun klien
- [ ] Database berada di akun klien
- [ ] Payment gateway atas nama klien
- [ ] Email transaksional di akun klien
- [ ] Analytics/monitoring di akun klien
- [ ] File desain/logo source

Developer boleh diberi akses kolaborator. **Jangan** membeli semua atas akun developer
lalu memberi password akun pribadi.

## Dokumen Wajib

1. README: cara run lokal.
2. ARCHITECTURE.md: stack + layanan.
3. ENVIRONMENT.md: **nama** env variable + lokasi penyimpanan (tanpa nilai).
4. DEPLOYMENT.md: cara deploy + rollback.
5. BACKUP.md: jadwal, lokasi, cara restore.
6. RUNBOOK.md: apa yang dilakukan saat situs mati.
7. Daftar vendor + tanggal renewal.

## Transfer Credential Aman

Jangan kirim password/API key via WhatsApp/email plain. Gunakan password manager
(1Password/Bitwarden) atau secret-sharing one-time link. Setelah transfer:
- klien mengganti password,
- aktifkan 2FA milik klien,
- hapus akses developer yang tidak perlu,
- rotasi API key sementara.

## Sesi Handover

Rekam layar (dengan secret disensor):
1. Login dashboard admin.
2. Tambah/edit produk/konten.
3. Lihat order/user.
4. Deploy perubahan kecil.
5. Restore backup ke environment uji.
6. Cek log dan monitoring.
7. Hubungi support vendor.

## Acceptance Test

Klien sendiri (bukan developer) melakukan:
- [ ] login admin,
- [ ] update konten,
- [ ] proses alur utama,
- [ ] menerima email,
- [ ] melihat laporan/order,
- [ ] mengakses backup,
- [ ] tahu kontak darurat.

## Dokumen Sign-off

```text
Project:
URL:
Tanggal serah terima:
Scope yang diterima:
Known issues:
Garansi/support sampai:
Biaya maintenance setelah garansi:
Diserahkan oleh:
Diterima oleh:
```

Serah terima selesai hanya setelah klien bisa mengoperasikan, bukan setelah ZIP
dikirim.
