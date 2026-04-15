# 6️⃣ Real-time Updates dengan Stream & BlocBuilder

## Mengapa Real-time?

Di materi sebelumnya kita menggunakan `getTasks()` yang bersifat **one-time** — data hanya diambil sekali saat halaman dibuka.

Masalahnya: jika ada user lain yang menambah atau mengubah task, kamu harus **manual refresh**.

Solusinya: **Firestore Stream** → data otomatis update di UI setiap kali ada perubahan di Firestore.

```
Firestore              Flutter App
    │                      │
    │ ──── initial data ──► Stream
    │                      │ ──► UI (tampil data awal)
    │                      │
User B adds task           │
    │                      │
    │ ──── new snapshot ──► Stream
    │                      │ ──► UI (otomatis update!) 🔄
```

---

## Stream di Repository

Kita sudah membuat `watchTasks()` di repository:

```dart
// lib/data/repositories/task_repository.dart

Stream<List<TaskModel>> watchTasks(String userId) {
  return _tasksCollection
      .where('userId', isEqualTo: userId)
      .orderBy('createdAt', descending: true)
      .snapshots()           // ← .snapshots() = real-time stream
      .map((snapshot) =>
          snapshot.docs.map((doc) => TaskModel.fromFirestore(doc)).toList());
}
```

`.snapshots()` vs `.get()`:
| Method | Tipe | Kapan dipakai |
|---|---|---|
| `.get()` | `Future<QuerySnapshot>` | Baca sekali |
| `.snapshots()` | `Stream<QuerySnapshot>` | Real-time listener |

---

## Cara 1: StreamBuilder (Tanpa Bloc)

Cara paling sederhana — langsung pakai `StreamBuilder` di widget:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter/material.dart';
import '../../data/models/task_model.dart';
import '../../data/repositories/task_repository.dart';

class RealtimeTaskList extends StatelessWidget {
  final String userId;
  final TaskRepository repository;

  const RealtimeTaskList({
    super.key,
    required this.userId,
    required this.repository,
  });

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<List<TaskModel>>(
      stream: repository.watchTasks(userId),
      builder: (context, snapshot) {
        // Saat pertama kali, belum ada data
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }

        // Jika ada error
        if (snapshot.hasError) {
          return Center(child: Text('Error: ${snapshot.error}'));
        }

        // Jika tidak ada data
        final tasks = snapshot.data ?? [];
        if (tasks.isEmpty) {
          return const Center(child: Text('Belum ada task.'));
        }

        // Tampilkan list
        return ListView.builder(
          itemCount: tasks.length,
          itemBuilder: (context, index) {
            final task = tasks[index];
            return ListTile(
              title: Text(task.title),
              leading: Icon(
                task.isDone ? Icons.check_circle : Icons.circle_outlined,
                color: task.isDone ? Colors.green : null,
              ),
            );
          },
        );
      },
    );
  }
}
```

---

## Cara 2: Stream + Cubit (Clean Architecture)

Untuk aplikasi yang lebih besar, kelola stream di dalam Cubit:

### Update TaskCubit untuk stream:

```dart
import 'dart:async';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../data/models/task_model.dart';
import '../../data/repositories/task_repository.dart';

// Tambahkan state baru
class TaskStreamActive extends TaskState {
  final List<TaskModel> tasks;
  const TaskStreamActive(this.tasks);

  @override
  List<Object?> get props => [tasks];
}

// Di dalam TaskCubit:
class TaskCubit extends Cubit<TaskState> {
  final TaskRepository _repository;
  StreamSubscription<List<TaskModel>>? _taskSubscription;

  TaskCubit(this._repository) : super(TaskInitial());

  // Mulai listening ke stream
  void watchTasks(String userId) {
    emit(TaskLoading());

    // Batalkan subscription lama jika ada
    _taskSubscription?.cancel();

    _taskSubscription = _repository.watchTasks(userId).listen(
      (tasks) {
        emit(TaskStreamActive(tasks));
      },
      onError: (error) {
        emit(TaskError('Stream error: $error'));
      },
    );
  }

  // PENTING: hentikan stream saat Cubit di-dispose
  @override
  Future<void> close() {
    _taskSubscription?.cancel();
    return super.close();
  }

  // ... method CRUD lainnya tetap sama
}
```

### Di Widget, gunakan `BlocBuilder`:

```dart
class RealtimeHomePage extends StatefulWidget {
  final String userId;
  const RealtimeHomePage({super.key, required this.userId});

  @override
  State<RealtimeHomePage> createState() => _RealtimeHomePageState();
}

