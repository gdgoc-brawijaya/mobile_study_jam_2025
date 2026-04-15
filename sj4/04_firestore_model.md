# 4️⃣ Struktur Data Firestore & Model Flutter

## Konsep Struktur Data Firestore

Firestore adalah database **NoSQL berbasis dokumen**. Berbeda dengan database relasional (SQL), Firestore tidak menggunakan tabel dan baris — melainkan **Collection** dan **Document**.

---

## Analogi: SQL vs Firestore

| SQL | Firestore |
|---|---|
| Database | Firebase Project |
| Tabel | Collection |
| Row | Document |
| Column | Field |
| Primary Key | Document ID |
| Foreign Key | Reference Field |

---

## Struktur Collection & Document

Kita akan membangun aplikasi **Task Manager** sederhana.

```
Firestore
└── tasks (Collection)
    ├── KJh2nXqmEabc (Document)
    │   ├── id: "KJh2nXqmEabc"       ← String
    │   ├── title: "Belajar Firebase"  ← String
    │   ├── description: "FirestoreCRUD" ← String
    │   ├── isDone: false              ← Boolean
    │   ├── createdAt: Timestamp       ← Timestamp
    │   └── userId: "user_uid_123"     ← String (relasi ke Auth)
    │
    └── Pq9rTvLmnDef (Document)
        ├── id: "Pq9rTvLmnDef"
        ├── title: "Bikin UI Flutter"
        ├── description: ""
        ├── isDone: true
        ├── createdAt: Timestamp
        └── userId: "user_uid_123"
```

---

## Tipe Data yang Didukung Firestore

| Tipe Firestore | Tipe Dart |
|---|---|
| `String` | `String` |
| `Number` (Integer) | `int` |
| `Number` (Float) | `double` |
| `Boolean` | `bool` |
| `Timestamp` | `DateTime` (via konversi) |
| `Array` | `List` |
| `Map` | `Map<String, dynamic>` |
| `Null` | `null` |
| `Reference` | `DocumentReference` |
| `GeoPoint` | `GeoPoint` |

---

## Membuat Model Dart

Kita akan menggunakan **Freezed** untuk membuat model yang immutable dan aman.

### Buat file `lib/data/models/task_model.dart`:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:freezed_annotation/freezed_annotation.dart';

part 'task_model.freezed.dart';
part 'task_model.g.dart';

@freezed
class TaskModel with _$TaskModel {
  const factory TaskModel({
    required String id,
    required String title,
    @Default('') String description,
    @Default(false) bool isDone,
    required DateTime createdAt,
    required String userId,
  }) = _TaskModel;

  // fromJson untuk JSON biasa
  factory TaskModel.fromJson(Map<String, dynamic> json) =>
      _$TaskModelFromJson(json);

  // fromFirestore: konversi dari DocumentSnapshot Firestore
  factory TaskModel.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return TaskModel(
      id: doc.id,
      title: data['title'] as String,
      description: data['description'] as String? ?? '',
      isDone: data['isDone'] as bool? ?? false,
      // Firestore Timestamp → DateTime
      createdAt: (data['createdAt'] as Timestamp).toDate(),
      userId: data['userId'] as String,
    );
  }
}

// Extension untuk konversi ke Map (untuk write ke Firestore)
extension TaskModelX on TaskModel {
  Map<String, dynamic> toFirestore() {
    return {
      'title': title,
      'description': description,
      'isDone': isDone,
      // DateTime → Firestore Timestamp
      'createdAt': Timestamp.fromDate(createdAt),
      'userId': userId,
      // Catatan: 'id' tidak disimpan sebagai field,
      // karena sudah jadi Document ID
    };
  }
}
```

### Generate kode Freezed:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Ini akan membuat dua file otomatis:
- `task_model.freezed.dart` — implementasi immutable class
- `task_model.g.dart` — serialisasi JSON

---

## Memahami Konversi Timestamp

Firestore menyimpan waktu sebagai `Timestamp`, bukan `DateTime` Dart. Kita perlu konversi:

```dart
// Firestore Timestamp → Dart DateTime
final DateTime dartDate = (data['createdAt'] as Timestamp).toDate();

