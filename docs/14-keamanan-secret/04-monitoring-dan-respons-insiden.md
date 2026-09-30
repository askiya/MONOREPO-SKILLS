# 04 — Monitoring dan Respons Insiden

## Tujuan

Tahu aplikasi rusak atau diserang sebelum pengguna yang melapor.

## Monitoring Minimum

| Hal | Pantau |
|---|---|
| Uptime | homepage, API health endpoint |
| Error | rasio 5xx, exception baru |
| Infrastruktur | CPU, RAM, disk, container restart |
| Database | koneksi, storage, slow query, backup |
| Auth | lonjakan gagal login/reset |
| Payment | webhook gagal, order pending terlalu lama |
| SSL/domain | masa berlaku certificate/domain |

## Opsi Tool

- Uptime: UptimeRobot, Better Stack, Healthchecks, self-hosted Uptime Kuma.
- Error tracking: Sentry atau alternatif kompatibel.
- Log: log platform hosting, Better Stack, Grafana Loki.
- Metrics: provider dashboard, Prometheus + Grafana bila skala menuntut.

Mulai dari uptime + error tracking. Jangan membangun observability stack besar
untuk aplikasi kecil.

## Health Endpoint

`GET /api/health` sebaiknya:
- cepat,
- tidak butuh auth,
- tidak membocorkan versi dependency/credential,
- mengembalikan status aplikasi dan, bila perlu, koneksi DB dengan timeout.

Contoh response:

```json
{ "status": "ok", "timestamp": "2026-01-01T00:00:00Z" }
```

## Severity Insiden

| Level | Contoh | Target respons |
|---|---|---|
| SEV-1 | seluruh app mati, data bocor | segera |
| SEV-2 | payment/login gagal luas | < 1 jam |
| SEV-3 | fitur sekunder rusak | hari kerja |
| SEV-4 | kosmetik/dokumentasi | backlog |

## Respons

1. Nyatakan insiden + severity.
2. Kurangi dampak: rollback, maintenance, matikan fitur berisiko.
3. Simpan log/bukti; jangan langsung menghapus semuanya.
4. Cari root cause berdasarkan timeline.
5. Pulihkan dan verifikasi alur kritis.
6. Rotate secret jika ada kemungkinan terekspos.
7. Tulis postmortem: dampak, timeline, sebab, perbaikan permanen.

## Checklist

- [ ] Monitor uptime aktif
- [ ] Error tracking aktif
- [ ] Alert sampai ke orang yang bertanggung jawab
- [ ] Disk/backup/SSL/domain dipantau
- [ ] Severity dan prosedur rollback diketahui
- [ ] Insiden diuji lewat simulasi sederhana