class _RealtimeHomePageState extends State<RealtimeHomePage> {
  @override
  void initState() {
    super.initState();
    // Mulai stream saat widget dibuat
    context.read<TaskCubit>().watchTasks(widget.userId);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Tasks (Real-time)')),
      body: BlocBuilder<TaskCubit, TaskState>(
        builder: (context, state) {
          if (state is TaskLoading) {
            return const Center(child: CircularProgressIndicator());
          }

          // Gunakan TaskStreamActive sebagai state real-time
          if (state is TaskStreamActive) {
            if (state.tasks.isEmpty) {
              return const Center(child: Text('Belum ada task.'));
            }
            return ListView.builder(
              itemCount: state.tasks.length,
              itemBuilder: (context, index) {
                final task = state.tasks[index];
                return ListTile(
                  title: Text(task.title),
                  subtitle: Text(task.description),
                  leading: Checkbox(
                    value: task.isDone,
                    onChanged: (_) => context
                        .read<TaskCubit>()
                        .toggleTask(task.id, task.isDone, widget.userId),
                  ),
                );
              },
            );
          }

          if (state is TaskError) {
            return Center(child: Text(state.message));
          }

          return const SizedBox.shrink();
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddDialog(BuildContext context) {
    final controller = TextEditingController();
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Task Baru'),
        content: TextField(
          controller: controller,
          decoration: const InputDecoration(hintText: 'Judul task...'),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Batal'),
          ),
          ElevatedButton(
            onPressed: () {
              if (controller.text.isNotEmpty) {
                context.read<TaskCubit>().addTask(
                      title: controller.text.trim(),
                      description: '',
                      userId: widget.userId,
                    );
                Navigator.pop(context);
              }
            },
            child: const Text('Tambah'),
          ),
        ],
      ),
    );
  }
}
```

---

## Visualisasi Aliran Real-time

```
Firestore (Cloud)
      │
      │  .snapshots()
      ▼
  Stream<QuerySnapshot>
      │
      │  .map(convert to TaskModel)
      ▼
  Stream<List<TaskModel>>
      │
      │  .listen()
      ▼
  TaskCubit._taskSubscription
      │
      │  emit(TaskStreamActive(tasks))
      ▼
  TaskState
      │
      │  BlocBuilder rebuild
      ▼
  UI (ListView, etc.)
```

Setiap perubahan di Firestore (dari device manapun) akan mengalir melalui pipeline ini secara otomatis.

---

## Snapshot Changes: Deteksi Perubahan Spesifik

Jika ingin tahu *apa* yang berubah (ditambah / diubah / dihapus):

```dart
Stream<void> watchTaskChanges(String userId) {
  return _tasksCollection
      .where('userId', isEqualTo: userId)
      .snapshots()
      .map((snapshot) {
        for (final change in snapshot.docChanges) {
          switch (change.type) {
            case DocumentChangeType.added:
              print('Dokumen baru: ${change.doc.id}');
              break;
            case DocumentChangeType.modified:
              print('Dokumen diubah: ${change.doc.id}');
              break;
            case DocumentChangeType.removed:
              print('Dokumen dihapus: ${change.doc.id}');
              break;
          }
        }
      });
}
```

---

## Perbedaan `snapshots()` vs `getDocuments()`

```dart
// ONE-TIME READ
final snapshot = await collection.get();
// Mengambil data sekali, Future selesai

// REAL-TIME READ
final stream = collection.snapshots();
stream.listen((snapshot) {
  // Dipanggil SETIAP KALI ada perubahan data
  // Tetap aktif sampai kamu cancel subscription
});
```

---

## Tips Performa

### Batasi Data yang Di-fetch
```dart
// Jangan fetch semua dokumen — batasi dengan query
_tasksCollection
    .where('userId', isEqualTo: userId)
    .limit(50)           // ambil max 50
    .snapshots();
```

### Pagination (Cursor-based)
```dart
// Halaman pertama
final firstPage = await _tasksCollection
    .orderBy('createdAt', descending: true)
    .limit(10)
    .get();

// Halaman berikutnya (mulai setelah dokumen terakhir)
final lastDoc = firstPage.docs.last;
final nextPage = await _tasksCollection
    .orderBy('createdAt', descending: true)
    .startAfterDocument(lastDoc)
    .limit(10)
    .get();
```

### Selalu Cancel Stream Subscription
```dart
// Di Cubit:
@override
Future<void> close() {
  _taskSubscription?.cancel(); // ← WAJIB
  return super.close();
}

// Di StatefulWidget (jika pakai StreamBuilder langsung):
// StreamBuilder otomatis handle ini ✅
```

---

*Lanjut ke [07_firebase_auth.md](07_firebase_auth.md) →*
