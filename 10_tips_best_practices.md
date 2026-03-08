# Tips, Best Practices & Checklist Final

## 1. Jangan Lakukan Ini di Widget

```dart
// ❌ JANGAN: Logic bisnis di dalam build()
class UserPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // ❌ Fetch langsung di build — dipanggil setiap rebuild!
    final dio = Dio();
    dio.get('/users').then((res) => print(res.data));
    
    return Scaffold(/* ... */);
  }
}

// ✅ LAKUKAN: Delegate ke Cubit
class UserPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => sl<UserCubit>()..fetchUsers(), // ← Di sini tempatnya
      child: Scaffold(/* ... */),
    );
  }
}
```

---

## 2. Hindari Magic Strings

```dart
// ❌ JANGAN
prefs.setString('access_token', token);
prefs.getString('acces_token'); // typo → null!

// ✅ LAKUKAN: Konstanta
class StorageKeys {
  static const accessToken = 'ACCESS_TOKEN';
  static const username = 'USERNAME';
}

prefs.setString(StorageKeys.accessToken, token);
prefs.getString(StorageKeys.accessToken); // Aman!
```

---

## 3. Selalu Tutup Cubit yang Tidak Digunakan

Cubit tidak ditutup secara otomatis jika kamu tidak menggunakan `BlocProvider`.

```dart
// ✅ Gunakan BlocProvider — otomatis memanggil cubit.close() saat widget dispose
BlocProvider(
  create: (_) => sl<UserCubit>(),
  child: UserPage(),
)

// ❌ Jika buat manual, wajib close sendiri
class _MyPageState extends State<MyPage> {
  late UserCubit _cubit;

  @override
  void initState() {
    super.initState();
    _cubit = sl<UserCubit>();
  }

  @override
  void dispose() {
    _cubit.close(); // ← Jangan lupa!
    super.dispose();
  }
}
```

---

## 4. Gunakan `Equatable` untuk State

Tanpa `Equatable`, state yang sama bisa memicu rebuild yang tidak perlu.

```dart
// ❌ Tanpa Equatable
class UserLoaded extends UserState {
  final List<User> users;
  UserLoaded(this.users);
  // BlocBuilder akan rebuild meski data sama!
}

// ✅ Dengan Freezed (sudah include equality)
@freezed
class UserState with _$UserState {
  const factory UserState.loaded(List<User> users) = _Loaded;
  // Otomatis equality ✅
}
```

---

## 5. Gunakan `buildWhen` untuk Optimasi Rebuild

```dart
BlocBuilder<UserCubit, UserState>(
  // Hanya rebuild jika state berubah dari/ke loaded
  buildWhen: (previous, current) {
    return current.maybeWhen(
      loaded: (_) => true,
      orElse: () => false,
    );
  },
  builder: (context, state) {
    // Widget ini tidak akan rebuild saat loading atau error
    return UserListWidget(/* ... */);
  },
),
```

---

## 6. Repository Pattern: Satu Sumber Kebenaran

```dart
// ✅ Baik: UI hanya tahu Repository, tidak peduli dari mana data berasal
class UserCubit extends Cubit<UserState> {
  final UserRepository _repository; // ← Abstraction
  
  Future<void> fetchUsers() async {
    // Tidak peduli apakah data dari API atau cache
    final users = await _repository.getUsers();
    emit(UserState.loaded(users));
  }
}
```

---

## 7. Environment-Based Configuration

```dart
// ✅ Jangan hardcode URL
// ❌
const baseUrl = 'https://api.production.com';

// ✅
static String get baseUrl => dotenv.env['BASE_URL'] ?? '';
```

---

## 8. Penamaan yang Konsisten

```
Feature: users

File naming:
├── user.dart            (entity)
├── user_model.dart      (model)
├── user_repository.dart (interface)
├── user_repository_impl.dart (implementation)
├── user_cubit.dart      (cubit)
├── user_state.dart      (state)
├── user_page.dart       (page)
└── user_card.dart       (widget)
```

---

## Roadmap Belajar Selanjutnya

Setelah materi ini, kamu bisa lanjut ke:

```
Sekarang kamu di sini:
  Dio + Bloc/Cubit + SharedPrefs

```

---

## Referensi & Sumber Belajar

| Topik | Link |
|---|---|
| Dio Documentation | https://pub.dev/packages/dio |
| flutter_bloc | https://bloclibrary.dev |
| Freezed | https://pub.dev/packages/freezed |
| GetIt | https://pub.dev/packages/get_it |
| shared_preferences | https://pub.dev/packages/shared_preferences |
| Flutter Architecture Samples | https://fluttersamples.com |

---

## Selamat!

Kamu telah menyelesaikan materi **Flutter Data Architecture: Fetch, Manage, Persist**.

Kamu sekarang tahu cara membangun aplikasi Flutter dengan arsitektur yang:
- **Terstruktur**: Setiap layer punya tanggung jawab yang jelas
- **Skalabel**: Mudah ditambah fitur baru
- **Testable**: Bisa di-test per layer
- **Maintainable**: Mudah dipelihara jangka panjang

---

*GDG on Campus — Study Jam Flutter*