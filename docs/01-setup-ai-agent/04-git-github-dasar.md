# 04 — Git dan GitHub untuk Vibe Coding

## Tujuan

Punya checkpoint aman setiap task dan repo GitHub tanpa secret.

## Buat Repo Lokal

Di root project:

```bash
git init
git branch -M main
git status
```

Pastikan `.gitignore` ada **sebelum** commit pertama.

## Commit Pertama

```bash
git add README.md AGENTS.md PRD.md SDLC.md DESIGN.md ARCHITECTURE.md TASKS.md .gitignore
git status
git commit -m "docs: initialize project specification"
```

Jangan pakai `git add .` sebelum melihat `git status`.

## Buat Repo GitHub

1. Login https://github.com.
2. Klik **New repository**.
3. Nama repo pakai huruf kecil dan tanda hubung.
4. Pilih **Private** untuk project klien/internal.
5. Jangan centang README kalau repo lokal sudah punya README.
6. Copy URL repo.

Hubungkan:

```bash
git remote add origin https://github.com/USERNAME/NAMA-REPO.git
git push -u origin main
```

## Siklus Harian

```bash
git status
git diff
# setelah yakin
git add path/file-yang-disengaja
git commit -m "feat(products): add product list page"
git push
```

## Commit Message

Format: `jenis(area): hasil`

| Jenis | Kapan |
|---|---|
| `feat` | fitur baru |
| `fix` | perbaikan bug |
| `docs` | dokumentasi |
| `test` | test |
| `refactor` | ubah struktur tanpa ubah perilaku |
| `chore` | konfigurasi/perawatan |

## Balik ke Checkpoint

Lihat riwayat:

```bash
git log --oneline -10
```

Buang perubahan **belum commit** pada satu file:

```bash
git restore path/file.ts
```

Perintah destruktif seperti `git reset --hard` atau force push tidak boleh
dijalankan agent tanpa persetujuan dan backup.

## Cek Secret Sebelum Push

```bash
git status
git diff --cached
```

Pastikan tidak ada `.env`, password, token, cookie, private key, atau file sesi.
Kalau secret pernah ter-commit, menghapus file saja tidak cukup: rotasi secret,
lalu bersihkan history dengan prosedur khusus.

## Checklist

- [ ] Repo lokal aktif
- [ ] `.gitignore` dibuat sebelum commit
- [ ] Repo GitHub terhubung
- [ ] Commit memakai nama jelas
- [ ] `git diff --cached` dicek sebelum push
- [ ] Tidak ada secret dalam Git
