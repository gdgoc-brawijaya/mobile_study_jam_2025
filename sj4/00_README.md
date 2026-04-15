# Flutter Cloud Architecture: Connect & Sync with Firebase

## Study Jam Materials — GDG on Campus Brawijaya

Selamat datang di **Study Jam 4**! 🚀☁️

Pada modul ini, kita akan belajar bagaimana aplikasi Flutter **terhubung ke cloud**, **menyimpan data secara real-time**, dan **sinkronisasi antar perangkat** menggunakan Firebase — platform Backend-as-a-Service (BaaS) dari Google.

---

## Tech Stack

| Layer | Package / Service | Fungsi |
|---|---|---|
| **Cloud Database** | `cloud_firestore` | NoSQL real-time database |
| **Authentication** | `firebase_auth` | Login / Register user |
| **Firebase Core** | `firebase_core` | Inisialisasi Firebase di Flutter |
| **State Management** | `flutter_bloc` | Mengelola state Firestore |
| **DI** | `get_it` | Dependency Injection |
| **Model** | `freezed` + `json_serializable` | Data class & serialisasi |

---

## Daftar Materi

| No | File | Topik |
|---|---|---|
| 1 | [`01_arsitektur_cloud.md`](01_arsitektur_cloud.md) | Konsep Cloud Architecture & Firebase Ecosystem |
| 2 | [`02_setup_firebase.md`](02_setup_firebase.md) | Membuat project di Firebase Console |
| 3 | [`03_flutter_firebase_connect.md`](03_flutter_firebase_connect.md) | Menghubungkan Flutter ke Firebase (FlutterFire CLI) |
| 4 | [`04_firestore_model.md`](04_firestore_model.md) | Struktur Data Firestore & Model Flutter |
| 5 | [`05_firestore_crud.md`](05_firestore_crud.md) | CRUD Operasi dengan Cloud Firestore |
| 6 | [`06_realtime_stream.md`](06_realtime_stream.md) | Real-time Updates dengan Stream & BlocBuilder |
| 7 | [`07_firebase_auth.md`](07_firebase_auth.md) | Autentikasi User dengan Firebase Auth |
| 8 | [`08_error_handling.md`](08_error_handling.md) | Error Handling & Offline Support |
| 9 | [`09_tips_best_practices.md`](09_tips_best_practices.md) | Tips, Security Rules & Best Practices |

---

## Target Pembelajaran

Setelah menyelesaikan modul ini, peserta diharapkan:

1. Memahami konsep **Cloud Architecture** dan posisi Firebase di dalamnya.
2. Bisa membuat dan mengkonfigurasi **Firebase Project** dari Firebase Console.
3. Menghubungkan project Flutter ke Firebase menggunakan **FlutterFire CLI**.
4. Memahami struktur data **Collection & Document** di Firestore.
5. Melakukan operasi **Create, Read, Update, Delete (CRUD)** ke Firestore.
6. Menampilkan data **real-time** menggunakan Firestore Stream & `StreamBuilder`.
7. Implementasi **Firebase Authentication** (email/password).
8. Menerapkan **Firestore Security Rules** yang aman.

---

## Prerequisites

Sebelum memulai, pastikan kamu sudah:

- ✅ Menyelesaikan materi Study Jam 3 (atau familiar dengan Bloc/Cubit)
- ✅ Flutter SDK terpasang (`flutter --version`)
- ✅ Akun Google aktif (untuk Firebase Console)
- ✅ Node.js terpasang (untuk FlutterFire CLI)

---

## Quick Start

```bash
# Buat project Flutter baru
flutter create flutter_firebase_app
cd flutter_firebase_app

# Install FlutterFire CLI (butuh Node.js)
dart pub global activate flutterfire_cli

# Login ke Firebase
firebase login
```

---

## Struktur Folder Project

```
flutter_firebase_app/
├── lib/
│   ├── core/
│   │   ├── di/
│   │   │   └── injection.dart          # GetIt setup
│   │   └── errors/
│   │       └── firebase_exception.dart
│   ├── data/
│   │   ├── models/
│   │   │   └── task_model.dart         # Freezed model
│   │   └── repositories/
│   │       └── task_repository.dart    # Abstraksi Firestore
│   ├── presentation/
│   │   ├── bloc/
│   │   │   └── task_cubit.dart         # State management
│   │   └── pages/
│   │       ├── home_page.dart
│   │       ├── add_task_page.dart
│   │       └── login_page.dart
│   ├── firebase_options.dart           # Auto-generated oleh FlutterFire CLI
│   └── main.dart
├── firebase.json
└── pubspec.yaml
```

---

*GDG on Campus Brawijaya — Tech Series Study Jam 2025*
