# Setup Project & Struktur Folder

## Langkah 1: Buat Project Flutter

```bash
flutter create flutter_data_arch
cd flutter_data_arch
```

---

## Langkah 2: Install Dependencies

Tambahkan semua package ke `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter

  # Networking
  dio: ^5.4.3+1

  # State Management
  flutter_bloc: ^8.1.5
  equatable: ^2.0.5        # Supaya state bisa dibandingkan (==)

  # Local Storage
  shared_preferences: ^2.2.3

  # Dependency Injection
  get_it: ^7.7.0

  # Data Model (code generation)
  freezed_annotation: ^2.4.1
  json_annotation: ^4.9.0

  # Env / Config
  flutter_dotenv: ^5.1.0

dev_dependencies:
  flutter_test:
    sdk: flutter

  # Code generation
  build_runner: ^2.4.9
  freezed: ^2.5.2
  json_serializable: ^6.8.0
```

Jalankan:

```bash
flutter pub get
```

---

## Langkah 3: Setup File `.env`

Buat file `.env` di **root project** (sejajar `pubspec.yaml`):

```env
BASE_URL=https://jsonplaceholder.typicode.com
API_KEY=your_api_key_here
```

> **Jangan lupa** tambahkan `.env` ke `.gitignore` agar API Key tidak bocor!

```bash
# .gitignore
.env
```

Daftarkan file `.env` di `pubspec.yaml`:

```yaml
flutter:
  assets:
    - .env
```

Load di `main.dart`:

```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load(fileName: ".env");
  runApp(const MyApp());
}
```

---

## Langkah 4: Buat Struktur Folder

```bash
# Buat folder sekaligus dari terminal
mkdir -p lib/core/network/interceptors
mkdir -p lib/core/di
mkdir -p lib/core/error
mkdir -p lib/core/constants
mkdir -p lib/features/users/data/models
mkdir -p lib/features/users/data/repositories
mkdir -p lib/features/users/domain/entities
mkdir -p lib/features/users/domain/repositories
mkdir -p lib/features/users/presentation/bloc
mkdir -p lib/features/users/presentation/pages
mkdir -p lib/features/users/presentation/widgets
```

Struktur lengkap yang kita tuju:

```
lib/
├── core/
│   ├── constants/
│   │   └── api_constants.dart      ← Base URL, endpoint path
│   ├── di/
│   │   └── injection.dart          ← Registrasi GetIt
│   ├── error/
│   │   └── failures.dart           ← Custom error/failure class
│   └── network/
│       ├── dio_client.dart          ← Konfigurasi Dio
│       └── interceptors/
│           ├── auth_interceptor.dart
│           └── log_interceptor.dart
├── features/
│   └── users/
│       ├── data/
│       │   ├── models/
│       │   │   ├── user_model.dart
│       │   │   └── user_model.freezed.dart  ← (generated)
│       │   └── repositories/
│       │       └── user_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── user.dart
│       │   └── repositories/
│       │       └── user_repository.dart     ← abstract class (interface)
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

## Langkah 5: Konfigurasi API Constants

Buat `lib/core/constants/api_constants.dart`:

```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

class ApiConstants {
  // Ambil dari .env file
  static String get baseUrl => dotenv.env['BASE_URL'] ?? '';
  static String get apiKey => dotenv.env['API_KEY'] ?? '';

  // Endpoints
  static const String usersEndpoint = '/users';
  static const String postsEndpoint = '/posts';
}
```

---

## Langkah 6: Setup main.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'core/di/injection.dart';
import 'features/users/presentation/pages/user_page.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Load environment variables
  await dotenv.load(fileName: ".env");
  
  // Setup dependency injection
  setupInjection();
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Data Arch',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const UserPage(),
    );
  }
}
```

---

## Checklist Setup

- [ ] Project dibuat dengan `flutter create`
- [ ] Semua package ditambahkan dan `flutter pub get` dijalankan
- [ ] File `.env` dibuat dan didaftarkan di `pubspec.yaml`
- [ ] File `.env` ditambahkan ke `.gitignore`
- [ ] Struktur folder dibuat
- [ ] `api_constants.dart` dibuat
- [ ] `main.dart` diupdate

---

**Selanjutnya: [03_fetch_dio.md](03_fetch_dio.md)**