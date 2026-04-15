# 7️⃣ Autentikasi User dengan Firebase Auth

## Mengapa Firebase Auth?

Membangun sistem login dari nol membutuhkan:
- Database user
- Hashing password
- Session management
- Token refresh
- Email verification
- Password reset

Firebase Auth menyediakan semua ini dalam beberapa baris kode.

---

## Konsep: Auth State

```
App Start
    │
    ▼
FirebaseAuth.instance.authStateChanges()
    │
    ├── user == null  →  Tampilkan LoginPage
    │
    └── user != null  →  Tampilkan HomePage (dengan user.uid)
```

`authStateChanges()` adalah stream yang otomatis emit setiap kali status login berubah.

---

## Auth Repository

Buat `lib/data/repositories/auth_repository.dart`:

```dart
import 'package:firebase_auth/firebase_auth.dart';

class AuthRepository {
  final FirebaseAuth _auth;

  AuthRepository({FirebaseAuth? auth})
      : _auth = auth ?? FirebaseAuth.instance;

  // Stream perubahan status auth (login/logout)
  Stream<User?> get authStateChanges => _auth.authStateChanges();

  // User saat ini (null jika belum login)
  User? get currentUser => _auth.currentUser;

  // Register dengan email & password
  Future<UserCredential> register({
    required String email,
    required String password,
  }) async {
    return await _auth.createUserWithEmailAndPassword(
      email: email,
      password: password,
    );
  }

  // Login dengan email & password
  Future<UserCredential> login({
    required String email,
    required String password,
  }) async {
    return await _auth.signInWithEmailAndPassword(
      email: email,
      password: password,
    );
  }

  // Logout
  Future<void> logout() async {
    await _auth.signOut();
  }

  // Reset password via email
  Future<void> resetPassword(String email) async {
    await _auth.sendPasswordResetEmail(email: email);
  }

  // Update display name
  Future<void> updateDisplayName(String name) async {
    await _auth.currentUser?.updateDisplayName(name);
  }
}
```

---

## Auth Cubit

Buat `lib/presentation/bloc/auth_cubit.dart`:

```dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';
import '../../data/repositories/auth_repository.dart';

// ── State ──────────────────────────────────────────────────────
abstract class AuthState extends Equatable {
  @override
  List<Object?> get props => [];
}

class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}

class AuthAuthenticated extends AuthState {
  final User user;
  const AuthAuthenticated(this.user);

  @override
  List<Object?> get props => [user];
}

class AuthUnauthenticated extends AuthState {}

class AuthError extends AuthState {
  final String message;
  const AuthError(this.message);

  @override
  List<Object?> get props => [message];
}

// ── Cubit ──────────────────────────────────────────────────────
class AuthCubit extends Cubit<AuthState> {
  final AuthRepository _repository;

  AuthCubit(this._repository) : super(AuthInitial()) {
    // Otomatis cek apakah user sudah login sebelumnya
    _checkAuthState();
  }

  void _checkAuthState() {
    _repository.authStateChanges.listen((user) {
      if (user != null) {
        emit(AuthAuthenticated(user));
      } else {
        emit(AuthUnauthenticated());
      }
    });
  }

  Future<void> register({
    required String email,
    required String password,
  }) async {
    emit(AuthLoading());
    try {
      final credential = await _repository.register(
        email: email,
        password: password,
      );
      emit(AuthAuthenticated(credential.user!));
    } on FirebaseAuthException catch (e) {
      emit(AuthError(_mapFirebaseError(e.code)));
    }
  }

  Future<void> login({
    required String email,
    required String password,
  }) async {
    emit(AuthLoading());
    try {
      final credential = await _repository.login(
        email: email,
        password: password,
      );
      emit(AuthAuthenticated(credential.user!));
    } on FirebaseAuthException catch (e) {
      emit(AuthError(_mapFirebaseError(e.code)));
    }
  }

  Future<void> logout() async {
    await _repository.logout();
    emit(AuthUnauthenticated());
  }

  // Mapping error code Firebase ke pesan yang user-friendly
  String _mapFirebaseError(String code) {
    switch (code) {
      case 'user-not-found':
        return 'Email tidak terdaftar.';
      case 'wrong-password':
        return 'Password salah.';
      case 'email-already-in-use':
        return 'Email sudah digunakan akun lain.';
      case 'invalid-email':
        return 'Format email tidak valid.';
      case 'weak-password':
        return 'Password terlalu lemah (min. 6 karakter).';
      case 'too-many-requests':
        return 'Terlalu banyak percobaan. Coba lagi nanti.';
      case 'network-request-failed':
        return 'Tidak ada koneksi internet.';
      default:
        return 'Terjadi kesalahan. Kode: $code';
    }
  }
}
```

---

## Halaman Login

