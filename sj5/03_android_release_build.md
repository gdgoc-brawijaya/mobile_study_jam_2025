# 3️⃣ Build Release Android: APK, AAB, dan Signing

## Gambaran Besar

Untuk Android, ada dua output utama:

| Output | Kegunaan |
|---|---|
| **APK** | Mudah dibagikan untuk testing / sideloading |
| **AAB** | Format yang disarankan untuk Google Play Store |

Untuk publish ke Play Store, biasanya kamu akan meng-upload **AAB**.

---

## 1. Build APK Release

```bash
flutter build apk --release
```

Hasil build biasanya ada di:

```text
build/app/outputs/flutter-apk/app-release.apk
```

APK cocok untuk:
- testing ke teman,
- demo ke panitia,
- atau distribusi manual.

---

## 2. Build App Bundle

```bash
flutter build appbundle --release
```

Hasilnya:

```text
build/app/outputs/bundle/release/app-release.aab
```

AAB adalah format yang disarankan karena:
- ukuran download ke user lebih efisien,
- lebih cocok untuk distribusi melalui Play Store,
- mendukung optimized delivery.

---

## 3. Signing Android Release

Aplikasi release Android harus ditandatangani.

### 3.1 Buat keystore

Jalankan perintah berikut:

```bash
keytool -genkey -v -keystore upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

Simpan file keystore di lokasi aman, misalnya:

```text
android/app/upload-keystore.jks
```

### 3.2 Buat `key.properties`

Buat file `android/key.properties`:

```properties
storePassword=PASSWORD_KAMU
keyPassword=PASSWORD_KAMU
keyAlias=upload
storeFile=app/upload-keystore.jks
```

> Jangan commit file ini ke repository publik.

### 3.3 Konfigurasi `android/app/build.gradle`

Tambahkan signing config:

```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

Jika project memakai Gradle Kotlin DSL, bentuknya sedikit berbeda.

---

## 4. Versioning Release

Setiap rilis harus punya versi yang jelas.

Contoh:

```yaml
version: 1.0.0+1
version: 1.0.1+2
version: 1.1.0+5
```

Gunakan format:
- patch untuk bug fix,
- minor untuk fitur baru,
- major untuk perubahan besar.

---

## 5. Uji Build Release Secara Lokal

Sebelum upload:

```bash
flutter clean
flutter pub get
flutter build apk --release
flutter build appbundle --release
```

Kalau perlu, install APK ke device:

```bash
adb install build/app/outputs/flutter-apk/app-release.apk
```

---

## 6. Masalah Release yang Umum

| Masalah | Penyebab | Solusi |
|---|---|---|
| Build gagal signing | Keystore salah / path salah | Cek `key.properties` |
| App crash saat release | Ada kode debug-only | Cek log dan test release |
| App beda dengan debug | Optimisasi release mempengaruhi flow | Uji ulang API, state, dan permission |
| Build lama / gagal cache | Cache rusak | Jalankan `flutter clean` |

---

## Checklist

- [ ] APK release berhasil dibuat
- [ ] AAB release berhasil dibuat
- [ ] Keystore sudah aman disimpan
- [ ] `key.properties` tidak ikut repository
- [ ] Versi app sudah dinaikkan
- [ ] Build release sudah diuji di device

---

*Study Jam 5 — Flutter Deployment*
