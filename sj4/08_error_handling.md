# 8️⃣ Error Handling & Offline Support

## Mengapa Error Handling Penting?

Aplikasi cloud sangat bergantung pada jaringan internet. Hal-hal yang bisa salah:
- Koneksi internet terputus
- Firestore Security Rules menolak akses
- Data tidak ditemukan
- Operasi timeout
- Quota Firebase habis

Tanpa error handling yang baik, user akan melihat **layar putih** atau **crash** tanpa penjelasan.

---

## Jenis Error di Firebase

### 1. `FirebaseException`
Error dari Firestore, Authentication, Storage, dll.

```dart
try {
  await firestore.collection('tasks').add(data);
} on FirebaseException catch (e) {
  print('Code: ${e.code}');       // e.g., 'permission-denied'
  print('Message: ${e.message}'); // Pesan detail
  print('Plugin: ${e.plugin}');   // e.g., 'cloud_firestore'
}
```

### 2. `FirebaseAuthException`
Subclass spesifik untuk Authentication.

```dart
try {
  await auth.signInWithEmailAndPassword(email: email, password: password);
} on FirebaseAuthException catch (e) {
  // e.code berisi: 'user-not-found', 'wrong-password', dll.
}
```

### 3. Network Error
Ketika perangkat tidak terhubung ke internet.

---

## Kode Error Firestore yang Umum

| Error Code | Penyebab | Solusi |
|---|---|---|
| `permission-denied` | Security Rules menolak | Cek rules di Firebase Console |
| `not-found` | Dokumen tidak ada | Tangani kasus `doc.exists == false` |
| `already-exists` | Dokumen sudah ada saat `create` | Gunakan `set()` dengan merge |
| `resource-exhausted` | Quota habis | Upgrade plan atau kurangi reads/writes |
| `unavailable` | Firestore sementara tidak bisa diakses | Retry dengan backoff |
| `deadline-exceeded` | Request timeout | Cek koneksi, retry |
| `cancelled` | Operasi dibatalkan | Normal jika user navigasi pergi |

---

## Membuat Custom Exception

Buat `lib/core/errors/app_exception.dart`:

```dart
/// Base class untuk semua exception di app
abstract class AppException implements Exception {
  final String message;
  const AppException(this.message);

  @override
  String toString() => message;
}

/// Exception untuk error dari Firestore
class FirestoreException extends AppException {
  final String? code;
  const FirestoreException(super.message, {this.code});
}

/// Exception untuk error dari Firebase Auth
class AuthException extends AppException {
  const AuthException(super.message);
}

/// Exception ketika tidak ada koneksi internet
class NetworkException extends AppException {
  const NetworkException()
      : super('Tidak ada koneksi internet. Periksa jaringan kamu.');
}

/// Exception ketika data tidak ditemukan
class NotFoundException extends AppException {
  const NotFoundException(super.message);
}
```

---

## Error Handler Terpusat

Buat `lib/core/errors/firebase_error_handler.dart`:

```dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'app_exception.dart';

class FirebaseErrorHandler {
  /// Konversi FirebaseException (Firestore) ke AppException
  static AppException handleFirestoreError(FirebaseException e) {
    switch (e.code) {
      case 'permission-denied':
        return const FirestoreException(
          'Akses ditolak. Kamu tidak punya izin untuk operasi ini.',
          code: 'permission-denied',
        );
      case 'not-found':
        return const NotFoundException('Data tidak ditemukan.');
      case 'unavailable':
        return const NetworkException();
      case 'deadline-exceeded':
        return const FirestoreException(
          'Koneksi timeout. Coba lagi.',
          code: 'deadline-exceeded',
        );
      case 'resource-exhausted':
        return const FirestoreException(
          'Layanan sedang sibuk. Coba beberapa saat lagi.',
          code: 'resource-exhausted',
        );
      default:
        return FirestoreException(
          e.message ?? 'Terjadi kesalahan tidak diketahui.',
          code: e.code,
        );
    }
  }

  /// Konversi FirebaseAuthException ke AppException
  static AppException handleAuthError(FirebaseAuthException e) {
    switch (e.code) {
      case 'user-not-found':
        return const AuthException('Email tidak terdaftar.');
      case 'wrong-password':
      case 'invalid-credential':
        return const AuthException('Email atau password salah.');
      case 'email-already-in-use':
        return const AuthException('Email sudah digunakan akun lain.');
      case 'invalid-email':
        return const AuthException('Format email tidak valid.');
      case 'weak-password':
        return const AuthException('Password terlalu lemah (min. 6 karakter).');
      case 'too-many-requests':
        return const AuthException(
            'Terlalu banyak percobaan. Coba lagi nanti.');
      case 'network-request-failed':
        return const NetworkException();
      case 'user-disabled':
        return const AuthException('Akun ini telah dinonaktifkan.');
      default:
        return AuthException(e.message ?? 'Login gagal.');
    }
  }
}
```

---

## Menggunakan Error Handler di Repository