Buat `lib/presentation/pages/login_page.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/auth_cubit.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isRegisterMode = false;
  bool _obscurePassword = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _submit() {
    if (!_formKey.currentState!.validate()) return;

    final email = _emailController.text.trim();
    final password = _passwordController.text;

    if (_isRegisterMode) {
      context.read<AuthCubit>().register(email: email, password: password);
    } else {
      context.read<AuthCubit>().login(email: email, password: password);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: BlocConsumer<AuthCubit, AuthState>(
        listener: (context, state) {
          if (state is AuthError) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text(state.message),
                backgroundColor: Colors.red,
              ),
            );
          }
        },
        builder: (context, state) {
          final isLoading = state is AuthLoading;

          return SafeArea(
            child: Padding(
              padding: const EdgeInsets.all(24),
              child: Form(
                key: _formKey,
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  crossAxisAlignment: CrossAxisAlignment.stretch,
                  children: [
                    // Logo / Icon
                    const Icon(
                      Icons.cloud_done,
                      size: 80,
                      color: Colors.deepPurple,
                    ),
                    const SizedBox(height: 8),
                    Text(
                      _isRegisterMode ? 'Daftar Akun' : 'Selamat Datang',
                      textAlign: TextAlign.center,
                      style: Theme.of(context).textTheme.headlineMedium,
                    ),
                    const SizedBox(height: 32),

                    // Email
                    TextFormField(
                      controller: _emailController,
                      keyboardType: TextInputType.emailAddress,
                      decoration: const InputDecoration(
                        labelText: 'Email',
                        prefixIcon: Icon(Icons.email_outlined),
                        border: OutlineInputBorder(),
                      ),
                      validator: (v) {
                        if (v == null || v.isEmpty) return 'Email wajib diisi';
                        if (!v.contains('@')) return 'Format email tidak valid';
                        return null;
                      },
                    ),
                    const SizedBox(height: 16),

                    // Password
                    TextFormField(
                      controller: _passwordController,
                      obscureText: _obscurePassword,
                      decoration: InputDecoration(
                        labelText: 'Password',
                        prefixIcon: const Icon(Icons.lock_outline),
                        border: const OutlineInputBorder(),
                        suffixIcon: IconButton(
                          icon: Icon(_obscurePassword
                              ? Icons.visibility_off
                              : Icons.visibility),
                          onPressed: () => setState(
                              () => _obscurePassword = !_obscurePassword),
                        ),
                      ),
                      validator: (v) {
                        if (v == null || v.isEmpty) return 'Password wajib diisi';
                        if (v.length < 6) return 'Password min. 6 karakter';
                        return null;
                      },
                    ),
                    const SizedBox(height: 24),

                    // Submit button
                    ElevatedButton(
                      onPressed: isLoading ? null : _submit,
                      style: ElevatedButton.styleFrom(
                        padding: const EdgeInsets.symmetric(vertical: 14),
                      ),
                      child: isLoading
                          ? const SizedBox(
                              height: 20,
                              width: 20,
                              child: CircularProgressIndicator(strokeWidth: 2),
                            )
                          : Text(_isRegisterMode ? 'Daftar' : 'Masuk'),
                    ),
                    const SizedBox(height: 12),

                    // Toggle login/register
                    TextButton(
                      onPressed: () =>
                          setState(() => _isRegisterMode = !_isRegisterMode),
                      child: Text(
                        _isRegisterMode
                            ? 'Sudah punya akun? Masuk'
                            : 'Belum punya akun? Daftar',
                      ),
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

---

## Routing Berdasarkan Auth State

Update `main.dart` untuk menampilkan halaman yang tepat:

```dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'firebase_options.dart';
import 'core/di/injection.dart';
import 'presentation/bloc/auth_cubit.dart';
import 'presentation/bloc/task_cubit.dart';
import 'presentation/pages/login_page.dart';
import 'presentation/pages/home_page.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  setupDependencies(); // GetIt setup
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider(create: (_) => getIt<AuthCubit>()),
        BlocProvider(create: (_) => getIt<TaskCubit>()),
      ],
      child: MaterialApp(
        title: 'Flutter Firebase App',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
          useMaterial3: true,
        ),
        home: const AuthWrapper(),
      ),
    );
  }
}

/// Widget yang menentukan tampilan berdasarkan status auth
class AuthWrapper extends StatelessWidget {
  const AuthWrapper({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<AuthCubit, AuthState>(
      builder: (context, state) {
        if (state is AuthAuthenticated) {
          return HomePage(userId: state.user.uid);
        }
        if (state is AuthUnauthenticated) {
          return const LoginPage();
        }
        // Loading / Initial
        return const Scaffold(
          body: Center(child: CircularProgressIndicator()),
        );
      },
    );
  }
}
```

---

## Informasi User yang Tersedia

```dart
final user = FirebaseAuth.instance.currentUser;

if (user != null) {
  print(user.uid);           // ID unik user (dipakai sebagai userId di Firestore)
  print(user.email);         // Email
  print(user.displayName);   // Nama (jika di-set)
  print(user.photoURL);      // URL foto profil
  print(user.emailVerified); // Apakah email sudah diverifikasi
  print(user.metadata.creationTime); // Waktu registrasi
}
```

---

*Lanjut ke [08_error_handling.md](08_error_handling.md) →*
