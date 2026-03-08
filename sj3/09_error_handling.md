# Error Handling, Loading State & UX

## Mengapa Error Handling Penting?

```
Tanpa error handling:        Dengan error handling:
┌──────────────────┐         ┌──────────────────────────┐
│                  │         │                          │
│   App crash! 💥  │         │  ⚠️ Tidak ada koneksi.  │
│                  │   vs    │                          │
│                  │         │  [Coba Lagi]             │
│                  │         │                          │
└──────────────────┘         └──────────────────────────┘
    User kabur 😢                 User puas 😊
```

---

## A. Tipe-Tipe Error yang Umum

```
Error saat fetch data
      │
      ├── Network Error
      │   ├── Timeout (connectTimeout / receiveTimeout)
      │   ├── No internet (connectionError)
      │   └── DNS error
      │
      ├── Server Error
      │   ├── 4xx: Client error (400 bad request, 401 unauthorized, 404 not found)
      │   └── 5xx: Server error (500 internal server error)
      │
      └── App Error
          ├── JSON parsing error
          └── Null safety error
```

---

## B. Custom Failure Classes

Buat `lib/core/error/failures.dart` (versi lengkap):

```dart
abstract class Failure implements Exception {
  final String message;
  final int? statusCode;

  const Failure(this.message, {this.statusCode});

  @override
  String toString() => message;
}

// Gagal karena masalah koneksi internet
class NetworkFailure extends Failure {
  const NetworkFailure(super.message);
}

// Gagal karena server merespons dengan error
class ServerFailure extends Failure {
  const ServerFailure(super.message, {super.statusCode});
}

// Gagal karena data tidak ada / parsing error
class CacheFailure extends Failure {
  const CacheFailure(super.message);
}

// Gagal karena user tidak terautentikasi
class UnauthorizedFailure extends Failure {
  const UnauthorizedFailure() : super('Sesi berakhir. Silakan login kembali.');
}
```

---

## C. Global Error Handler di Interceptor

Update `auth_interceptor.dart` untuk handle 401:

```dart
@override
void onError(DioException err, ErrorInterceptorHandler handler) async {
  final statusCode = err.response?.statusCode;

  if (statusCode == 401) {
    // Token expired → hapus session dan arahkan ke login
    await LocalStorageService.clearSession();
    
    // Navigasi ke login (pakai navigatorKey jika di luar widget)
    navigatorKey.currentState?.pushNamedAndRemoveUntil('/login', (_) => false);
    return; // Hentikan error chain
  }

  handler.next(err);
}
```

Tambahkan navigatorKey di `main.dart`:

```dart
// lib/main.dart
final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      navigatorKey: navigatorKey, // ← Tambahkan ini
      // ...
    );
  }
}
```

---

## D. Widget Error yang Reusable

Buat `lib/core/widgets/error_widget.dart`:

```dart
import 'package:flutter/material.dart';

class AppErrorWidget extends StatelessWidget {
  final String message;
  final VoidCallback? onRetry;

  const AppErrorWidget({
    super.key,
    required this.message,
    this.onRetry,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.wifi_off_rounded,
              size: 80,
              color: Colors.grey,
            ),
            const SizedBox(height: 24),
            Text(
              'Oops! Terjadi Kesalahan',
              style: Theme.of(context).textTheme.titleLarge?.copyWith(
                    fontWeight: FontWeight.bold,
                  ),
            ),
            const SizedBox(height: 8),
            Text(
              message,
              textAlign: TextAlign.center,
              style: const TextStyle(color: Colors.grey),
            ),
            if (onRetry != null) ...[
              const SizedBox(height: 24),
              ElevatedButton.icon(
                onPressed: onRetry,
                icon: const Icon(Icons.refresh),
                label: const Text('Coba Lagi'),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

---

## E. Loading State Widget

Buat `lib/core/widgets/loading_widget.dart`:

```dart
import 'package:flutter/material.dart';

