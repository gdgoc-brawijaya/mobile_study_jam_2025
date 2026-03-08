# 1️Konsep Arsitektur Data Flutter

## Mengapa Butuh Arsitektur?

Bayangkan kamu membangun aplikasi tanpa arsitektur:
- Widget `ListView` langsung memanggil `http.get()`
- Data tersebar di mana-mana
- Sulit di-*test*, sulit di-*maintain*

Arsitektur yang baik memisahkan **siapa yang mengambil data**, **siapa yang menyimpan sementara**, dan **siapa yang menampilkan**.

---

## Big Picture: Aliran Data

```
  ┌─────────────────────────────────────────┐
  │            INTERNET / REST API           │
  └──────────────┬──────────────────────────┘
                 │  HTTP Request (Dio)
                 ▼
  ┌─────────────────────────────────────────┐
  │           DATA LAYER                     │
  │  ┌──────────────┐  ┌─────────────────┐  │
  │  │  API Service  │  │  Local Storage  │  │
  │  │    (Dio)      │  │ (SharedPrefs)   │  │
  │  └──────┬───────┘  └────────┬────────┘  │
  │         └──────────┬────────┘           │
  │                    ▼                    │
  │            Repository                   │
  └──────────────────┬──────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────────┐
  │           DOMAIN LAYER                   │
  │         Business Logic / UseCase         │
  └──────────────────┬──────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────────┐
  │        PRESENTATION LAYER                │
  │  ┌──────────────┐  ┌─────────────────┐  │
  │  │ Bloc / Cubit  │  │   UI / Widget   │  │
  │  │  (State)      │◄─►  (BlocBuilder) │  │
  │  └──────────────┘  └─────────────────┘  │
  └─────────────────────────────────────────┘
```

---

## 3 Layer Utama

### 1. Data Layer (Sumber Data)
Bertanggung jawab mengambil dan menyimpan data.
- **`ApiService`** → Berinteraksi dengan server via Dio
- **`LocalStorageService`** → Berinteraksi dengan Shared Preferences
- **`Repository`** → Menggabungkan kedua sumber data di atas

### 2. Domain Layer (Otak Bisnis)
Berisi logika utama aplikasi, **tidak bergantung** pada Flutter maupun package eksternal.
- **`Entity`** → Model data murni (plain Dart class)
- **`Repository Interface`** → Kontrak yang harus dipenuhi Data Layer

### 3. Presentation Layer (Tampilan)
Bertanggung jawab menampilkan data dan menerima input user.
- **`Cubit/Bloc`** → Mengelola state UI
- **`Page/Widget`** → Menampilkan data dari state

---

## Alur Ketika User Membuka Halaman

```
User buka halaman
      │
      ▼
  UserPage dibuat
      │
      ▼
  UserCubit.fetchUsers() dipanggil
      │
      ▼
  State berubah: UserLoading
      │
      ▼
  UserRepository.getUsers() dipanggil
      │
      ├── [Cache ada?] ──Yes──► Ambil dari SharedPreferences
      │                              │
      └── [Cache kosong] ──────► Fetch dari API (Dio)
                                     │
                                     ▼
                              Simpan ke cache (SharedPrefs)
                                     │
                                     ▼
                         State berubah: UserLoaded(data)
                                     │
                                     ▼
                            BlocBuilder rebuild UI
                                     │
                                     ▼
                           ListView tampil 
```

---

## Prinsip SOLID yang Diterapkan

| Prinsip | Penerapan |
|---|---|
| **S** - Single Responsibility | Setiap class punya 1 tugas: `ApiService` hanya fetch, `Cubit` hanya kelola state |
| **O** - Open/Closed | Bisa tambah fitur baru tanpa mengubah kode yang sudah ada |
| **D** - Dependency Inversion | `Cubit` bergantung pada interface `Repository`, bukan implementasi langsung |

---

## Takeaway

> **"UI tidak boleh tahu dari mana data berasal — dari internet atau dari cache. Itu urusan Repository."**

Dengan prinsip ini, kita bisa dengan mudah:
- Mengganti Dio dengan `http` tanpa menyentuh UI
- Mengganti SharedPrefs dengan Hive tanpa menyentuh Cubit
- Men-*test* setiap layer secara terpisah

---

**Selanjutnya: [02_setup_project.md](02_setup_project.md)**