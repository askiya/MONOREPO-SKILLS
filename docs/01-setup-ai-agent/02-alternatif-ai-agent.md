# 02 — Alternatif AI Agent

## Tujuan

Memilih agent lain kalau Antigravity tidak cocok. Alur kelas tetap sama karena
sumber kebenaran ada di file Markdown, bukan di merek agent.

## Syarat Minimal Agent

Agent harus bisa:

1. Membaca banyak file dalam folder project.
2. Mengedit file langsung, bukan hanya memberi potongan kode.
3. Menjalankan terminal.
4. Menampilkan diff sebelum perubahan diterima.
5. Meminta izin sebelum aksi berisiko.

Kalau hanya chatbot web tanpa akses folder, bukan pilihan utama untuk kelas ini.

## Perbandingan Ringkas

| Agent | Bentuk | Cocok untuk | Catatan |
|---|---|---|---|
| Antigravity | editor/agent | jalur utama kelas | ikuti bab sebelumnya |
| Cursor | editor IDE | pemula visual | impor project, rules di root |
| Windsurf | editor IDE | pemula visual | workflow mirip Cursor |
| GitHub Copilot | ekstensi/agent | pengguna VS Code | butuh akun GitHub |
| Claude Code | terminal | project kompleks | nyaman untuk pengguna CLI |
| OpenAI Codex CLI | terminal | coding agent via terminal | cek provider/login |
| Gemini CLI | terminal | alternatif Google | cek kuota saat ini |
| Hermes Agent | desktop/terminal | provider fleksibel + skills | open source, multi-platform |
| Aider | terminal | edit berbasis Git | ringan, fokus pair programming |

Harga, kuota, model, dan dukungan OS berubah cepat. Cek website resmi sebelum
memilih — jangan menjadikan tabel harga screenshot sebagai sumber permanen.

## Cara Mengganti Agent Tanpa Ulang dari Nol

1. Commit semua perubahan di agent lama.
2. Tutup agent lama.
3. Buka **folder yang sama** di agent baru.
4. Berikan prompt onboarding:

```text
Baca AGENTS.md dan seluruh dokumen root Markdown. Jangan mengubah apa pun.
Jalankan git status dan git log -5 --oneline. Jelaskan kondisi project,
aturan, task aktif, dan risiko sebelum lanjut.
```

5. Cocokkan jawaban dengan kondisi project.
6. Lanjut dari task belum dicentang di `TASKS.md`.

## Prinsip Portabilitas

- Simpan aturan di `AGENTS.md`, bukan cuma memori chat agent.
- Simpan keputusan di `ARCHITECTURE.md`.
- Simpan progres di `TASKS.md` dan Git.
- Jangan bergantung pada fitur proprietary kalau file Markdown cukup.
- Chat bukan dokumentasi. Hal penting wajib masuk repo.

## Checklist

- [ ] Agent pilihan memenuhi 5 syarat minimal
- [ ] Semua konteks project tersimpan di repo
- [ ] Prosedur pindah agent sudah dipahami
- [ ] Harga/kuota dicek dari sumber resmi saat ini