// Loading untuk list
class ShimmerListLoading extends StatelessWidget {
  final int itemCount;

  const ShimmerListLoading({super.key, this.itemCount = 5});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      padding: const EdgeInsets.all(16),
      itemCount: itemCount,
      itemBuilder: (context, index) => const _ShimmerCard(),
    );
  }
}

class _ShimmerCard extends StatefulWidget {
  const _ShimmerCard();

  @override
  State<_ShimmerCard> createState() => _ShimmerCardState();
}

class _ShimmerCardState extends State<_ShimmerCard>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1000),
    )..repeat(reverse: true);
    _animation = Tween<double>(begin: 0.4, end: 1.0).animate(_controller);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _animation,
      child: Card(
        margin: const EdgeInsets.only(bottom: 12),
        child: ListTile(
          leading: const CircleAvatar(backgroundColor: Colors.grey),
          title: Container(
            height: 14,
            width: 100,
            decoration: BoxDecoration(
              color: Colors.grey[300],
              borderRadius: BorderRadius.circular(4),
            ),
          ),
          subtitle: Container(
            height: 12,
            width: 150,
            decoration: BoxDecoration(
              color: Colors.grey[200],
              borderRadius: BorderRadius.circular(4),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## F. Penggunaan Lengkap di BlocBuilder

```dart
body: BlocConsumer<UserCubit, UserState>(
  listener: (context, state) {
    state.whenOrNull(
      error: (message) {
        ScaffoldMessenger.of(context)
          ..hideCurrentSnackBar()
          ..showSnackBar(
            SnackBar(
              content: Text(message),
              backgroundColor: Colors.red.shade700,
              action: SnackBarAction(
                label: 'Coba Lagi',
                textColor: Colors.white,
                onPressed: () => context.read<UserCubit>().fetchUsers(),
              ),
            ),
          );
      },
    );
  },
  builder: (context, state) {
    return state.when(
      initial: () => const SizedBox(),
      loading: () => const ShimmerListLoading(itemCount: 5), // ← Shimmer!
      loaded: (users) => RefreshIndicator(
        onRefresh: () => context.read<UserCubit>().refreshUsers(),
        child: users.isEmpty
          ? const Center(child: Text('Tidak ada data'))
          : ListView.builder(
              itemCount: users.length,
              itemBuilder: (_, i) => UserCard(user: users[i]),
            ),
      ),
      error: (message) => AppErrorWidget(
        message: message,
        onRetry: () => context.read<UserCubit>().fetchUsers(), // ← Retry!
      ),
    );
  },
),
```

---

## G. Connectivity Check (Bonus)

Tambahkan package `connectivity_plus` untuk cek koneksi sebelum fetch:

```bash
flutter pub add connectivity_plus
```

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

class UserCubit extends Cubit<UserState> {
  Future<void> fetchUsers() async {
    // Cek koneksi internet terlebih dahulu
    final connectivity = await Connectivity().checkConnectivity();
    if (connectivity == ConnectivityResult.none) {
      emit(const UserState.error('Tidak ada koneksi internet. Periksa WiFi atau data seluler kamu.'));
      return;
    }

    emit(const UserState.loading());
    try {
      final users = await _repository.getUsers();
      emit(UserState.loaded(users));
    } catch (e) {
      emit(UserState.error(e.toString()));
    }
  }
}
```

---

## Checklist Error Handling

- [ ] Semua `async` function dibungkus `try-catch`
- [ ] DioException diconvert ke Failure class yang deskriptif
- [ ] UI menampilkan pesan error yang *user-friendly* (bukan stack trace!)
- [ ] Ada tombol "Coba Lagi" di state error
- [ ] 401 Unauthorized mengarahkan user ke login
- [ ] Loading state menggunakan Shimmer (bukan spinner biasa)
- [ ] RefreshIndicator untuk pull-to-refresh

---

**Selanjutnya: [10_tips_best_practices.md](10_tips_best_practices.md)**