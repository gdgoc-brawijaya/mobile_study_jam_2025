# 3️⃣ Menghubungkan Flutter ke Firebase (FlutterFire CLI)

## Apa itu FlutterFire CLI?

**FlutterFire CLI** adalah tool resmi dari tim Flutter/Firebase yang mengotomatisasi proses:
- Menginisialisasi Firebase di project Flutter
- Membuat file `firebase_options.dart` secara otomatis
- Mengkonfigurasi `google-services.json` dan `GoogleService-Info.plist`

Tanpanya, kamu harus mengkonfigurasi semua file secara manual.

---

## Step 1: Install Dependencies

### 1.1 Install Firebase CLI (via npm)
```bash
npm install -g firebase-tools
```

Verifikasi:
```bash
firebase --version
# Output: 13.x.x
```

### 1.2 Install FlutterFire CLI
```bash
dart pub global activate flutterfire_cli
```

Pastikan PATH dart pub global sudah ditambahkan:
- **Windows:** `%APPDATA%\Pub\Cache\bin` sudah di PATH
- **macOS/Linux:** `$HOME/.pub-cache/bin` sudah di PATH

Verifikasi:
```bash
flutterfire --version
# Output: FlutterFire CLI 1.x.x
```

---

## Step 2: Login ke Firebase

```bash
firebase login
```

Browser akan terbuka → login dengan akun Google yang sama dengan Firebase Console → klik **Allow**.

Verifikasi login berhasil:
```bash
firebase projects:list
# Output: list semua project Firebase kamu
```

---

## Step 3: Buat Project Flutter

```bash
flutter create flutter_firebase_app
cd flutter_firebase_app
```

> Jika sudah punya project, skip step ini.

---

## Step 4: Jalankan FlutterFire Configure

Di dalam folder root project Flutter:

```bash
flutterfire configure
```

### Proses interaktif:
```
i Found 2 Firebase projects.
? Select a Firebase project to configure your Flutter application with ›
❯ flutter-study-jam (flutter-study-jam-a1b2c)
  other-project (other-project-xyz)

? Which platforms should your configuration support (use arrow keys & space to select) ›
✔ android
✔ ios
  web
  macos
  windows
  linux

✔  Firebase configuration file lib/firebase_options.dart generated successfully
✔  Firebase android app com.example.flutter_firebase_app is not registered on Firebase for the "android" platform.
   Registered a new Firebase android app on Firebase.
✔  Generated android/app/google-services.json file.
```

Setelah selesai, file-file berikut akan otomatis dibuat/dimodifikasi:
- `lib/firebase_options.dart` ← file konfigurasi utama
- `android/app/google-services.json` ← konfigurasi Android
- `ios/Runner/GoogleService-Info.plist` ← konfigurasi iOS

---

## Step 5: Tambahkan Package Firebase ke pubspec.yaml

Buka `pubspec.yaml` dan tambahkan:

```yaml
dependencies:
  flutter:
    sdk: flutter

  # Firebase
  firebase_core: ^3.6.0
  cloud_firestore: ^5.4.4
  firebase_auth: ^5.3.1

  # State Management
  flutter_bloc: ^8.1.5
  equatable: ^2.0.5

  # Dependency Injection
  get_it: ^7.7.0

  # Model (opsional tapi recommended)
  freezed_annotation: ^2.4.1
  json_annotation: ^4.9.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.9
  freezed: ^2.5.2
  json_serializable: ^6.8.0
```

```bash
flutter pub get
```

---

## Step 6: Inisialisasi Firebase di main.dart

Buka `lib/main.dart` dan modifikasi:

```dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'firebase_options.dart';

void main() async {
  // Wajib: pastikan binding Flutter sudah siap sebelum Firebase init
  WidgetsFlutterBinding.ensureInitialized();

  // Inisialisasi Firebase
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Firebase App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const Scaffold(
        body: Center(
          child: Text('Firebase Connected! 🎉'),
        ),
      ),
    );
  }
}
```

