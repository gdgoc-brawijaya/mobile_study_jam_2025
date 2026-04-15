# 5️⃣ CRUD Operasi dengan Cloud Firestore

## Gambaran Besar

CRUD = **C**reate, **R**ead, **U**pdate, **D**elete

Di Firestore:
| Operasi | Method |
|---|---|
| **Create** | `collection.add()` / `document.set()` |
| **Read** | `document.get()` / `collection.get()` |
| **Update** | `document.update()` / `document.set(merge: true)` |
| **Delete** | `document.delete()` |

---

## Setup: Cubit untuk Task

Buat `lib/presentation/bloc/task_cubit.dart`:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';
import '../../data/models/task_model.dart';
import '../../data/repositories/task_repository.dart';

// ── State ──────────────────────────────────────────────────────
abstract class TaskState extends Equatable {
  @override
  List<Object?> get props => [];
}

class TaskInitial extends TaskState {}

class TaskLoading extends TaskState {}

class TaskLoaded extends TaskState {
  final List<TaskModel> tasks;
  const TaskLoaded(this.tasks);

  @override
  List<Object?> get props => [tasks];
}

class TaskError extends TaskState {
  final String message;
  const TaskError(this.message);

  @override
  List<Object?> get props => [message];
}

class TaskActionSuccess extends TaskState {
  final String message;
  const TaskActionSuccess(this.message);

  @override
  List<Object?> get props => [message];
}

// ── Cubit ──────────────────────────────────────────────────────
class TaskCubit extends Cubit<TaskState> {
  final TaskRepository _repository;

  TaskCubit(this._repository) : super(TaskInitial());

  // CREATE
  Future<void> addTask({
    required String title,
    required String description,
    required String userId,
  }) async {
    emit(TaskLoading());
    try {
      final task = TaskModel(
        id: '',                        // Akan di-set oleh Firestore
        title: title,
        description: description,
        isDone: false,
        createdAt: DateTime.now(),     // Placeholder; server akan overwrite
        userId: userId,
      );
      await _repository.addTask(task);
      emit(const TaskActionSuccess('Task berhasil ditambahkan!'));
    } catch (e) {
      emit(TaskError('Gagal menambahkan task: $e'));
    }
  }

  // READ (one-time)
  Future<void> loadTasks(String userId) async {
    emit(TaskLoading());
    try {
      final tasks = await _repository.getTasks(userId);
      emit(TaskLoaded(tasks));
    } catch (e) {
      emit(TaskError('Gagal memuat tasks: $e'));
    }
  }

  // UPDATE
  Future<void> updateTask(TaskModel task) async {
    try {
      await _repository.updateTask(task);
      emit(const TaskActionSuccess('Task berhasil diupdate!'));
    } catch (e) {
      emit(TaskError('Gagal mengupdate task: $e'));
    }
  }

  // DELETE
  Future<void> deleteTask(String taskId, String userId) async {
    try {
      await _repository.deleteTask(taskId);
      // Reload setelah delete
      await loadTasks(userId);
    } catch (e) {
      emit(TaskError('Gagal menghapus task: $e'));
    }
  }

  // TOGGLE isDone
  Future<void> toggleTask(String taskId, bool currentStatus, String userId) async {
    try {
      await _repository.toggleTaskDone(taskId, currentStatus);
      await loadTasks(userId);
    } catch (e) {
      emit(TaskError('Gagal update status: $e'));
    }
  }
}
```

---

## C — Create (Menambahkan Dokumen)

### Cara 1: `add()` — ID Otomatis (Recommended)
```dart
// Repository
Future<void> addTask(TaskModel task) async {
  // Firestore generates random Document ID
  final docRef = await _tasksCollection.add(task.toFirestore());
  print('New doc ID: ${docRef.id}');
}
```

### Cara 2: `set()` — ID Manual
```dart
// Gunakan ini jika kamu ingin control ID-nya
Future<void> addTaskWithCustomId(TaskModel task, String customId) async {
  await _tasksCollection.doc(customId).set(task.toFirestore());
}
```

### Cara 3: Tambahkan `serverTimestamp()`
```dart
Future<void> addTask(TaskModel task) async {
  final data = task.toFirestore();
  // Override createdAt dengan server timestamp
  data['createdAt'] = FieldValue.serverTimestamp();
  await _tasksCollection.add(data);
}
```

---

## R — Read (Membaca Dokumen)

### Baca Satu Dokumen
```dart
Future<TaskModel?> getTaskById(String taskId) async {
  final doc = await _tasksCollection.doc(taskId).get();
  
  if (!doc.exists) return null;
  return TaskModel.fromFirestore(doc);
}
```

### Baca Semua Dokumen di Collection
```dart
Future<List<TaskModel>> getAllTasks() async {
  final snapshot = await _tasksCollection.get();
  return snapshot.docs.map((doc) => TaskModel.fromFirestore(doc)).toList();
}
```

### Baca dengan Query (Filter)
```dart
Future<List<TaskModel>> getCompletedTasks(String userId) async {
  final snapshot = await _tasksCollection
      .where('userId', isEqualTo: userId)   // filter by user
      .where('isDone', isEqualTo: true)      // filter done tasks
      .orderBy('createdAt', descending: true) // sort by date
      .limit(20)                             // batasi 20 dokumen
      .get();

  return snapshot.docs.map((doc) => TaskModel.fromFirestore(doc)).toList();
}
```

### Operator Query yang Tersedia
```dart
// Sama dengan
.where('status', isEqualTo: 'active')

