# Checklist Keamanan Minimum

## Secret
- [ ] `.env` dan varian lokal di `.gitignore`
- [ ] `.env.example` hanya placeholder
- [ ] Tidak ada key/token/password di Git history terbaru
- [ ] Secret server tidak memakai prefix publik
- [ ] Secret staging dan produksi berbeda

## Authentication
- [ ] Password di-hash Argon2id/bcrypt
- [ ] Cookie session HttpOnly, Secure produksi, SameSite sesuai flow
- [ ] Login/register/reset password di-rate-limit
- [ ] Pesan reset tidak bocorkan keberadaan akun
- [ ] Session expiry dan logout berfungsi

## Authorization
- [ ] Role dicek server pada tiap endpoint sensitif
- [ ] 401 tanpa auth, 403 role salah
- [ ] User hanya bisa melihat/mengubah resource miliknya
- [ ] Admin route tidak hanya disembunyikan dari UI

## API
- [ ] Body/query/path tervalidasi
- [ ] Upload dibatasi type dan size
- [ ] Error publik tidak bocorkan stack/SQL
- [ ] CORS dibatasi sesuai origin bila diperlukan
- [ ] SSRF/open redirect/path traversal ditinjau bila relevan

## Infrastruktur
- [ ] HTTPS wajib
- [ ] SSH key, root/password login dimatikan setelah diuji
- [ ] Firewall hanya membuka port perlu
- [ ] Database tidak terbuka ke internet tanpa pembatasan
- [ ] Backup terenkripsi dan di luar server
- [ ] Dependency update/audit terjadwal

## Payment
- [ ] Webhook signature diverifikasi
- [ ] Amount/currency/reference diverifikasi
- [ ] Idempotency constraint ada
- [ ] Secret/card data tidak di-log
