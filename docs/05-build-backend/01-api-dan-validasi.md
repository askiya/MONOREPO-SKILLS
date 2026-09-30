# 01 — Build Backend API

## Tujuan

Membuat API aman dengan kontrak jelas, validasi batas masuk, dan error konsisten.

## Urutan

1. Tulis endpoint di `ARCHITECTURE.md`.
2. Tulis skema database.
3. Implementasi read-only endpoint pertama.
4. Tambah create/update/delete satu per satu.
5. Tambah auth dan authorization.
6. Sambungkan frontend setelah API teruji sendiri.

## Bentuk Response Konsisten

Sukses:

```json
{ "data": { "id": "..." }, "error": null }
```

Gagal:

```json
{ "data": null, "error": { "code": "VALIDATION_ERROR", "message": "Email tidak valid", "fields": { "email": "Format salah" } } }
```

Gunakan status HTTP benar: `200`, `201`, `204`, `400`, `401`, `403`, `404`,
`409`, `422`, `500`.

## Validasi Trust Boundary

Selalu validasi:
- body JSON,
- query string dan pagination,
- path parameter,
- upload file (type, size),
- webhook dari pihak luar.

Jangan percaya validasi frontend; request bisa dikirim langsung.

## Prompt Implementasi

```text
Kerjakan T-031: POST /api/products sesuai kontrak ARCHITECTURE.md.
Baca schema database dan pola route tetangga. Validasi semua input di server,
wajib role admin, jangan kembalikan field internal. Tambah test sukses, invalid,
unauthorized, forbidden, duplicate. Jalankan test, lint, build. Jangan commit.
```

## Error Handling

- Log detail internal di server; response publik jangan bocorkan stack trace.
- Error DB unik menjadi `409`, bukan `500`.
- Resource tidak ada menjadi `404`.
- Request tanpa login `401`; login tapi bukan role benar `403`.

## Checklist

- [ ] Kontrak API tertulis dulu
- [ ] Input server tervalidasi
- [ ] Auth dan role diuji
- [ ] Error tidak bocorkan internal
- [ ] Test jalur sukses dan gagal lolos
