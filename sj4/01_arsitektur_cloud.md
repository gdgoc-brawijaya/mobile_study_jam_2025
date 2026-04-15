# 1️⃣ Konsep Cloud Architecture & Firebase Ecosystem

## Mengapa Cloud Architecture?

Bayangkan aplikasi To-Do sederhana:
- User A menambahkan task di HP-nya
- User B (partner-nya) ingin melihat task tersebut di HP-nya
- Bagaimana data bisa **sinkron** tanpa keduanya menekan tombol refresh?

Jawabannya: **Cloud Architecture** dengan **Real-time Database**.

---

## Tradisional vs Cloud Architecture

### Arsitektur Tradisional (REST API)
```
Flutter App ──── HTTP Request ────► Backend Server ──── Query ────► Database
Flutter App ◄─── HTTP Response ───  Backend Server ◄─── Result ───  Database
```
- Data hanya diperbarui saat user **meminta** (pull)
- Butuh polling atau WebSocket manual untuk real-time
- Kamu harus maintain server sendiri

### Cloud Architecture (Firebase)
```
Flutter App ◄──── Stream / Snapshot ────► Firestore (Cloud)
Flutter App ─────── Write Data ─────────► Firestore (Cloud)
                                              │
Flutter App B ◄──── Auto Update ────────────┘
```
- Data **otomatis terpush** ke semua device yang subscribe
- Tidak perlu server sendiri — Firebase mengurus skalabilitas
- Tersedia **offline support** built-in

---

## Big Picture: Firebase Ecosystem

```
  ┌─────────────────────────────────────────────────────┐
  │                  FIREBASE PLATFORM                   │
  │                                                     │
  │  ┌─────────────┐  ┌──────────────┐  ┌───────────┐  │
  │  │  Firestore   │  │  Auth        │  │ Storage   │  │
  │  │  (Database)  │  │  (Login)     │  │ (Files)   │  │
  │  └──────┬───────┘  └──────┬───────┘  └─────┬─────┘  │
  │         └─────────────────┴──────────────────┘       │
  │                           │                          │
  │              Firebase SDK (firebase_core)             │
  └───────────────────────────┬─────────────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │   Flutter App       │
                    │                    │
                    │  ┌──────────────┐  │
                    │  │  Repository  │  │
                    │  └──────┬───────┘  │
                    │         │          │
                    │  ┌──────▼───────┐  │
                    │  │ Bloc / Cubit │  │
                    │  └──────┬───────┘  │
                    │         │          │
                    │  ┌──────▼───────┐  │
                    │  │   UI Widget  │  │
                    │  └──────────────┘  │
                    └────────────────────┘
```

---

## Firebase Services yang Kita Gunakan

### 1. Cloud Firestore
Database NoSQL berbasis dokumen yang tersimpan di cloud.

**Analogi:** Bayangkan seperti Google Drive berisi banyak folder (Collection) dan di dalamnya ada file-file (Document). Setiap file berisi data (Field).

```
Firestore
└── tasks (Collection)
    ├── abc123 (Document)
    │   ├── title: "Belajar Firebase"
    │   ├── isDone: false
    │   └── createdAt: Timestamp
    └── def456 (Document)
        ├── title: "Bikin app Flutter"
        ├── isDone: true
        └── createdAt: Timestamp
```

**Keunggulan Firestore:**
- Real-time listeners — data update otomatis
- Offline support — bisa pakai saat internet off, sync saat online
- Skalabel secara otomatis
- Keamanan via Security Rules

### 2. Firebase Authentication
Sistem autentikasi siap pakai tanpa perlu backend sendiri.

**Metode yang didukung:**
- Email & Password
- Google Sign-In
- Phone OTP
- Anonymous Login
- GitHub, Twitter, dll.

### 3. Firebase Storage
Menyimpan file besar (foto, video, dokumen) di cloud.

> Pada modul ini, fokus kita ada di **Firestore** dan **Authentication**.

---

## Pola Aliran Data di Flutter + Firebase

```
User action (tap button)
        │
        ▼
   UI Widget
        │  event
        ▼
  Bloc / Cubit
        │  call
        ▼
   Repository
        │  call Firestore SDK
        ▼
  Cloud Firestore ──────────────────────────────────┐
        │                                           │
        │  Future (one-time)   Stream (real-time)   │
        ▼                              │            │
   Repository                         │            │
        │                             │            │
        ▼                             ▼            │
  Bloc / Cubit ◄──────── StreamSubscription        │
        │                                          │
        ▼                                          │
   State (emit)                                    │
        │                                          │
        ▼                                          │
   BlocBuilder                         Other devices
        │                              auto-updated │
        ▼                                          │
   UI rebuild ◄────────────────────────────────────┘
```

---

## Firestore vs Realtime Database

Firebase punya dua produk database. Gunakan **Cloud Firestore** (bukan Realtime Database lama):

| Fitur | Cloud Firestore | Realtime Database |
|---|---|---|
| Struktur data | Collection/Document | JSON tree |
| Query | Powerful, multi-field | Terbatas |
| Offline support | ✅ | ✅ |
| Skalabilitas | Sangat baik | Terbatas |
| Pricing | Per read/write/delete | Per bandwidth |
| **Rekomendasi** | ✅ Gunakan ini | ❌ Legacy |

---

## Istilah Penting

| Istilah | Penjelasan |
|---|---|
| **Collection** | Kumpulan dokumen (seperti tabel di SQL) |
| **Document** | Satu unit data berisi field-field (seperti row di SQL) |
| **Field** | Key-value pair dalam dokumen |
| **DocumentSnapshot** | Hasil pembacaan satu dokumen |
| **QuerySnapshot** | Hasil query yang mengembalikan banyak dokumen |
| **Stream** | Aliran data real-time yang terus update |
| **Security Rules** | Aturan siapa yang boleh baca/tulis data |

---

*Lanjut ke [02_setup_firebase.md](02_setup_firebase.md) →*
