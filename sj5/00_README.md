# Flutter Android Deployment: Build, Release & Ship

## Study Jam Materials — GDG on Campus Brawijaya

Selamat datang di **Study Jam 5**! 🚀

Pada modul ini, kita akan belajar bagaimana cara **menyiapkan aplikasi Flutter untuk production**, membuat **build release**, mengatur **environment configuration**, lalu **mendistribusikan aplikasi Android** ke channel rilis yang tepat.

---

## Tech Stack

| Layer | Package / Service | Fungsi |
|---|---|---|
| **Build Tool** | `flutter build` | Generate release APK dan AAB |
| **Environment Config** | `--dart-define` | Menyimpan config per environment |
| **Signing** | Keystore / Play App Signing | Menandatangani aplikasi Android |
| **Distribution** | Play Console / Internal Testing | Distribusi aplikasi Android |
| **CI/CD** | GitHub Actions | Automasi build dan deploy |
| **Monitoring** | Firebase Crashlytics / Analytics | Memantau error dan usage setelah rilis |

---

## Daftar Materi

| No | File | Topik |
|---|---|---|
| 1 | [`01_konsep_deployment.md`](01_konsep_deployment.md) | Konsep deployment, release flow, dan target platform |
| 2 | [`02_setup_release_project.md`](02_setup_release_project.md) | Setup project untuk production-ready deployment |
| 3 | [`03_android_release_build.md`](03_android_release_build.md) | Build release Android: APK, AAB, signing, dan upload |
| 4 | [`04_android_distribution.md`](04_android_distribution.md) | Distribusi Android ke Internal Testing / Play Store |
| 5 | [`05_environment_config.md`](05_environment_config.md) | Environment config, flavor, dan `--dart-define` |
| 6 | [`06_ci_cd_github_actions.md`](06_ci_cd_github_actions.md) | CI/CD sederhana dengan GitHub Actions |
| 7 | [`07_play_store_publish.md`](07_play_store_publish.md) | Publish ke Google Play Console |
| 8 | [`08_monitoring_rollbacks.md`](08_monitoring_rollbacks.md) | Monitoring, crash reporting, dan rollback strategy |
| 9 | [`09_tips_best_practices.md`](09_tips_best_practices.md) | Tips, checklist, dan best practices deployment |

---

## Target Pembelajaran

Setelah menyelesaikan modul ini, peserta diharapkan:

1. Memahami perbedaan **debug**, **profile**, dan **release** mode di Flutter.
2. Bisa menyiapkan aplikasi Flutter untuk **production build**.
3. Membuat dan menandatangani **APK/AAB Android release** dengan benar.
4. Menyiapkan distribusi Android melalui **APK**, **AAB**, dan **testing track**.
5. Mengelola **environment** untuk staging dan production.
6. Menjalankan build dan release otomatis menggunakan **CI/CD**.
7. Mengetahui alur dasar publish aplikasi ke **Google Play Console**.
8. Menyusun checklist agar rilis Android lebih aman, rapi, dan mudah di-maintain.

---

## Prerequisites

Sebelum memulai, pastikan kamu sudah:

- ✅ Menyelesaikan dasar Flutter dan state management
- ✅ Memiliki project Flutter yang sudah berjalan
- ✅ Memiliki akun Google
- ✅ Memiliki akun GitHub
- ✅ Jika ingin publish Android: akses ke **Google Play Console**
- ✅ Akses ke **Google Play Console** atau jalur distribusi internal Android

---

## Quick Start

```bash
# Buat project Flutter baru
flutter create flutter_deployment_app
cd flutter_deployment_app

# Jalankan di mode debug dulu
flutter run

# Build release untuk Android
flutter build apk --release
flutter build appbundle --release
```

---

## Struktur Folder yang Disarankan

```
flutter_deployment_app/
├── lib/
│   ├── core/
│   │   ├── config/
│   │   │   └── app_config.dart
│   │   ├── env/
│   │   │   └── env.dart
│   │   └── utils/
│   │       └── logger.dart
│   ├── features/
│   │   └── home/
│   │       ├── presentation/
│   │       │   └── home_page.dart
│   │       └── widgets/
│   │           └── ...
│   └── main.dart
├── android/
│   └── app/
│       └── key.properties           # Signing config lokal
├── .github/
│   └── workflows/
│       └── android.yml              # CI/CD pipeline
└── pubspec.yaml
```

## Catatan Penting

- Jangan pernah mengirim file keystore ke publik.
- Jangan hardcode API key production di source code.
- Selalu uji build release sebelum upload ke store.
- Simpan dokumentasi versi build, changelog, dan SHA key untuk kebutuhan rilis.

---

*GDG on Campus Brawijaya — Tech Series Study Jam 2025*