// Dart DateTime → Firestore Timestamp
final Timestamp firestoreTs = Timestamp.fromDate(DateTime.now());

// Gunakan FieldValue.serverTimestamp() untuk waktu server
// (lebih akurat dari DateTime.now() karena konsisten)
final serverTime = FieldValue.serverTimestamp();
```

> **Best Practice:** Selalu gunakan `FieldValue.serverTimestamp()` saat *create* dokumen baru, bukan `DateTime.now()` — karena jam perangkat user bisa tidak akurat.

---

## Membuat Repository

Repository adalah lapisan abstraksi antara Cubit dan Firestore SDK.

### Buat file `lib/data/repositories/task_repository.dart`:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/task_model.dart';

class TaskRepository {
  final FirebaseFirestore _firestore;

  // Collection reference — bisa di-reuse
  CollectionReference<Map<String, dynamic>> get _tasksCollection =>
      _firestore.collection('tasks');

  TaskRepository({FirebaseFirestore? firestore})
      : _firestore = firestore ?? FirebaseFirestore.instance;

  // ── CREATE ──────────────────────────────────────────────────
  Future<void> addTask(TaskModel task) async {
    await _tasksCollection.add(task.toFirestore());
  }

  // ── READ (one-time) ─────────────────────────────────────────
  Future<List<TaskModel>> getTasks(String userId) async {
    final snapshot = await _tasksCollection
        .where('userId', isEqualTo: userId)
        .orderBy('createdAt', descending: true)
        .get();

    return snapshot.docs
        .map((doc) => TaskModel.fromFirestore(doc))
        .toList();
  }

  // ── READ (real-time stream) ──────────────────────────────────
  Stream<List<TaskModel>> watchTasks(String userId) {
    return _tasksCollection
        .where('userId', isEqualTo: userId)
        .orderBy('createdAt', descending: true)
        .snapshots()
        .map((snapshot) =>
            snapshot.docs.map((doc) => TaskModel.fromFirestore(doc)).toList());
  }

  // ── UPDATE ──────────────────────────────────────────────────
  Future<void> updateTask(TaskModel task) async {
    await _tasksCollection.doc(task.id).update(task.toFirestore());
  }

  // ── DELETE ──────────────────────────────────────────────────
  Future<void> deleteTask(String taskId) async {
    await _tasksCollection.doc(taskId).delete();
  }

  // ── TOGGLE isDone ───────────────────────────────────────────
  Future<void> toggleTaskDone(String taskId, bool currentStatus) async {
    await _tasksCollection.doc(taskId).update({
      'isDone': !currentStatus,
    });
  }
}
```

---

## Struktur Folder Lengkap

```
lib/
├── data/
│   ├── models/
│   │   ├── task_model.dart
│   │   ├── task_model.freezed.dart   ← auto-generated
│   │   └── task_model.g.dart         ← auto-generated
│   └── repositories/
│       └── task_repository.dart
├── presentation/
│   ├── bloc/
│   │   └── task_cubit.dart
│   └── pages/
│       └── home_page.dart
├── core/
│   └── di/
│       └── injection.dart
├── firebase_options.dart
└── main.dart
```

---

## Tips: Menggunakan `withConverter`

Cara lebih type-safe untuk membaca/menulis Firestore:

```dart
// Buat typed collection reference
final tasksRef = FirebaseFirestore.instance
    .collection('tasks')
    .withConverter<TaskModel>(
      fromFirestore: (snapshot, _) => TaskModel.fromFirestore(snapshot),
      toFirestore: (task, _) => task.toFirestore(),
    );

// Sekarang type-safe!
final QuerySnapshot<TaskModel> snapshot = await tasksRef.get();
final List<TaskModel> tasks = snapshot.docs.map((d) => d.data()).toList();
```

---

*Lanjut ke [05_firestore_crud.md](05_firestore_crud.md) →*
