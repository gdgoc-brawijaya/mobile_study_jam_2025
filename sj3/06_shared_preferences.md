# Local Storage dengan Shared Preferences

## Apa itu Shared Preferences?

Shared Preferences menyimpan data sebagai **key-value pairs** di penyimpanan internal perangkat. Data tersimpan bahkan setelah aplikasi ditutup.

```
RAM (Sementara)           Disk (Permanen)
┌──────────────┐         ┌────────────────────┐
│  Cubit State │         │  SharedPreferences  │
│              │         │                     │
│ users: [...]  │  ←──→  │ "token": "abc123"  │
│ isLoading: T │         │ "theme": "dark"     │
│              │         │ "username": "Ali"   │
└──────────────┘         └────────────────────┘
  Hilang saat app close    Tetap ada setelah close
```

---

## Kapan Menggunakan SharedPreferences?

**Cocok untuk:**
- Token autentikasi (access token)
- Preferensi user (tema gelap/terang)
- Status login (`isLoggedIn: true`)
- Data ringan (username, user ID)
- Onboarding sudah dilihat (`hasSeenOnboarding: true`)

**Tidak cocok untuk:**
- Data besar (gunakan SQLite/Hive)
- Data relasional
- Password (gunakan `flutter_secure_storage`)
- List objek kompleks (gunakan Hive/Isar)

---

## Step 1: Buat Service Wrapper

Membungkus SharedPreferences dalam sebuah service class agar lebih mudah digunakan dan di-*test*.

Buat `lib/core/services/local_storage_service.dart`:

```dart
import 'package:shared_preferences/shared_preferences.dart';

class LocalStorageService {
  // Kunci-kunci yang digunakan (hindari magic string!)
  static const String _keyAccessToken = 'ACCESS_TOKEN';
  static const String _keyUsername = 'USERNAME';
  static const String _keyUserId = 'USER_ID';
  static const String _keyTheme = 'APP_THEME';
  static const String _keyHasSeenOnboarding = 'HAS_SEEN_ONBOARDING';

  // ──────────────────────────────────────────
  //  TOKEN
  // ──────────────────────────────────────────

  static Future<void> saveToken(String token) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_keyAccessToken, token);
  }

  static Future<String?> getToken() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_keyAccessToken);
  }

  // ──────────────────────────────────────────
  //  USER DATA
  // ──────────────────────────────────────────

  static Future<void> saveUserData({
    required String username,
    required int userId,
  }) async {
    final prefs = await SharedPreferences.getInstance();
    // Bisa batch save beberapa key sekaligus
    await Future.wait([
      prefs.setString(_keyUsername, username),
      prefs.setInt(_keyUserId, userId),
    ]);
  }

  static Future<String?> getUsername() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_keyUsername);
  }

  static Future<int?> getUserId() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getInt(_keyUserId);
  }

  // ──────────────────────────────────────────
  //  THEME
  // ──────────────────────────────────────────

  static Future<void> saveTheme(String theme) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_keyTheme, theme);
  }

  static Future<String> getTheme() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_keyTheme) ?? 'light'; // Default: light
  }

  // ──────────────────────────────────────────
  //  ONBOARDING
  // ──────────────────────────────────────────

  static Future<void> setOnboardingSeen() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool(_keyHasSeenOnboarding, true);
  }

  static Future<bool> hasSeenOnboarding() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getBool(_keyHasSeenOnboarding) ?? false;
  }

  // ──────────────────────────────────────────
  //  CLEAR (Logout)
  // ──────────────────────────────────────────

  /// Hapus hanya data sesi user (token, username, user ID)
  static Future<void> clearSession() async {
    final prefs = await SharedPreferences.getInstance();
    await Future.wait([
      prefs.remove(_keyAccessToken),
      prefs.remove(_keyUsername),
      prefs.remove(_keyUserId),
    ]);
  }

  /// Hapus semua data (factory reset)
  static Future<void> clearAll() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.clear();
  }
}
```

---

