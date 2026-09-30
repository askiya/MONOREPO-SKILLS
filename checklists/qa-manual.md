# Checklist Manual QA

## Semua Halaman
- [ ] Tidak ada error merah di console
- [ ] Tidak ada request 404/500 tak disengaja
- [ ] Heading dan hierarchy jelas
- [ ] Semua tombol/link berfungsi atau dihapus
- [ ] Focus keyboard terlihat
- [ ] Light mode punya kontras cukup

## Responsive
- [ ] 375px: tidak ada overflow horizontal
- [ ] 768px: layout tidak janggal
- [ ] 1280px: konten tidak terlalu melebar
- [ ] Menu mobile buka/tutup
- [ ] Tombol mudah ditekan
- [ ] Modal tidak terpotong

## Data State
- [ ] Loading terlihat
- [ ] Empty state informatif
- [ ] Error state punya retry bila masuk akal
- [ ] Success state memberi feedback
- [ ] Submit ganda dicegah

## Auth
- [ ] Register valid berhasil
- [ ] Register invalid memberi pesan field
- [ ] Login benar/salah sesuai
- [ ] Logout menghapus session
- [ ] Route member/admin terlindungi

## Alur Bisnis
- [ ] Pengguna baru bisa mencapai nilai utama produk
- [ ] CRUD utama berfungsi
- [ ] Refresh halaman tidak menghilangkan data
- [ ] Back/forward browser tidak merusak state
- [ ] Payment sandbox end-to-end bila ada
