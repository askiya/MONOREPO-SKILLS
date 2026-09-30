# 01 — CI dengan GitHub Actions

CI menjalankan pemeriksaan otomatis setiap push/pull request. Tujuan: error
ditangkap sebelum deploy, bukan setelah user menemukan.

## Workflow Minimum untuk Aplikasi Node.js

Buat `.github/workflows/ci.yml` di repo aplikasi:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build
```

`npm ci` dipakai di CI karena instalasi deterministik dari `package-lock.json`.
Jangan pakai `npm install` di CI.

## Environment Variable di CI

GitHub repo → Settings → Secrets and variables → Actions → New repository secret.

Di workflow:
```yaml
env:
  DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
```

Jangan menulis nilai secret langsung di file YAML.

## DB untuk Test

Pilihan:
1. Mock/stub untuk unit test.
2. PostgreSQL service container untuk integration test.
3. Database khusus CI dari provider.

Jangan pernah menjalankan test ke database produksi.

## Branch Protection

GitHub → Settings → Branches → Add branch protection rule:
- Branch name: `main`
- Require a pull request before merging
- Require status checks to pass
- Pilih job `test`
- Do not allow bypass (untuk tim)

## CI Repo Dokumentasi Ini

Repo MONOREPO-SKILLS memakai:
- link checker untuk link Markdown,
- Gitleaks untuk mendeteksi secret.

Workflow ada di `.github/workflows/docs-quality.yml`.

## Checklist

- [ ] Workflow trigger pada pull_request + main
- [ ] `npm ci`, lint, test, build dijalankan
- [ ] Permission minimum (`contents: read`)
- [ ] Secret di GitHub Secrets, bukan YAML
- [ ] Test tidak menyentuh database produksi
- [ ] Branch protection mewajibkan CI hijau
