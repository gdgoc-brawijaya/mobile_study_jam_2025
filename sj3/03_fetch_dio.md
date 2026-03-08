# HTTP Request dengan Dio

## Mengapa Dio?

| Fitur | `http` (bawaan) | `dio` |
|---|---|---|
| GET / POST | ✅ | ✅ |
| Interceptor | ❌ | ✅ |
| Timeout built-in | ❌ | ✅ |
| Cancel request | ❌ | ✅ |
| FormData / multipart | ❌ | ✅ |
| Response transformer | ❌ | ✅ |

---

## Step 1: Konfigurasi Dio Client

Buat `lib/core/network/dio_client.dart`:

```dart
import 'package:dio/dio.dart';
import '../constants/api_constants.dart';
import 'interceptors/auth_interceptor.dart';
import 'interceptors/log_interceptor.dart';

class DioClient {
  late final Dio _dio;

  DioClient() {
    _dio = Dio(
      BaseOptions(
        baseUrl: ApiConstants.baseUrl,
        connectTimeout: const Duration(seconds: 10),
        receiveTimeout: const Duration(seconds: 10),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      ),
    );

    // Tambahkan interceptors
    _dio.interceptors.addAll([
      AuthInterceptor(),      // Menambahkan token otomatis
      AppLogInterceptor(),    // Log request & response
    ]);
  }

  // Expose Dio instance
  Dio get dio => _dio;
}
```

---

## Step 2: Membuat Interceptor

### Auth Interceptor
Menambahkan token secara otomatis ke setiap request.

Buat `lib/core/network/interceptors/auth_interceptor.dart`:

```dart
import 'package:dio/dio.dart';
import 'package:shared_preferences/shared_preferences.dart';

class AuthInterceptor extends Interceptor {
  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    // Ambil token dari SharedPreferences
    final prefs = await SharedPreferences.getInstance();
    final token = prefs.getString('ACCESS_TOKEN');

    // Jika ada token, tambahkan ke header
    if (token != null && token.isNotEmpty) {
      options.headers['Authorization'] = 'Bearer $token';
    }

    // Lanjutkan request
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    // Handle 401 Unauthorized: paksa logout
    if (err.response?.statusCode == 401) {
      // TODO: Navigasi ke halaman login / clear session
      print('Unauthorized! Redirect to login...');
    }
    handler.next(err);
  }
}
```

### Log Interceptor
Menampilkan detail request & response di console saat development.

Buat `lib/core/network/interceptors/log_interceptor.dart`:

```dart
import 'package:dio/dio.dart';

class AppLogInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    print('┌── REQUEST ─────────────────────────');
    print('│ [${options.method}] ${options.uri}');
    print('│ Headers: ${options.headers}');
    if (options.data != null) print('│ Body: ${options.data}');
    print('└────────────────────────────────────');
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    print('┌── RESPONSE ────────────────────────');
    print('│ Status: ${response.statusCode}');
    print('│ Data: ${response.data.toString().substring(0, 
          response.data.toString().length.clamp(0, 200))}...');
    print('└────────────────────────────────────');
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    print('┌── ERROR ───────────────────────────');
    print('│ Status: ${err.response?.statusCode}');
    print('│ Message: ${err.message}');
    print('└────────────────────────────────────');
    handler.next(err);
  }
}
```

---

## Step 3: Membuat API Service

Buat `lib/features/users/data/datasources/user_remote_datasource.dart`:

```dart
import 'package:dio/dio.dart';
import '../../../../core/constants/api_constants.dart';
import '../../../../core/error/failures.dart';
import '../models/user_model.dart';

abstract class UserRemoteDataSource {
  Future<List<UserModel>> getUsers();
  Future<UserModel> getUserById(int id);
}

class UserRemoteDataSourceImpl implements UserRemoteDataSource {
  final Dio dio;

  UserRemoteDataSourceImpl({required this.dio});

  @override
  Future<List<UserModel>> getUsers() async {
    try {
      final response = await dio.get(ApiConstants.usersEndpoint);

      if (response.statusCode == 200) {
        // Parse list dari JSON
        final List<dynamic> jsonList = response.data;
        return jsonList.map((json) => UserModel.fromJson(json)).toList();
      }

      throw ServerFailure('Unexpected status: ${response.statusCode}');
    } on DioException catch (e) {
      throw _handleDioError(e);
    }
  }

  @override
  Future<UserModel> getUserById(int id) async {
    try {
      final response = await dio.get('${ApiConstants.usersEndpoint}/$id');

      if (response.statusCode == 200) {
        return UserModel.fromJson(response.data);
      }

      throw ServerFailure('User not found');
    } on DioException catch (e) {
      throw _handleDioError(e);
    }
  }

  // Helper: konversi DioException ke Failure yang lebih deskriptif
  Failure _handleDioError(DioException e) {
    switch (e.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.receiveTimeout:
        return NetworkFailure('Connection timeout. Periksa internet kamu.');
      case DioExceptionType.connectionError:
        return NetworkFailure('Tidak ada koneksi internet.');
      case DioExceptionType.badResponse:
        final statusCode = e.response?.statusCode;
        if (statusCode == 404) return ServerFailure('Data tidak ditemukan (404).');
        if (statusCode == 500) return ServerFailure('Server error (500).');
        return ServerFailure('Server error: $statusCode');
      default:
        return ServerFailure('Terjadi kesalahan: ${e.message}');
    }
  }
}
```

---

## Step 4: Failure Classes

Buat `lib/core/error/failures.dart`:

```dart
abstract class Failure {
  final String message;
  const Failure(this.message);
}

class ServerFailure extends Failure {
  const ServerFailure(super.message);
}

class NetworkFailure extends Failure {
  const NetworkFailure(super.message);
}

class CacheFailure extends Failure {
  const CacheFailure(super.message);
}
```

---

## Test Manual (Main)

Untuk mengetes sebelum integrasi penuh, kamu bisa test langsung di `main.dart`:

```dart
void main() async {
  final dioClient = DioClient();
  final dataSource = UserRemoteDataSourceImpl(dio: dioClient.dio);

  try {
    final users = await dataSource.getUsers();
    print('Berhasil! Jumlah user: ${users.length}');
    print('User pertama: ${users.first.name}');
  } catch (e) {
    print('Error: $e');
  }
}
```

---

## Ringkasan

```
DioClient
  ├── BaseOptions (baseUrl, timeout, headers)
  └── Interceptors
        ├── AuthInterceptor  → tambah token otomatis
        └── LogInterceptor   → tampilkan log di console
              │
              ▼
    UserRemoteDataSource
        ├── getUsers()       → GET /users
        └── getUserById(id)  → GET /users/:id
```

---

**Selanjutnya: [04_model_freezed.md](04_model_freezed.md)**