# Data Model dengan Freezed & JSON Parsing

## Mengapa Freezed?

Menulis data class di Dart secara manual itu melelahkan:

```dart
class User {
  final int id;
  final String name;

  User({required this.id, required this.name});

  // Harus tulis sendiri:
  User copyWith({int? id, String? name}) => User(id: id ?? this.id, name: name ?? this.name);
  bool operator ==(Object o) => o is User && o.id == id && o.name == name;
  int get hashCode => id.hashCode ^ name.hashCode;
  String toString() => 'User(id: $id, name: $name)';
  factory User.fromJson(Map<String, dynamic> json) => User(id: json['id'], name: json['name']);
  Map<String, dynamic> toJson() => {'id': id, 'name': name};
}
```

Dengan **Freezed**, semua itu di-*generate* otomatis! ✨

---

## Step 1: Buat Entity (Domain Layer)

Entity adalah model **murni** tanpa tergantung package eksternal.

Buat `lib/features/users/domain/entities/user.dart`:

```dart
class User {
  final int id;
  final String name;
  final String email;
  final String phone;
  final String website;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.phone,
    required this.website,
  });
}
```

---

## Step 2: Buat Model dengan Freezed (Data Layer)

Model adalah Entity + kemampuan parsing JSON.

Buat `lib/features/users/data/models/user_model.dart`:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../domain/entities/user.dart';

// Wajib! Referensi ke file yang di-generate
part 'user_model.freezed.dart';
part 'user_model.g.dart';

@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required int id,
    required String name,
    required String email,
    required String phone,
    required String website,
  }) = _UserModel;

  // Factory untuk parsing dari JSON (dari API)
  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);
}

// Extension untuk konversi UserModel → User (Entity)
extension UserModelX on UserModel {
  User toEntity() {
    return User(
      id: id,
      name: name,
      email: email,
      phone: phone,
      website: website,
    );
  }
}
```

---

## Step 3: Jalankan Code Generation

```bash
dart run build_runner build --delete-conflicting-outputs
```

> Ini akan membuat file `user_model.freezed.dart` dan `user_model.g.dart` secara otomatis.

Untuk mode watch (otomatis regenerate saat file berubah):

```bash
dart run build_runner watch --delete-conflicting-outputs
```

---

## Step 4: Buat State Model dengan Freezed

State Cubit juga bisa pakai Freezed! Ini memudahkan kita mendefinisikan berbagai kondisi state.

Buat `lib/features/users/presentation/bloc/user_state.dart`:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../domain/entities/user.dart';

part 'user_state.freezed.dart';

@freezed
class UserState with _$UserState {
  // State awal — belum ada aksi
  const factory UserState.initial() = _Initial;

  // Sedang loading data
  const factory UserState.loading() = _Loading;

  // Data berhasil dimuat
  const factory UserState.loaded(List<User> users) = _Loaded;

  // Terjadi error
  const factory UserState.error(String message) = _Error;
}
```

Jalankan build_runner lagi setelah membuat file ini:

```bash
dart run build_runner build --delete-conflicting-outputs
```

---

## Fitur Unggulan Freezed

### `copyWith` — Salin dengan Perubahan

```dart
final user = UserModel(id: 1, name: 'Ali', email: 'ali@email.com', phone: '08xx', website: 'ali.dev');

// Buat salinan dengan name yang berbeda
final updatedUser = user.copyWith(name: 'Budi');
print(updatedUser.name);  // Budi
print(updatedUser.email); // ali@email.com (tetap sama)
```

### `when` — Pattern Matching pada State

```dart
// Di BlocBuilder, kita bisa pakai .when() untuk pattern matching
state.when(
  initial: () => const SizedBox(),
  loading: () => const CircularProgressIndicator(),
  loaded: (users) => UserListWidget(users: users),
  error: (message) => ErrorWidget(message: message),
);
```

### `==` Comparison Otomatis

```dart
final user1 = UserModel(id: 1, name: 'Ali', email: 'ali@email.com', phone: '08xx', website: 'ali.dev');
final user2 = UserModel(id: 1, name: 'Ali', email: 'ali@email.com', phone: '08xx', website: 'ali.dev');

print(user1 == user2); // ✅ true — tanpa override manual!
```

---

## Perbedaan Model vs Entity

| | **Entity** | **Model** |
|---|---|---|
| Lokasi | `domain/entities/` | `data/models/` |
| Tujuan | Representasi bisnis | Parsing data eksternal |
| Dependency | Tidak ada | `freezed_annotation`, `json_annotation` |
| JSON parsing | ❌ | ✅ (`fromJson`, `toJson`) |

---

## File yang Dihasilkan

```
lib/features/users/data/models/
├── user_model.dart           ← Kamu tulis ini
├── user_model.freezed.dart   ← Auto-generated (jangan edit!)
└── user_model.g.dart         ← Auto-generated (jangan edit!)
```

> Jangan pernah mengedit file `.freezed.dart` atau `.g.dart` secara manual. Selalu ubah melalui source file-nya, lalu jalankan ulang `build_runner`.

---

**Selanjutnya: [05_bloc_cubit.md](05_bloc_cubit.md)**