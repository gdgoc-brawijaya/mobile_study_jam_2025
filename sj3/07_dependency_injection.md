# Dependency Injection dengan GetIt

## Masalah Tanpa DI

Bayangkan membuat Cubit secara manual di setiap halaman:

```dart
// Tanpa DI — verbose, susah di-test, susah di-maintain
final dioClient = DioClient();
final dataSource = UserRemoteDataSourceImpl(dio: dioClient.dio);
final repository = UserRepositoryImpl(remoteDataSource: dataSource);
final cubit = UserCubit(repository: repository);
```

Bayangkan harus menuliskan ini di setiap halaman yang butuh `UserCubit`. 😱

---

## Solusi: GetIt sebagai Service Locator

```dart
// Dengan DI — cukup satu baris
final cubit = sl<UserCubit>();
```

**`sl`** adalah singkatan dari *service locator* — GetIt yang sudah dikonfigurasi.

---

## Konsep DI dengan GetIt

```
Saat App Start:
┌─────────────────────────────────────┐
│         injection.dart              │
│                                     │
│  sl.registerLazySingleton(          │
│    () => DioClient()                │  ← Daftarkan
│  )                                  │
│  sl.registerFactory(                │
│    () => UserCubit(repo: sl())      │  ← Daftarkan
│  )                                  │
└─────────────────────────────────────┘

Saat Dibutuhkan:
┌─────────────────────────────────────┐
│  BlocProvider(                      │
│    create: (_) => sl<UserCubit>()   │  ← Ambil instance
│  )                                  │
└─────────────────────────────────────┘
```

---

## Step 1: Setup injection.dart

Buat `lib/core/di/injection.dart`:

```dart
import 'package:get_it/get_it.dart';
import 'package:dio/dio.dart';

import '../network/dio_client.dart';
import '../../features/users/data/datasources/user_remote_datasource.dart';
import '../../features/users/data/repositories/user_repository_impl.dart';
import '../../features/users/domain/repositories/user_repository.dart';
import '../../features/users/presentation/bloc/user_cubit.dart';

// Global service locator instance
final sl = GetIt.instance;

void setupInjection() {
  // ── CORE ─────────────────────────────────────────────
  
  // DioClient: satu instance untuk seluruh app (Singleton)
  sl.registerLazySingleton<DioClient>(() => DioClient());
  
  // Expose Dio dari DioClient
  sl.registerLazySingleton<Dio>(() => sl<DioClient>().dio);

  // ── DATA SOURCES ──────────────────────────────────────
  
  sl.registerLazySingleton<UserRemoteDataSource>(
    () => UserRemoteDataSourceImpl(dio: sl()),
  );

  // ── REPOSITORIES ─────────────────────────────────────
  
  // Daftarkan sebagai abstract class (interface)
  // sehingga bisa swap implementasi saat testing
  sl.registerLazySingleton<UserRepository>(
    () => UserRepositoryImpl(remoteDataSource: sl()),
  );

  // ── CUBITS / BLOCS ───────────────────────────────────
  
  // Factory: setiap kali dipanggil, buat instance baru
  // (Cubit punya lifecycle, tidak boleh singleton)
  sl.registerFactory<UserCubit>(
    () => UserCubit(repository: sl()),
  );
}
```

---

## Perbedaan Tipe Registrasi

| Tipe | Penjelasan | Cocok untuk |
|---|---|---|
| `registerLazySingleton` | Instance dibuat saat pertama dipanggil, lalu di-cache | Dio, Repository, Services |
| `registerSingleton` | Instance dibuat langsung saat registrasi | Jarang digunakan |
| `registerFactory` | Instance baru setiap kali `sl()` dipanggil | Cubit, Bloc |

```dart
// Singleton: dibuat sekali, dipakai terus
sl.registerLazySingleton(() => DioClient());

// Factory: buat baru setiap kali dipanggil
sl.registerFactory(() => UserCubit(repository: sl()));
```

---

## Step 2: Panggil setupInjection di main.dart

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load(fileName: ".env");
  
  setupInjection(); // ← Wajib dipanggil sebelum runApp!
  
  runApp(const MyApp());
}
```

---

## Step 3: Gunakan di BlocProvider

```dart
// Di halaman manapun, cukup:
BlocProvider(
  create: (_) => sl<UserCubit>()..fetchUsers(),
  child: const UserPage(),
)
```

---

## Bonus: Multi-Feature Setup

Untuk aplikasi besar, pisahkan DI per fitur:

```dart
// lib/core/di/injection.dart
void setupInjection() {
  _setupCore();
  _setupUserFeature();
  _setupAuthFeature();
  _setupPostFeature();
}

void _setupCore() {
  sl.registerLazySingleton<DioClient>(() => DioClient());
  sl.registerLazySingleton<Dio>(() => sl<DioClient>().dio);
}

void _setupUserFeature() {
  sl.registerLazySingleton<UserRemoteDataSource>(
    () => UserRemoteDataSourceImpl(dio: sl()),
  );
  sl.registerLazySingleton<UserRepository>(
    () => UserRepositoryImpl(remoteDataSource: sl()),
  );
  sl.registerFactory<UserCubit>(
    () => UserCubit(repository: sl()),
  );
}

void _setupAuthFeature() {
  // ... registrasi auth dependencies
}
```

---

## Manfaat DI dalam Testing

```dart
// Di test, kamu bisa swap dependency dengan mock!
setUp(() {
  sl.reset(); // Reset GetIt
  
  // Daftarkan mock sebagai pengganti implementasi asli
  sl.registerLazySingleton<UserRepository>(() => MockUserRepository());
  sl.registerFactory<UserCubit>(() => UserCubit(repository: sl()));
});
```

---

**Selanjutnya: [08_integrasi_lengkap.md](08_integrasi_lengkap.md)**