Jalankan aplikasi:
```bash
flutter run
```

Jika berhasil tanpa error di console, Firebase sudah terhubung!

---

## Step 7: Verifikasi Koneksi Firebase

Tambahkan tes sederhana untuk memastikan Firestore bisa diakses:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

// Coba baca dari Firestore (di initState atau tombol)
Future<void> testFirestoreConnection() async {
  try {
    final firestore = FirebaseFirestore.instance;
    
    // Coba tulis dokumen tes
    await firestore.collection('_test').add({
      'message': 'Hello Firebase!',
      'timestamp': FieldValue.serverTimestamp(),
    });
    
    print('✅ Firestore connected successfully!');
  } catch (e) {
    print('❌ Firestore error: $e');
  }
}
```

Cek di Firebase Console → Firestore → Collection `_test` apakah dokumen muncul.

---

## Konfigurasi Tambahan Android

Pastikan `android/build.gradle` (level project) memiliki:
```groovy
buildscript {
    dependencies {
        // ...
        classpath 'com.google.gms:google-services:4.4.2'  // ← pastikan ada
    }
}
```

Dan `android/app/build.gradle` (level app):
```groovy
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'  // ← pastikan ada
}

android {
    compileSdkVersion 34
    
    defaultConfig {
        minSdkVersion 21    // ← Firebase butuh minimal SDK 21
        // ...
    }
}
```

> FlutterFire CLI biasanya sudah mengatur ini otomatis. Cek jika ada build error.

---

## Memahami `firebase_options.dart`

File ini berisi konfigurasi per platform yang di-generate otomatis:

```dart
// lib/firebase_options.dart (auto-generated, jangan edit manual)
class DefaultFirebaseOptions {
  static FirebaseOptions get currentPlatform {
    if (kIsWeb) return web;
    switch (defaultTargetPlatform) {
      case TargetPlatform.android:
        return android;
      case TargetPlatform.iOS:
        return ios;
      // ...
    }
  }

  static const FirebaseOptions android = FirebaseOptions(
    apiKey: 'AIza...',         // API Key
    appId: '1:123:android:abc', // App ID
    messagingSenderId: '123',
    projectId: 'flutter-study-jam-a1b2c',
    storageBucket: 'flutter-study-jam-a1b2c.appspot.com',
  );
  
  // ...
}
```

> ⚠️ Meskipun `firebase_options.dart` berisi API keys, file ini **aman untuk di-commit** ke Git karena Firebase menggunakan Security Rules sebagai lapisan keamanan utama. Namun tetap **jangan commit** `google-services.json` langsung.

---

## Troubleshooting Umum

| Error | Solusi |
|---|---|
| `firebase: command not found` | Install Firebase CLI atau tambahkan ke PATH |
| `flutterfire: command not found` | Tambahkan `~/.pub-cache/bin` ke PATH |
| `minSdkVersion` error | Set `minSdkVersion 21` di `android/app/build.gradle` |
| `google-services.json not found` | Pastikan file ada di `android/app/` bukan di root |
| `FirebaseException: permission-denied` | Pastikan Firestore rules sudah di-set ke test mode |
| Build error Kotlin | Update `kotlin_version = "1.9.0"` di `android/build.gradle` |

---

## Checklist Step 3

- [ ] Firebase CLI terinstall (`firebase --version`)
- [ ] FlutterFire CLI terinstall (`flutterfire --version`)
- [ ] `firebase login` berhasil
- [ ] `flutterfire configure` berhasil dijalankan
- [ ] `firebase_options.dart` sudah terbuat
- [ ] Package Firebase sudah ditambahkan ke `pubspec.yaml`
- [ ] `Firebase.initializeApp()` sudah ada di `main.dart`
- [ ] App berjalan tanpa error

---

*Lanjut ke [04_firestore_model.md](04_firestore_model.md) →*