// Tidak sama dengan
.where('status', isNotEqualTo: 'deleted')

// Lebih dari / kurang dari
.where('count', isGreaterThan: 5)
.where('count', isLessThanOrEqualTo: 100)

// Array contains
.where('tags', arrayContains: 'flutter')

// IN operator
.where('status', whereIn: ['active', 'pending'])

// Range
.where('createdAt', isGreaterThan: Timestamp.fromDate(startDate))
```

---

## U — Update (Mengubah Dokumen)

### Update Field Tertentu (Partial Update)
```dart
// Hanya update field yang disebutkan — field lain tidak berubah
Future<void> updateTaskTitle(String taskId, String newTitle) async {
  await _tasksCollection.doc(taskId).update({
    'title': newTitle,
    'updatedAt': FieldValue.serverTimestamp(),
  });
}
```

### Update Seluruh Dokumen (Full Update)
```dart
Future<void> updateTask(TaskModel task) async {
  await _tasksCollection.doc(task.id).update(task.toFirestore());
}
```

### Update atau Buat jika Tidak Ada (Upsert)
```dart
Future<void> upsertTask(TaskModel task) async {
  // merge: true → update field yang ada, buat jika dokumen tidak ada
  await _tasksCollection.doc(task.id).set(
    task.toFirestore(),
    SetOptions(merge: true),
  );
}
```

### Increment / Decrement Numerik
```dart
// Atomic increment — aman untuk concurrent writes
Future<void> incrementCount(String docId) async {
  await _tasksCollection.doc(docId).update({
    'count': FieldValue.increment(1),
  });
}
```

### Array Operations
```dart
// Tambah elemen ke array (tanpa duplikat)
await docRef.update({
  'tags': FieldValue.arrayUnion(['firebase', 'flutter']),
});

// Hapus elemen dari array
await docRef.update({
  'tags': FieldValue.arrayRemove(['old-tag']),
});
```

---

## D — Delete (Menghapus Dokumen)

### Hapus Dokumen
```dart
Future<void> deleteTask(String taskId) async {
  await _tasksCollection.doc(taskId).delete();
}
```

### Hapus Field Tertentu (bukan seluruh dokumen)
```dart
Future<void> removeDescription(String taskId) async {
  await _tasksCollection.doc(taskId).update({
    'description': FieldValue.delete(), // hapus field ini
  });
}
```

### Hapus Collection (Batch Delete)
```dart
// Firestore tidak bisa hapus collection sekaligus
// Harus hapus dokumen satu-satu (atau gunakan batch)
Future<void> deleteAllUserTasks(String userId) async {
  final snapshot = await _tasksCollection
      .where('userId', isEqualTo: userId)
      .get();
  
  // Gunakan WriteBatch untuk efisiensi
  final batch = _firestore.batch();
  for (final doc in snapshot.docs) {
    batch.delete(doc.reference);
  }
  await batch.commit();
}
```

---

## Batch Write (Operasi Berganda Atomik)

Gunakan `WriteBatch` ketika perlu beberapa operasi yang harus sukses semua atau gagal semua:

```dart
Future<void> completeManyTasks(List<String> taskIds) async {
  final batch = _firestore.batch();
  
  for (final id in taskIds) {
    batch.update(_tasksCollection.doc(id), {
      'isDone': true,
      'completedAt': FieldValue.serverTimestamp(),
    });
  }
  
  // Semua update dikirim dalam satu request
  await batch.commit();
}
```

> Satu `WriteBatch` bisa menampung maksimal **500 operasi**.

---

## Transaction (Baca + Tulis Atomik)

Gunakan `Transaction` ketika operasi tulis bergantung pada hasil baca:

```dart
Future<void> transferTask(String taskId, String toUserId) async {
  await _firestore.runTransaction((transaction) async {
    // Baca dulu
    final docRef = _tasksCollection.doc(taskId);
    final snapshot = await transaction.get(docRef);
    
    if (!snapshot.exists) {
      throw Exception('Task tidak ditemukan');
    }
    
    // Tulis berdasarkan hasil baca
    transaction.update(docRef, {
      'userId': toUserId,
      'transferredAt': FieldValue.serverTimestamp(),
    });
  });
}
```

---

## UI: Halaman Utama dengan List Tasks

Buat `lib/presentation/pages/home_page.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/task_cubit.dart';
import '../../data/models/task_model.dart';
import 'add_task_page.dart';

