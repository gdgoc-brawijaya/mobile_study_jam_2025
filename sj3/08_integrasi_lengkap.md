# Integrasi Lengkap: Dio + Cubit + SharedPreferences

## Skenario: Aplikasi User List dengan Cache & Auth

Kita akan membangun fitur lengkap dengan alur:

```
App dibuka
    │
    ▼
SplashScreen: Cek token di SharedPrefs
    │
    ├── [Token ada] ──────────► HomePage (UserList)
    │                               │
    │                               ▼
    │                         UserCubit.fetchUsers()
    │                               │
    │                         [Cache ada?]
    │                         ├── Ya  → tampil dari cache
    │                         └── Tidak → fetch dari Dio → simpan cache
    │
    └── [Token tidak ada] ──► LoginPage
```

---

## Repository dengan Cache Strategy

Ini adalah kunci integrasi: **Repository** menjadi penghubung antara remote (Dio) dan local (SharedPrefs).

Update `lib/features/users/data/repositories/user_repository_impl.dart`:

```dart
import 'dart:convert';
import 'package:shared_preferences/shared_preferences.dart';
import '../../domain/entities/user.dart';
import '../../domain/repositories/user_repository.dart';
import '../datasources/user_remote_datasource.dart';
import '../models/user_model.dart';

class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;

  static const String _cacheKey = 'CACHED_USERS';
  static const Duration _cacheDuration = Duration(minutes: 10);

  UserRepositoryImpl({required this.remoteDataSource});

  @override
  Future<List<User>> getUsers() async {
    // 1. Coba ambil dari cache dulu
    final cachedUsers = await _getCachedUsers();
    if (cachedUsers != null) {
      print('Data dari cache');
      return cachedUsers;
    }

    // 2. Jika tidak ada cache, fetch dari API
    print('Fetch dari API...');
    final userModels = await remoteDataSource.getUsers();
    final users = userModels.map((m) => m.toEntity()).toList();

    // 3. Simpan ke cache untuk request berikutnya
    await _cacheUsers(userModels);

    return users;
  }

  // ── Cache Helpers ──────────────────────────────────────

  Future<List<User>?> _getCachedUsers() async {
    final prefs = await SharedPreferences.getInstance();

    // Cek apakah cache masih valid (belum kadaluarsa)
    final cacheTimestamp = prefs.getInt('${_cacheKey}_timestamp');
    if (cacheTimestamp != null) {
      final cacheTime = DateTime.fromMillisecondsSinceEpoch(cacheTimestamp);
      final now = DateTime.now();
      if (now.difference(cacheTime) > _cacheDuration) {
        print('Cache sudah kadaluarsa, hapus cache...');
        await _clearCache();
        return null;
      }
    }

    final cachedJson = prefs.getString(_cacheKey);
    if (cachedJson == null) return null;

    try {
      final List<dynamic> jsonList = jsonDecode(cachedJson);
      return jsonList
          .map((json) => UserModel.fromJson(json).toEntity())
          .toList();
    } catch (_) {
      await _clearCache();
      return null;
    }
  }

  Future<void> _cacheUsers(List<UserModel> models) async {
    final prefs = await SharedPreferences.getInstance();
    final jsonList = models.map((m) => m.toJson()).toList();
    await prefs.setString(_cacheKey, jsonEncode(jsonList));
    await prefs.setInt(
      '${_cacheKey}_timestamp',
      DateTime.now().millisecondsSinceEpoch,
    );
    print('Data berhasil di-cache');
  }

  Future<void> _clearCache() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(_cacheKey);
    await prefs.remove('${_cacheKey}_timestamp');
  }

  /// Paksa refresh dari API (hapus cache dulu)
  @override
  Future<List<User>> refreshUsers() async {
    await _clearCache();
    return getUsers();
  }

  @override
  Future<User> getUserById(int id) async {
    final userModel = await remoteDataSource.getUserById(id);
    return userModel.toEntity();
  }
}
```

Update domain interface `user_repository.dart`:

```dart
abstract class UserRepository {
  Future<List<User>> getUsers();
  Future<List<User>> refreshUsers(); // ← tambahkan ini
  Future<User> getUserById(int id);
}
```

---

## Update UserCubit dengan Refresh Support

```dart
class UserCubit extends Cubit<UserState> {
  final UserRepository _repository;

  UserCubit({required UserRepository repository})
      : _repository = repository,
        super(const UserState.initial());

  Future<void> fetchUsers() async {
    emit(const UserState.loading());
    try {
      final users = await _repository.getUsers();
      emit(UserState.loaded(users));
    } catch (e) {
      emit(UserState.error(e.toString()));
    }
  }

  // Paksa refresh dari server, bypass cache
  Future<void> refreshUsers() async {
    emit(const UserState.loading());
    try {
      final users = await _repository.refreshUsers();
      emit(UserState.loaded(users));
    } catch (e) {
      emit(UserState.error(e.toString()));
    }
  }
}
```

---

## SplashPage dengan Auth Check

Buat `lib/features/auth/presentation/pages/splash_page.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../../core/services/local_storage_service.dart';

class SplashPage extends StatefulWidget {
  const SplashPage({super.key});

  @override
  State<SplashPage> createState() => _SplashPageState();
}

class _SplashPageState extends State<SplashPage> {
  @override
  void initState() {
    super.initState();
    _checkAuth();
  }

  Future<void> _checkAuth() async {
    // Tampilkan splash screen setidaknya 2 detik
    await Future.delayed(const Duration(seconds: 2));

    if (!mounted) return;

    final token = await LocalStorageService.getToken();

    if (token != null && token.isNotEmpty) {
      // Ada token → ke halaman utama
      Navigator.of(context).pushReplacementNamed('/home');
    } else {
      // Tidak ada token → ke halaman login
      Navigator.of(context).pushReplacementNamed('/login');
    }
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FlutterLogo(size: 100),
            SizedBox(height: 24),
            CircularProgressIndicator(),
            SizedBox(height: 16),
            Text('Memuat...', style: TextStyle(fontSize: 16)),
          ],
        ),
      ),
    );
  }
}
```

---

## main.dart Final

```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'core/di/injection.dart';
import 'features/auth/presentation/pages/splash_page.dart';
import 'features/users/presentation/pages/user_page.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load(fileName: ".env");
  setupInjection();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Data Arch',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      initialRoute: '/',
      routes: {
        '/': (_) => const SplashPage(),
        '/home': (_) => const UserPage(),
        // '/login': (_) => const LoginPage(),
      },
    );
  }
}
```

---

## Alur Data Lengkap (Ringkasan)

```
UserPage
  └─► BlocProvider → UserCubit (via GetIt)
         │
         ▼
      fetchUsers()
         │
         ▼
    UserRepository
      ├─► [Cache valid?] ──Yes──► SharedPreferences.getString()
      │                                │
      │                                ▼
      │                           return List<User>
      │
      └─► [No cache] ────────► DioClient.get('/users')
                                       │
                                       ▼
                                UserModel.fromJson()
                                       │
                                       ▼
                               SharedPreferences.setString() (cache)
                                       │
                                       ▼
                                 return List<User>
                                       │
                                       ▼
                         emit(UserState.loaded(users))
                                       │
                                       ▼
                          BlocBuilder rebuild UI ✅
```

---

**Selanjutnya: [09_error_handling.md](09_error_handling.md)**