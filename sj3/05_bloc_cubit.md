# State Management dengan Bloc & Cubit

## Bloc vs Cubit — Mana yang Dipilih?

```
          EVENTS                    CUBIT
    ┌───────────────┐         ┌───────────────┐
    │  UserEvent    │         │               │
    │  ├─ Fetch     │         │ method calls  │
    │  ├─ Refresh   │ ──vs──  │ fetchUsers()  │
    │  └─ Delete    │         │ refreshUsers()│
    └───────────────┘         │ deleteUser()  │
           │                  └───────────────┘
           ▼
    ┌───────────────┐
    │   UserBloc    │
    │  on<Event>()  │
    └───────────────┘
```

| | **Cubit** | **Bloc** |
|---|---|---|
| Trigger | Method call langsung | Event object |
| Kompleksitas | Lebih sederhana ✅ | Lebih verbose |
| Cocok untuk | Sebagian besar kasus | Event sourcing, audit log |
| Testability | Mudah | Sangat mudah |

> **Rekomendasi:** Mulai dengan **Cubit**. Migrasi ke Bloc hanya jika perlu event sourcing atau kondisi yang sangat kompleks.

---

## A. Implementasi dengan Cubit

### 1. Buat Repository Interface

Buat `lib/features/users/domain/repositories/user_repository.dart`:

```dart
import '../entities/user.dart';

abstract class UserRepository {
  Future<List<User>> getUsers();
  Future<User> getUserById(int id);
}
```

### 2. Buat Repository Implementation

Buat `lib/features/users/data/repositories/user_repository_impl.dart`:

```dart
import '../../domain/entities/user.dart';
import '../../domain/repositories/user_repository.dart';
import '../datasources/user_remote_datasource.dart';
import '../models/user_model.dart';

class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;

  UserRepositoryImpl({required this.remoteDataSource});

  @override
  Future<List<User>> getUsers() async {
    // Ambil data dari remote
    final userModels = await remoteDataSource.getUsers();
    
    // Konversi Model → Entity
    return userModels.map((model) => model.toEntity()).toList();
  }

  @override
  Future<User> getUserById(int id) async {
    final userModel = await remoteDataSource.getUserById(id);
    return userModel.toEntity();
  }
}
```

### 3. Buat UserCubit

Buat `lib/features/users/presentation/bloc/user_cubit.dart`:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../domain/repositories/user_repository.dart';
import '../../domain/entities/user.dart';
import 'user_state.dart';

class UserCubit extends Cubit<UserState> {
  final UserRepository _repository;

  // State awal adalah 'initial'
  UserCubit({required UserRepository repository})
      : _repository = repository,
        super(const UserState.initial());

  // Ambil semua user
  Future<void> fetchUsers() async {
    // Ubah state menjadi loading
    emit(const UserState.loading());

    try {
      final users = await _repository.getUsers();
      
      // Jika data kosong, tetap emit loaded dengan list kosong
      emit(UserState.loaded(users));
    } catch (e) {
      // Ubah state menjadi error dengan pesan
      emit(UserState.error(e.toString()));
    }
  }

  // Refresh: sama dengan fetch tapi bisa dipanggil ulang
  Future<void> refreshUsers() => fetchUsers();
}
```

### 4. Buat UI dengan BlocBuilder

Buat `lib/features/users/presentation/pages/user_page.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../../core/di/injection.dart';
import '../bloc/user_cubit.dart';
import '../bloc/user_state.dart';
import '../widgets/user_card.dart';

