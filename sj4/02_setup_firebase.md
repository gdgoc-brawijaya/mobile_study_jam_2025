# 2️⃣ Membuat Project Firebase & Setup Console

## Overview

Sebelum kode Flutter bisa terhubung ke Firebase, kita perlu:
1. Membuat project di **Firebase Console**
2. Mengaktifkan layanan **Firestore** dan **Authentication**
3. Mendaftarkan aplikasi Android/iOS/Web ke project Firebase

---

## Step 1: Buka Firebase Console

1. Buka browser, pergi ke **[https://console.firebase.google.com](https://console.firebase.google.com)**
2. Login dengan akun Google kamu
3. Klik **"Add project"** atau **"Tambahkan project"**

---

## Step 2: Buat Project Firebase

### 2.1 Nama Project
- Masukkan nama project, contoh: `flutter-study-jam`
- Firebase akan otomatis generate **Project ID** unik, misalnya `flutter-study-jam-a1b2c`
- Klik **Continue**

> **Project ID** ini permanent dan tidak bisa diubah, jadi pilih nama yang deskriptif.

### 2.2 Google Analytics (Opsional)
- Untuk latihan, pilih **"Disable Google Analytics"** agar lebih simpel
- Klik **Create project**
- Tunggu beberapa detik hingga project siap
- Klik **Continue**

---

## Step 3: Aktifkan Cloud Firestore

1. Di sidebar kiri, klik **"Build"** → **"Firestore Database"**
2. Klik tombol **"Create database"**

### 3.1 Pilih Mode
Pilih **"Start in test mode"** untuk development awal:

```
○ Production mode
  Rules akan memblokir semua akses — perlu setup rules dulu

● Test mode  ← Pilih ini untuk latihan
  Semua orang bisa baca/tulis selama 30 hari
  (Akan kita amankan di materi Security Rules)
```

> ⚠️ **Jangan gunakan test mode di production!** Ini hanya untuk development.

### 3.2 Pilih Region
- Pilih region terdekat: **`asia-southeast2` (Jakarta)** atau **`asia-east1` (Taiwan)**
- Region tidak bisa diubah setelah dibuat
- Klik **Enable**

Setelah berhasil, kamu akan melihat tampilan Firestore dengan panel kosong.

---

## Step 4: Aktifkan Firebase Authentication

1. Di sidebar kiri, klik **"Build"** → **"Authentication"**
2. Klik **"Get started"**
3. Di tab **"Sign-in method"**, pilih **"Email/Password"**
4. Toggle **"Email/Password"** → **Enable**
5. Klik **Save**

---

## Step 5: Daftarkan Aplikasi Flutter (Android)

### 5.1 Tambah App Android
1. Di halaman utama project (klik logo Firebase di pojok kiri atas)
2. Klik ikon Android **`</>`** atau tombol **"Add app"**
3. Pilih platform **Android**

### 5.2 Isi Detail Aplikasi

**Android package name:**
Ini adalah package name dari project Flutter kamu. Cek di file `android/app/build.gradle`:
```groovy
defaultConfig {
    applicationId "com.example.flutter_firebase_app"  // ← ini
}
```
Atau cek di `android/app/src/main/AndroidManifest.xml`:
```xml
<manifest xmlns:android="..."
    package="com.example.flutter_firebase_app">   <!-- ← ini -->
```

**App nickname** (opsional): `Flutter Firebase App`

**Debug signing certificate SHA-1** (opsional untuk sekarang, wajib untuk Google Sign-In):
```bash
# Jalankan di terminal dari root project Flutter
cd android
./gradlew signingReport
# Windows:
gradlew.bat signingReport
```
Cari baris `SHA1:` di output.

### 5.3 Download `google-services.json`
- Klik **"Download google-services.json"**
- Simpan file ini ke: `android/app/google-services.json`

> **PENTING:** Jangan commit file `google-services.json` ke repository publik!
> Tambahkan ke `.gitignore`:
> ```
> android/app/google-services.json
> ```

### 5.4 Skip langkah selanjutnya
- Klik **Next** → **Next** → **Continue to console**
- Kita akan konfigurasi Gradle via FlutterFire CLI di langkah berikutnya

---

## Step 6: (Opsional) Daftarkan Aplikasi iOS

1. Klik **"Add app"** → pilih **iOS**
2. Masukkan **iOS bundle ID** dari `ios/Runner.xcodeproj/project.pbxproj`:
   ```
   PRODUCT_BUNDLE_IDENTIFIER = com.example.flutterFirebaseApp;
   ```
3. Download `GoogleService-Info.plist`
4. Simpan di `ios/Runner/GoogleService-Info.plist`

---

## Step 7: Mengenal Tampilan Firestore Console

Sekarang Firestore kamu masih kosong. Mari kenali navigasinya:

```
Firestore Database
├── Data tab          → Lihat & edit data Collection/Document secara manual
├── Rules tab         → Edit Security Rules
├── Indexes tab       → Composite indexes untuk query kompleks
└── Usage tab         → Monitor usage & quota
```

### Coba buat data manual (untuk latihan)
1. Klik **"Start collection"**
2. Collection ID: `tasks`
3. Document ID: (klik **"Auto-ID"**)
4. Tambahkan field:
   - `title` (string): `Belajar Firebase`
   - `isDone` (boolean): `false`
   - `createdAt` (timestamp): pilih tanggal sekarang
5. Klik **Save**

Kamu sudah punya data pertama di Firestore! Data ini nanti akan kita baca dari Flutter.

---

## Ringkasan Struktur Console

```
Firebase Console
└── flutter-study-jam (project)
    ├── Firestore Database
    │   └── tasks (collection)
    │       └── {doc-id} (document)
    │           ├── title: string
    │           ├── isDone: boolean
    │           └── createdAt: timestamp
    └── Authentication
        └── Sign-in providers
            └── Email/Password (enabled)
```

---

## Checklist Step 2

- [ ] Project Firebase sudah dibuat
- [ ] Firestore sudah aktif (test mode)
- [ ] Authentication Email/Password sudah aktif
- [ ] Aplikasi Android sudah didaftarkan
- [ ] `google-services.json` sudah di-download ke `android/app/`

---

*Lanjut ke [03_flutter_firebase_connect.md](03_flutter_firebase_connect.md) →*
