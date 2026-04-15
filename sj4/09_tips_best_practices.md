# 9️⃣ Tips, Security Rules & Best Practices

## 1. Firestore Security Rules

Security Rules adalah lapisan keamanan paling penting di Firestore. Tanpanya, **siapapun bisa membaca dan menulis data kamu** di test mode.

### Akses Rules di Firebase Console
1. Firestore Database → tab **Rules**
2. Edit langsung di editor online

### Sintaks Dasar Rules

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Format dasar:
    // match /{path} {
    //   allow [operation]: if [condition];
    // }

    // Izinkan semua (JANGAN di production!)
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

### Rules untuk Aplikasi Task Manager

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Collection: tasks
    match /tasks/{taskId} {

      // Hanya user yang sudah login yang bisa membaca
      // dan hanya bisa membaca task miliknya sendiri
      allow read: if request.auth != null
                  && resource.data.userId == request.auth.uid;

      // Hanya user yang sudah login yang bisa membuat task
      // dan task harus milik user tersebut (userId harus match)
      allow create: if request.auth != null
                    && request.resource.data.userId == request.auth.uid
                    && request.resource.data.title is string
                    && request.resource.data.title.size() > 0
                    && request.resource.data.title.size() <= 200;

      // Hanya pemilik yang bisa update task-nya
      allow update: if request.auth != null
                    && resource.data.userId == request.auth.uid;

      // Hanya pemilik yang bisa hapus task-nya
      allow delete: if request.auth != null
                    && resource.data.userId == request.auth.uid;
    }
  }
}
```

### Variabel Penting di Rules

| Variabel | Deskripsi |
|---|---|
| `request.auth` | Info user yang sedang request (null jika belum login) |
| `request.auth.uid` | UID user yang request |
| `resource.data` | Data dokumen yang *sudah ada* (untuk read/update/delete) |
| `request.resource.data` | Data dokumen yang *akan ditulis* (untuk create/update) |
| `request.time` | Waktu request |

### Fungsi Helper di Rules

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Fungsi helper — bisa dipanggil di mana saja
    function isAuthenticated() {
      return request.auth != null;
    }

    function isOwner(userId) {
      return request.auth.uid == userId;
    }

    function isValidTask() {
      let data = request.resource.data;
      return data.title is string
          && data.title.size() > 0
          && data.title.size() <= 200
          && data.isDone is bool
          && data.userId == request.auth.uid;
    }

    match /tasks/{taskId} {
      allow read: if isAuthenticated() && isOwner(resource.data.userId);
      allow create: if isAuthenticated() && isValidTask();
      allow update: if isAuthenticated() && isOwner(resource.data.userId);
      allow delete: if isAuthenticated() && isOwner(resource.data.userId);
    }
  }
}
```

---

## 2. Dependency Injection Setup

Buat `lib/core/di/injection.dart`:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:get_it/get_it.dart';
import '../../data/repositories/auth_repository.dart';
import '../../data/repositories/task_repository.dart';
import '../../presentation/bloc/auth_cubit.dart';
import '../../presentation/bloc/task_cubit.dart';

final getIt = GetIt.instance;

void setupDependencies() {
  // Firebase instances
  getIt.registerLazySingleton<FirebaseFirestore>(
      () => FirebaseFirestore.instance);
  getIt.registerLazySingleton<FirebaseAuth>(() => FirebaseAuth.instance);

  // Repositories
  getIt.registerLazySingleton<AuthRepository>(
      () => AuthRepository(auth: getIt<FirebaseAuth>()));
  getIt.registerLazySingleton<TaskRepository>(
      () => TaskRepository(firestore: getIt<FirebaseFirestore>()));

  // Cubits
  getIt.registerFactory<AuthCubit>(
      () => AuthCubit(getIt<AuthRepository>()));
  getIt.registerFactory<TaskCubit>(
      () => TaskCubit(getIt<TaskRepository>()));
}
```

---

## 3. Checklist Keamanan

Sebelum deploy ke production, pastikan:

### Firebase Console
- [ ] Security Rules sudah diubah dari test mode
- [ ] Rules sudah di-test menggunakan **Rules Playground** di Firebase Console
- [ ] Authentication providers hanya yang dibutuhkan yang aktif
- [ ] Authorized domains dibatasi (untuk Web)

### Kode
- [ ] `google-services.json` ada di `.gitignore`
- [ ] `GoogleService-Info.plist` ada di `.gitignore`
- [ ] API Keys sensitif tidak hardcoded di kode
- [ ] `userId` selalu divalidasi sebelum operasi Firestore
- [ ] Input user di-sanitize / di-validate sebelum dikirim ke Firestore

### `.gitignore` yang Baik
```gitignore
# Firebase
android/app/google-services.json
ios/Runner/GoogleService-Info.plist

# Environment
.env
*.env

# Build artifacts
build/
.dart_tool/
```

---

## 4. Tips Performa Firestore

### Hindari Membaca Seluruh Collection
```dart
// ❌ BURUK — membaca semua dokumen
await firestore.collection('tasks').get();