class UserPage extends StatelessWidget {
  const UserPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      // Inject Cubit via GetIt, langsung fetch saat inisialisasi
      create: (_) => sl<UserCubit>()..fetchUsers(),
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Daftar User'),
          actions: [
            // Tombol refresh
            BlocBuilder<UserCubit, UserState>(
              builder: (context, state) {
                return IconButton(
                  icon: const Icon(Icons.refresh),
                  onPressed: () => context.read<UserCubit>().refreshUsers(),
                );
              },
            ),
          ],
        ),
        body: BlocConsumer<UserCubit, UserState>(
          // listener: untuk side-effect (snackbar, navigasi, dll)
          listener: (context, state) {
            state.whenOrNull(
              error: (message) {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text(message),
                    backgroundColor: Colors.red,
                  ),
                );
              },
            );
          },
          // builder: untuk membangun UI berdasarkan state
          builder: (context, state) {
            return state.when(
              initial: () => const SizedBox(),
              loading: () => const Center(
                child: CircularProgressIndicator(),
              ),
              loaded: (users) {
                if (users.isEmpty) {
                  return const Center(child: Text('Tidak ada data'));
                }
                return RefreshIndicator(
                  onRefresh: () => context.read<UserCubit>().refreshUsers(),
                  child: ListView.builder(
                    padding: const EdgeInsets.all(16),
                    itemCount: users.length,
                    itemBuilder: (context, index) {
                      return UserCard(user: users[index]);
                    },
                  ),
                );
              },
              error: (message) => Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Icon(Icons.error_outline, size: 48, color: Colors.red),
                    const SizedBox(height: 16),
                    Text(message, textAlign: TextAlign.center),
                    const SizedBox(height: 16),
                    ElevatedButton(
                      onPressed: () => context.read<UserCubit>().fetchUsers(),
                      child: const Text('Coba Lagi'),
                    ),
                  ],
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

### 5. Buat Widget UserCard

Buat `lib/features/users/presentation/widgets/user_card.dart`:

```dart
import 'package:flutter/material.dart';
import '../../domain/entities/user.dart';

class UserCard extends StatelessWidget {
  final User user;

  const UserCard({super.key, required this.user});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: Colors.blue,
          child: Text(
            user.name[0].toUpperCase(),
            style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
          ),
        ),
        title: Text(user.name, style: const TextStyle(fontWeight: FontWeight.bold)),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(user.email),
            Text(user.phone, style: const TextStyle(color: Colors.grey)),
          ],
        ),
        trailing: const Icon(Icons.chevron_right),
        isThreeLine: true,
      ),
    );
  }
}
```

---

## B. BlocBuilder vs BlocListener vs BlocConsumer

```
┌──────────────────────────────────────────────────────┐
│                  BlocConsumer                         │
│                                                       │
│  ┌─────────────────────┐  ┌──────────────────────┐   │
│  │    BlocBuilder      │  │    BlocListener      │   │
│  │                     │  │                      │   │
│  │  Rebuild UI saat    │  │  Side effects saat   │   │
│  │  state berubah      │  │  state berubah       │   │
│  │                     │  │  (Snackbar, Nav, dll)│   │
│  │  return Widget      │  │  return void         │   │
│  └─────────────────────┘  └──────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

| Widget | Gunakan Untuk |
|---|---|
| `BlocBuilder` | Rebuild UI berdasarkan state |
| `BlocListener` | Side-effect: navigasi, snackbar, dialog |
| `BlocConsumer` | Keduanya sekaligus |
| `context.read<C>()` | Memanggil method Cubit (tanpa listen) |
| `context.watch<C>()` | Mendapat state terbaru (rebuild widget) |

---

## C. Testing Cubit

Salah satu keuntungan Cubit adalah mudah di-test!

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:bloc_test/bloc_test.dart';
import 'package:mocktail/mocktail.dart';

class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late UserCubit cubit;
  late MockUserRepository mockRepo;

  setUp(() {
    mockRepo = MockUserRepository();
    cubit = UserCubit(repository: mockRepo);
  });

  tearDown(() => cubit.close());

  blocTest<UserCubit, UserState>(
    'emit [loading, loaded] when fetchUsers succeeds',
    build: () {
      when(() => mockRepo.getUsers()).thenAnswer(
        (_) async => [User(id: 1, name: 'Ali', email: 'ali@test.com', phone: '08', website: 'ali.dev')],
      );
      return cubit;
    },
    act: (c) => c.fetchUsers(),
    expect: () => [
      const UserState.loading(),
      isA<UserState>().having((s) => s, 'loaded state', isA<_Loaded>()),
    ],
  );

  blocTest<UserCubit, UserState>(
    'emit [loading, error] when fetchUsers fails',
    build: () {
      when(() => mockRepo.getUsers()).thenThrow(Exception('Network error'));
      return cubit;
    },
    act: (c) => c.fetchUsers(),
    expect: () => [
      const UserState.loading(),
      isA<UserState>().having((s) => s.runtimeType.toString(), 'error state', contains('Error')),
    ],
  );
}
```

---

**Selanjutnya: [06_shared_preferences.md](06_shared_preferences.md)**