```dart
// lib/data/repositories/task_repository.dart

import 'package:cloud_firestore/cloud_firestore.dart';
import '../../core/errors/firebase_error_handler.dart';
import '../models/task_model.dart';

class TaskRepository {
  final FirebaseFirestore _firestore;

  TaskRepository({FirebaseFirestore? firestore})
      : _firestore = firestore ?? FirebaseFirestore.instance;

  CollectionReference<Map<String, dynamic>> get _tasksCollection =>
      _firestore.collection('tasks');

  Future<void> addTask(TaskModel task) async {
    try {
      final data = task.toFirestore();
      data['createdAt'] = FieldValue.serverTimestamp();
      await _tasksCollection.add(data);
    } on FirebaseException catch (e) {
      throw FirebaseErrorHandler.handleFirestoreError(e);
    }
  }

  Future<List<TaskModel>> getTasks(String userId) async {
    try {
      final snapshot = await _tasksCollection
          .where('userId', isEqualTo: userId)
          .orderBy('createdAt', descending: true)
          .get();
      return snapshot.docs
          .map((doc) => TaskModel.fromFirestore(doc))
          .toList();
    } on FirebaseException catch (e) {
      throw FirebaseErrorHandler.handleFirestoreError(e);
    }
  }

  Future<void> deleteTask(String taskId) async {
    try {
      await _tasksCollection.doc(taskId).delete();
    } on FirebaseException catch (e) {
      throw FirebaseErrorHandler.handleFirestoreError(e);
    }
  }
}
```

---

## Offline Support Bawaan Firestore

Salah satu fitur terbaik Firestore adalah **offline persistence** — data tetap bisa dibaca dan ditulis meski tidak ada internet!

### Bagaimana Bekerjanya:
```
Online:
  App ──── Write ────► Local Cache ──── Sync ────► Firestore Cloud

Offline:
  App ──── Write ────► Local Cache (pending)
                           │
  Internet kembali          │
                           ▼
                       Sync ────► Firestore Cloud (otomatis!)
```

### Aktifkan Offline Persistence (Default aktif di mobile):
```dart
// Untuk Web, perlu diaktifkan manual
await FirebaseFirestore.instance.enablePersistence();

// Untuk mobile (Android/iOS), sudah aktif secara default
// Tidak perlu kode tambahan!
```

### Cek Apakah Data Dari Cache:
```dart
final snapshot = await _tasksCollection.get(
  const GetOptions(source: Source.cache), // paksa baca dari cache
);

// Atau
final snapshot = await _tasksCollection.get(
  const GetOptions(source: Source.server), // paksa baca dari server
);

// Default: serverFirst (server dulu, cache jika offline)
final snapshot = await _tasksCollection.get(); 
```

---

## Deteksi Status Koneksi

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

class ConnectivityService {
  final _connectivity = Connectivity();

  Stream<bool> get isConnected => _connectivity.onConnectivityChanged
      .map((result) => result != ConnectivityResult.none);

  Future<bool> get hasConnection async {
    final result = await _connectivity.checkConnectivity();
    return result != ConnectivityResult.none;
  }
}
```

Tambahkan package ke `pubspec.yaml`:
```yaml
dependencies:
  connectivity_plus: ^6.0.3
```

---

## UI: Menampilkan Banner Offline

```dart
class OfflineBanner extends StatelessWidget {
  const OfflineBanner({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<ConnectivityResult>(
      stream: Connectivity().onConnectivityChanged,
      builder: (context, snapshot) {
        final isOffline = snapshot.data == ConnectivityResult.none;
        if (!isOffline) return const SizedBox.shrink();
        
        return Container(
          width: double.infinity,
          color: Colors.orange,
          padding: const EdgeInsets.symmetric(vertical: 6, horizontal: 12),
          child: const Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(Icons.wifi_off, color: Colors.white, size: 16),
              SizedBox(width: 8),
              Text(
                'Offline — Perubahan akan disinkronkan saat online',
                style: TextStyle(color: Colors.white, fontSize: 12),
              ),
            ],
          ),
        );
      },
    );
  }
}
```

---

## Loading State yang Baik

```dart
// Di Widget
BlocBuilder<TaskCubit, TaskState>(
  builder: (context, state) {
    return Column(
      children: [
        // Banner offline
        const OfflineBanner(),
        
        // Konten utama
        Expanded(
          child: switch (state) {
            TaskLoading() => const Center(child: CircularProgressIndicator()),
            TaskLoaded(tasks: final tasks) => tasks.isEmpty
                ? const _EmptyState()
                : _TaskList(tasks: tasks),
            TaskError(message: final msg) => _ErrorState(
                message: msg,
                onRetry: () => context.read<TaskCubit>().loadTasks(userId),
              ),
            _ => const SizedBox.shrink(),
          },
        ),
      ],
    );
  },
)

// Widget kosong
class _EmptyState extends StatelessWidget {
  const _EmptyState();

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.inbox, size: 64, color: Colors.grey),
          SizedBox(height: 16),
          Text('Belum ada task', style: TextStyle(color: Colors.grey)),
        ],
      ),
    );
  }
}

// Widget error dengan tombol retry
class _ErrorState extends StatelessWidget {
  final String message;
  final VoidCallback onRetry;
  const _ErrorState({required this.message, required this.onRetry});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error_outline, size: 64, color: Colors.red),
            const SizedBox(height: 16),
            Text(message, textAlign: TextAlign.center),
            const SizedBox(height: 16),
            ElevatedButton.icon(
              onPressed: onRetry,
              icon: const Icon(Icons.refresh),
              label: const Text('Coba Lagi'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

*Lanjut ke [09_tips_best_practices.md](09_tips_best_practices.md) →*