// ✅ BAIK — filter dengan query
await firestore.collection('tasks')
    .where('userId', isEqualTo: currentUserId)
    .limit(20)
    .get();
```

### Buat Composite Index untuk Query Kompleks
Query dengan lebih dari satu `where` + `orderBy` butuh **Composite Index**.

Firebase biasanya memberikan error dengan link langsung untuk membuat index:
```
[cloud_firestore/failed-precondition] The query requires an index.
You can create it here: https://console.firebase.google.com/...
```
Klik link tersebut → Firebase Console akan otomatis membuat index.

### Gunakan Pagination untuk List Panjang
```dart
// Jangan load semua sekaligus
// Gunakan cursor-based pagination
QueryDocumentSnapshot? _lastDocument;

Future<List<TaskModel>> loadNextPage() async {
  Query<Map<String, dynamic>> query = _tasksCollection
      .where('userId', isEqualTo: userId)
      .orderBy('createdAt', descending: true)
      .limit(10);

  if (_lastDocument != null) {
    query = query.startAfterDocument(_lastDocument!);
  }

  final snapshot = await query.get();
  if (snapshot.docs.isNotEmpty) {
    _lastDocument = snapshot.docs.last;
  }
  return snapshot.docs.map((d) => TaskModel.fromFirestore(d)).toList();
}
```

### Cache Identitas Field
```dart
// ❌ Buat reference baru setiap kali (tidak efisien)
FirebaseFirestore.instance.collection('tasks').doc(id).update(data);

// ✅ Simpan reference sebagai field
final _tasksCollection = FirebaseFirestore.instance.collection('tasks');
_tasksCollection.doc(id).update(data);
```

---

## 5. Tips Struktur Data Firestore

### Denormalisasi Data
Firestore tidak mendukung JOIN seperti SQL. Kadang lebih efisien menyimpan data duplikat:

```
// Daripada relasi seperti SQL:
users/{userId} → { name, email }
tasks/{taskId} → { title, userId }  ← harus fetch user lagi untuk nama

// Denormalisasi — simpan nama user langsung di task:
tasks/{taskId} → { title, userId, userName, userAvatar }
// Trade-off: data bisa stale jika user update profilnya
```

### Sub-Collection untuk Data Berjenjang
```
users/{userId}/           ← Collection
    profile/              ← Sub-collection
    tasks/{taskId}/       ← Sub-collection
        subtasks/{subId}/ ← Sub-sub-collection
```

Sub-collection bagus untuk:
- Data yang hanya relevan dalam konteks parent
- Data yang banyak (misal: comments pada post)
- Memudahkan Security Rules per user

---

## 6. Ringkasan Alur Belajar

```
Materi 1: Konsep Cloud Architecture
    ↓
Materi 2: Firebase Console Setup
    ↓
Materi 3: FlutterFire CLI → flutter connect ke Firebase
    ↓
Materi 4: Data Model (Freezed) + Repository
    ↓
Materi 5: CRUD Firestore (Create/Read/Update/Delete)
    ↓
Materi 6: Real-time Stream + BlocBuilder
    ↓
Materi 7: Firebase Auth (Login/Register)
    ↓
Materi 8: Error Handling + Offline Support
    ↓
Materi 9: Security Rules + Best Practices ← kamu di sini
```

---

## 7. Checklist Akhir Project

### Fungsionalitas
- [ ] User bisa Register & Login
- [ ] User bisa Logout
- [ ] User bisa Create task baru
- [ ] User bisa Read semua task-nya (dengan filter)
- [ ] User bisa Update task (edit judul, toggle done)
- [ ] User bisa Delete task
- [ ] Data real-time (update otomatis saat ada perubahan)
- [ ] App bekerja saat offline (cache Firestore)

### Kualitas
- [ ] Loading state ditampilkan saat fetch data
- [ ] Error ditampilkan dengan pesan yang jelas
- [ ] Tombol retry tersedia saat terjadi error
- [ ] Firestore Security Rules sudah diamankan

### Code Quality
- [ ] Repository memisahkan logic Firestore dari UI
- [ ] Cubit/Bloc memisahkan business logic dari widget
- [ ] Tidak ada hardcoded string untuk collection name
- [ ] Stream subscription di-cancel saat widget/cubit di-dispose

---

## 8. Langkah Selanjutnya

Setelah menguasai dasar-dasar ini, kamu bisa eksplorasi:

| Fitur | Package / Layanan |
|---|---|
| Google Sign-In | `google_sign_in` |
| Upload foto | `firebase_storage` |
| Push notification | `firebase_messaging` |
| Analytics | `firebase_analytics` |
| Remote config | `firebase_remote_config` |
| Crash reporting | `firebase_crashlytics` |
| Cloud Functions | Firebase Functions (Node.js) |

---

*Selesai! Kamu sudah menguasai dasar Flutter + Firebase Cloud Architecture. 🎉*

*GDG on Campus Brawijaya — Tech Series Study Jam 2025*
