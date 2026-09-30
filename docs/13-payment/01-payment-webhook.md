# 01 — Payment Gateway dan Webhook

## Tujuan

Pembayaran tervalidasi server dan akun/produk aktif otomatis tanpa percaya
redirect browser.

## Pilihan Indonesia

- Xendit
- Midtrans
- Lynk.id (integrasi sesuai API/webhook merchant)

Biaya dan fitur berubah; cek dokumentasi dan pricing resmi.

## Alur Aman

```
User klik Bayar
  → server buat order status PENDING
  → server minta invoice/payment link ke provider
  → user bayar di provider
  → provider kirim webhook ke server
  → server verifikasi signature/token webhook
  → transaksi DB atomik: order PAID + entitlement aktif
  → server balas 2xx
```

Redirect "payment success" di browser bukan bukti pembayaran. Sumber kebenaran
adalah webhook tervalidasi atau pengecekan API provider dari server.

## Data Minimum Order

- `id` internal unik
- `externalId`/reference provider unik
- `userId`
- `productId`
- `amount` (integer)
- `status`: PENDING, PAID, FAILED, EXPIRED, REFUNDED
- `provider`
- `paidAt`
- timestamps

## Webhook Wajib

1. Baca raw body bila algoritma signature membutuhkannya.
2. Verifikasi signature/token dengan secret server.
3. Cocokkan nominal, currency, dan reference ID dengan order DB.
4. Idempotent: webhook duplikat tidak mengaktifkan dua kali.
5. Update order dan entitlement dalam satu transaksi DB.
6. Balas cepat 2xx; pekerjaan lambat masuk job queue bila ada.
7. Log event ID/status tanpa mencatat secret atau data kartu.

## Prompt Agent

```text
Implementasikan webhook payment sesuai dokumentasi resmi provider yang sudah
ada di ARCHITECTURE.md. Verifikasi signature dari raw body, cocokkan amount dan
externalId, proses idempotent, dan lakukan order PAID + entitlement dalam satu
transaksi DB. Tambah test invalid signature, nominal salah, order tak ditemukan,
sukses, dan webhook duplikat. Jangan log secret.
```

## Sandbox ke Produksi

1. Selesaikan semua kasus di sandbox.
2. Deploy endpoint webhook staging dengan HTTPS publik.
3. Kirim test webhook dari dashboard provider.
4. Verifikasi DB berubah sekali.
5. Uji refund/expired bila didukung.
6. Ganti ke production key hanya di environment produksi.
7. Lakukan transaksi kecil sungguhan end-to-end.

## Checklist

- [ ] Redirect browser tidak dipercaya
- [ ] Signature webhook diverifikasi
- [ ] Nominal + reference dicocokkan
- [ ] Webhook idempotent
- [ ] Mutasi DB atomik
- [ ] Test duplikat/invalid lolos
- [ ] Secret sandbox dan produksi terpisah