## Step 2: Contoh — Auth Flow dengan SharedPrefs

### Login dan Simpan Token

```dart
class AuthCubit extends Cubit<AuthState> {
  AuthCubit() : super(const AuthState.initial());

  Future<void> login(String email, String password) async {
    emit(const AuthState.loading());
    try {
      // Simulasi request login ke API
      final response = await AuthApiService().login(email, password);

      // Simpan token dan data user ke local storage
      await LocalStorageService.saveToken(response.token);
      await LocalStorageService.saveUserData(
        username: response.user.name,
        userId: response.user.id,
      );

      emit(AuthState.authenticated(response.user));
    } catch (e) {
      emit(AuthState.error(e.toString()));
    }
  }

  Future<void> logout() async {
    // Hapus session dari local storage
    await LocalStorageService.clearSession();
    emit(const AuthState.unauthenticated());
  }
}
```

### Cek Status Login Saat App Dibuka

```dart
class SplashCubit extends Cubit<SplashState> {
  SplashCubit() : super(const SplashState.loading());

  Future<void> checkAuthStatus() async {
    await Future.delayed(const Duration(seconds: 2)); // Splash duration

    // Cek apakah onboarding sudah dilihat
    final hasSeenOnboarding = await LocalStorageService.hasSeenOnboarding();
    if (!hasSeenOnboarding) {
      emit(const SplashState.goToOnboarding());
      return;
    }

    // Cek apakah ada token tersimpan
    final token = await LocalStorageService.getToken();
    if (token != null && token.isNotEmpty) {
      emit(const SplashState.authenticated());
    } else {
      emit(const SplashState.unauthenticated());
    }
  }
}
```

---

## Step 3: Contoh — Theme Switcher

```dart
// theme_cubit.dart
class ThemeCubit extends Cubit<ThemeMode> {
  ThemeCubit() : super(ThemeMode.light);

  Future<void> loadTheme() async {
    final savedTheme = await LocalStorageService.getTheme();
    emit(savedTheme == 'dark' ? ThemeMode.dark : ThemeMode.light);
  }

  Future<void> toggleTheme() async {
    final isDark = state == ThemeMode.dark;
    final newTheme = isDark ? ThemeMode.light : ThemeMode.dark;
    await LocalStorageService.saveTheme(isDark ? 'light' : 'dark');
    emit(newTheme);
  }
}
```

```dart
// Di MaterialApp
BlocBuilder<ThemeCubit, ThemeMode>(
  builder: (context, themeMode) {
    return MaterialApp(
      themeMode: themeMode,
      theme: ThemeData.light(),
      darkTheme: ThemeData.dark(),
      home: const HomePage(),
    );
  },
)
```

---

## Tipe Data yang Didukung SharedPreferences

| Dart Type | Method |
|---|---|
| `String` | `setString` / `getString` |
| `int` | `setInt` / `getInt` |
| `double` | `setDouble` / `getDouble` |
| `bool` | `setBool` / `getBool` |
| `List<String>` | `setStringList` / `getStringList` |

> 🔥 Untuk menyimpan **Map/Object**, konversikan ke JSON String dulu:
> ```dart
> import 'dart:convert';
> final jsonStr = jsonEncode(myMap);
> prefs.setString('my_key', jsonStr);
> 
> // Membaca kembali
> final jsonStr = prefs.getString('my_key');
> final myMap = jsonDecode(jsonStr!);
> ```

---

## ⚠️ Keamanan: Jangan Simpan Data Sensitif!

```
SharedPreferences
      │
      ├── ✅ Access Token (acceptable untuk sebagian besar kasus)
      ├── ✅ Username, User ID
      ├── ✅ Theme preference
      ├── ❌ Password  ← JANGAN!
      └── ❌ Credit card info ← JANGAN!

Untuk data sensitif, gunakan:
flutter_secure_storage → Data terenkripsi di Keychain (iOS) / Keystore (Android)
```

---

**Selanjutnya: [07_dependency_injection.md](07_dependency_injection.md)**