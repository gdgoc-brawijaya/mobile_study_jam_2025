# Flutter Data Architecture: Fetch, Manage, Persist

## Study Jam Materials — GDG on Campus

Selamat datang di materi lanjutan Study Jam Flutter! 🎉

Pada modul ini, kita akan belajar bagaimana aplikasi Flutter **berkomunikasi dengan internet**, **mengelola state secara efisien**, dan **menyimpan data secara lokal** — menggunakan stack yang production-ready.

---

## Tech Stack

| Layer | Package | Fungsi |
|---|---|---|
| **Networking** | `dio` | HTTP Client untuk REST API |
| **State Management** | `flutter_bloc` | Bloc / Cubit pattern |
| **Local Storage** | `shared_preferences` | Persist data ringan |
| **Model** | `freezed` + `json_serializable` | Data class & JSON parsing |
| **DI / Service Locator** | `get_it` | Dependency Injection |
| **Env Config** | `flutter_dotenv` | Menyimpan API Key & Base URL |

---

## Daftar Materi

| No | File | Topik |
|---|---|---|
| 1 | [`01_arsitektur_data.md`](01_arsitektur_data.md) | Konsep Aliran Data & Clean Architecture |
| 2 | [`02_setup_project.md`](02_setup_project.md) | Setup project, dependencies, struktur folder |
| 3 | [`03_fetch_dio.md`](03_fetch_dio.md) | HTTP Request dengan Dio + Interceptor |
| 4 | [`04_model_freezed.md`](04_model_freezed.md) | Data Model dengan Freezed & JSON Parsing |
| 5 | [`05_bloc_cubit.md`](05_bloc_cubit.md) | State Management dengan Bloc & Cubit |
| 6 | [`06_shared_preferences.md`](06_shared_preferences.md) | Local Storage dengan Shared Preferences |
| 7 | [`07_dependency_injection.md`](07_dependency_injection.md) | Dependency Injection dengan GetIt |
| 8 | [`08_integrasi_lengkap.md`](08_integrasi_lengkap.md) | Integrasi penuh: Dio + Cubit + SharedPrefs |
| 9 | [`09_error_handling.md`](09_error_handling.md) | Error Handling, Loading State & UX |
| 10 | [`10_tips_best_practices.md`](10_tips_best_practices.md) | Tips, Best Practices & Checklist |

---

## Target Pembelajaran

Setelah menyelesaikan modul ini, peserta diharapkan:

1. Memahami alur data modern di aplikasi Flutter (Clean Architecture).
2. Bisa melakukan HTTP Request (GET/POST) menggunakan **Dio** beserta Interceptor.
3. Membuat Data Model yang aman dengan **Freezed** dan `json_serializable`.
4. Memisahkan logika bisnis dan UI menggunakan **Bloc/Cubit**.
5. Membuat UI yang reaktif dengan `BlocBuilder` dan `BlocListener`.
6. Menyimpan data sesi user dengan **Shared Preferences**.
7. Mengelola dependensi dengan **GetIt** agar kode mudah di-test.
8. Menerapkan Error Handling yang baik dan user-friendly.

---

## Quick Start

```bash
# Clone atau buat project baru
flutter create flutter_data_arch
cd flutter_data_arch

# Install semua dependency
flutter pub add dio flutter_bloc shared_preferences get_it freezed_annotation json_annotation flutter_dotenv

# Install dev dependency
flutter pub add --dev build_runner freezed json_serializable
```

---

## Struktur Folder Final

```
lib/
├── core/
│   ├── di/
│   │   └── injection.dart          # GetIt setup
│   ├── network/
│   │   ├── dio_client.dart          # Dio instance & config
│   │   └── interceptors/
│   │       ├── auth_interceptor.dart
│   │       └── log_interceptor.dart
│   ├── error/
│   │   └── failures.dart           # Custom failure classes
│   └── constants/
│       └── api_constants.dart      # Base URL, endpoints
├── features/
│   └── users/
│       ├── data/
│       │   ├── models/
│       │   │   └── user_model.dart
│       │   └── repositories/
│       │       └── user_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── user.dart
│       │   └── repositories/
│       │       └── user_repository.dart
│       └── presentation/
│           ├── bloc/
│           │   ├── user_cubit.dart
│           │   └── user_state.dart
│           ├── pages/
│           │   └── user_page.dart
│           └── widgets/
│               └── user_card.dart
└── main.dart
```

---

**Happy Coding! 🚀**

*GDG on Campus — Study Jam Flutter*