class HomePage extends StatefulWidget {
  final String userId;
  const HomePage({super.key, required this.userId});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  @override
  void initState() {
    super.initState();
    // Load tasks saat halaman pertama dibuka
    context.read<TaskCubit>().loadTasks(widget.userId);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('My Tasks'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: BlocConsumer<TaskCubit, TaskState>(
        listener: (context, state) {
          if (state is TaskError) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text(state.message),
                backgroundColor: Colors.red,
              ),
            );
          }
          if (state is TaskActionSuccess) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.message)),
            );
            // Reload setelah aksi berhasil
            context.read<TaskCubit>().loadTasks(widget.userId);
          }
        },
        builder: (context, state) {
          if (state is TaskLoading) {
            return const Center(child: CircularProgressIndicator());
          }
          if (state is TaskLoaded) {
            if (state.tasks.isEmpty) {
              return const Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.task_alt, size: 64, color: Colors.grey),
                    SizedBox(height: 16),
                    Text('Belum ada task. Tambahkan sekarang!'),
                  ],
                ),
              );
            }
            return ListView.builder(
              itemCount: state.tasks.length,
              itemBuilder: (context, index) {
                final task = state.tasks[index];
                return _TaskTile(
                  task: task,
                  userId: widget.userId,
                );
              },
            );
          }
          return const SizedBox.shrink();
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          await Navigator.push(
            context,
            MaterialPageRoute(
              builder: (_) => AddTaskPage(userId: widget.userId),
            ),
          );
          // Reload setelah kembali dari AddTaskPage
          if (mounted) {
            context.read<TaskCubit>().loadTasks(widget.userId);
          }
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}

class _TaskTile extends StatelessWidget {
  final TaskModel task;
  final String userId;

  const _TaskTile({required this.task, required this.userId});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: Checkbox(
        value: task.isDone,
        onChanged: (_) => context.read<TaskCubit>().toggleTask(
              task.id,
              task.isDone,
              userId,
            ),
      ),
      title: Text(
        task.title,
        style: TextStyle(
          decoration: task.isDone ? TextDecoration.lineThrough : null,
          color: task.isDone ? Colors.grey : null,
        ),
      ),
      subtitle: task.description.isNotEmpty ? Text(task.description) : null,
      trailing: IconButton(
        icon: const Icon(Icons.delete_outline, color: Colors.red),
        onPressed: () => _confirmDelete(context),
      ),
    );
  }

  void _confirmDelete(BuildContext context) {
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Hapus Task'),
        content: Text('Hapus "${task.title}"?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Batal'),
          ),
          TextButton(
            onPressed: () {
              Navigator.pop(context);
              context.read<TaskCubit>().deleteTask(task.id, userId);
            },
            child: const Text('Hapus', style: TextStyle(color: Colors.red)),
          ),
        ],
      ),
    );
  }
}
```

---

## UI: Halaman Tambah Task

Buat `lib/presentation/pages/add_task_page.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/task_cubit.dart';

class AddTaskPage extends StatefulWidget {
  final String userId;
  const AddTaskPage({super.key, required this.userId});

  @override
  State<AddTaskPage> createState() => _AddTaskPageState();
}

class _AddTaskPageState extends State<AddTaskPage> {
  final _formKey = GlobalKey<FormState>();
  final _titleController = TextEditingController();
  final _descController = TextEditingController();

  @override
  void dispose() {
    _titleController.dispose();
    _descController.dispose();
    super.dispose();
  }

  void _submit() {
    if (_formKey.currentState!.validate()) {
      context.read<TaskCubit>().addTask(
            title: _titleController.text.trim(),
            description: _descController.text.trim(),
            userId: widget.userId,
          );
      Navigator.pop(context);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Tambah Task')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _titleController,
                decoration: const InputDecoration(
                  labelText: 'Judul Task *',
                  border: OutlineInputBorder(),
                ),
                validator: (v) =>
                    v == null || v.isEmpty ? 'Judul tidak boleh kosong' : null,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _descController,
                decoration: const InputDecoration(
                  labelText: 'Deskripsi (opsional)',
                  border: OutlineInputBorder(),
                ),
                maxLines: 3,
              ),
              const SizedBox(height: 24),
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _submit,
                  child: const Text('Simpan'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

*Lanjut ke [06_realtime_stream.md](06_realtime_stream.md) →*
