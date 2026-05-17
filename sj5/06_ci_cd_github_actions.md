# 6️⃣ CI/CD Sederhana dengan GitHub Actions

## Apa Tujuannya?

CI/CD membantu kita supaya proses build dan release Android tidak dilakukan manual terus-menerus.

| Bagian | Fungsi |
|---|---|
| **CI** | Jalankan test dan build otomatis saat ada perubahan kode |
| **CD** | Release otomatis ke artifact atau pipeline distribusi Android |

---

## Contoh Alur Sederhana

```text
Push ke main branch
        ↓
GitHub Actions jalan
        ↓
flutter pub get
flutter test
flutter build apk --release
flutter build appbundle --release
        ↓
Upload artifact / deploy
```

---

## 1. Workflow untuk Build Android

Buat file `.github/workflows/android.yml`:

```yaml
name: Build Flutter Android

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build APK release
        run: flutter build apk --release

      - name: Build AAB release
        run: flutter build appbundle --release

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: android-build
          path: |
            build/app/outputs/flutter-apk/app-release.apk
            build/app/outputs/bundle/release/app-release.aab
```

---

## 2. Workflow untuk Android Build

Kalau targetnya Android, workflow bisa build APK atau AAB:

```yaml
- name: Build APK
  run: flutter build apk --release

- name: Build AAB
  run: flutter build appbundle --release
```

Lalu artifact dapat diupload untuk dibagikan ke tim internal.

---

## 3. Secrets di GitHub

Jangan simpan credential di file workflow.

Gunakan **GitHub Secrets** untuk:
- API key,
- keystore password,
- service account,
- token deploy.

Contoh pemakaian:

```yaml
- name: Build with env
  run: flutter build apk --release --dart-define=API_URL=${{ secrets.API_URL }}
```

---

## 4. Kapan CD Full Otomatis Dipakai?

CD penuh cocok kalau:
- tim sudah punya workflow yang stabil,
- staging dan production jelas,
- dan release sudah rutin.

Kalau masih tahap awal, lebih aman:
- CI otomatis untuk test dan build,
- distribusi manual setelah artifact dicek.

---

## 5. Checklist Workflow

- [ ] Workflow berjalan di branch yang benar
- [ ] Test otomatis jalan dulu sebelum build
- [ ] Secret tidak ditulis hardcode
- [ ] Artifact hasil build bisa diunduh
- [ ] Build release berhasil di runner CI

---

*Study Jam 5 — Flutter Deployment*
