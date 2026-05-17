# 2️⃣ Setup Project untuk Production-Ready Release

## Tujuan

Sebelum build release, project Flutter perlu dirapikan supaya aman dipakai di production.

---

## 1. Rapikan Informasi Aplikasi

Cek file-file dasar berikut:
- `pubspec.yaml`
- `lib/main.dart`
- `android/app/build.gradle`

Hal yang perlu dicek:
- nama aplikasi,
- versi aplikasi,
- ikon aplikasi,
- konfigurasi environment,
- dan package dependencies.

### Contoh versi di `pubspec.yaml`

```yaml
version: 1.0.0+1
```

Makna format:
- `1.0.0` = version name yang dilihat user
- `+1` = build number internal

Saat release berikutnya, biasanya angka build dinaikkan:

```yaml
version: 1.0.0+2
version: 1.0.1+3
```

---

## 2. Tambahkan File `.gitignore`

Pastikan file sensitif tidak ikut ter-commit.

```gitignore
# Android signing
android/key.properties
android/app/*.jks
android/app/*.keystore

# Environment
.env
*.env

# Build output
build/
.dart_tool/
```

---
## 3. Siapkan App Icon dan Splash Screen

Aplikasi release harus terlihat rapi sejak pertama dibuka.

Paket yang sering dipakai:
- `flutter_launcher_icons` untuk icon
- `flutter_native_splash` untuk splash screen

Contoh penggunaan:

```bash
flutter pub add --dev flutter_launcher_icons flutter_native_splash
```

Lalu konfigurasi di `pubspec.yaml` atau file konfigurasi masing-masing package.

---

## 4. Rapikan Logging

Di production, jangan terlalu banyak `print()`.

### Contoh yang kurang aman
```dart
print('token: $token');
```

### Lebih baik
```dart
if (kDebugMode) {
  print('token: $token');
}
```

Atau gunakan logger yang bisa dimatikan di release.

---

## 5. Aktifkan Flavor Jika Perlu

Kalau project punya beberapa environment, gunakan:
- `development`
- `staging`
- `production`

Contoh sederhana:
- API staging untuk testing,
- API production untuk user final.

Kita akan bahas lebih lengkap di materi environment config.

### Kenapa perlu 2 flavor (dev & prod)?

Dengan dua flavor/variant, kamu bisa punya 2 aplikasi yang benar-benar terpisah:
- **Dev**: aman buat testing (API staging, logging aktif, icon/nama beda).
- **Prod**: stabil buat user (API production, logging minimal, package name final).

Manfaat utama:
- Mencegah data staging tercampur dengan data produksi.
- Bisa install **dua aplikasi bersamaan** di satu device.
- Rilis lebih aman karena config dan key bisa dipisah.

### Contoh setup flavor dev & prod (Android)

1) **Buat entrypoint terpisah**

`lib/main_dev.dart`
```dart
import 'package:flutter/material.dart';
import 'main.dart' as app;

void main() {
  const env = 'dev';
  app.startApp(env: env);
}
```

`lib/main_prod.dart`
```dart
import 'package:flutter/material.dart';
import 'main.dart' as app;

void main() {
  const env = 'prod';
  app.startApp(env: env);
}
```

Di `lib/main.dart`, siapkan fungsi entry yang bisa dipakai kedua flavor:
```dart
import 'package:flutter/material.dart';

void startApp({required String env}) {
  // Gunakan env untuk memilih baseUrl, config, dll.
  runApp(MyApp(env: env));
}
```

2) **Tambahkan product flavors di `android/app/build.gradle`**

```gradle
android {
  flavorDimensions "env"

  productFlavors {
    dev {
      dimension "env"
      applicationIdSuffix ".dev"
      versionNameSuffix "-dev"
      resValue "string", "app_name", "MyApp Dev"
    }
    prod {
      dimension "env"
      resValue "string", "app_name", "MyApp"
    }
  }
}
```

3) **Jalankan atau build dengan flavor**

```bash
flutter run --flavor dev -t lib/main_dev.dart
flutter run --flavor prod -t lib/main_prod.dart

flutter build apk --flavor prod -t lib/main_prod.dart
```

Catatan:
- `applicationIdSuffix` membuat package name beda, jadi dev & prod bisa diinstall bareng.
- `resValue app_name` bikin nama app beda di launcher.
- Untuk iOS, konsepnya sama tapi setup dilakukan di Xcode (target & scheme).

---

## 6. Dependency yang Sering Dipakai untuk Release

```bash
flutter pub add package_info_plus
flutter pub add flutter_dotenv
flutter pub add --dev flutter_launcher_icons flutter_native_splash
```

Fungsi umum:
- `package_info_plus` untuk baca versi app
- `flutter_dotenv` untuk config environment
- launcher icons dan splash untuk polish UI

---

## 7. Cek Kesiapan Build

Jalankan beberapa perintah berikut:

```bash
flutter pub get
flutter analyze
flutter test
```

Kalau semua lolos, baru lanjut ke build release.

---

## Checklist

- [ ] Versi app sudah dinaikkan
- [ ] File sensitif sudah di-`gitignore`
- [ ] App icon dan splash screen sudah siap
- [ ] Logging tidak berlebihan
- [ ] Test dasar sudah dijalankan
- [ ] Environment config sudah ditentukan
- [ ] Flavor dev/prod sudah dipisah

---

*Study Jam 5 — Flutter Deployment*
