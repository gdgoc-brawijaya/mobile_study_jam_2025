# 5️⃣ Environment Config, Flavor, dan `--dart-define`

## Kenapa Environment Penting?

Aplikasi production biasanya punya beberapa environment:

- **development**: untuk ngoding dan testing lokal
- **staging**: untuk QA / demo internal
- **production**: untuk user final

Bedanya bisa ada di:
- base URL API,
- Firebase project,
- feature flag,
- analytics key,
- atau mode debug tertentu.

---

## 1. Hindari Hardcode

### Jangan seperti ini
```dart
const apiUrl = 'https://api.production.com';
```

### Lebih baik
```dart
const apiUrl = String.fromEnvironment('API_URL');
```

atau pakai wrapper config.

---

## 2. `--dart-define`

Flutter mendukung passing config dari command line.

```bash
flutter run --dart-define=API_URL=https://staging.example.com
flutter build apk --release --dart-define=API_URL=https://api.example.com
flutter build apk --release --dart-define=API_URL=https://api.example.com
```

Contoh baca nilainya di Dart:

```dart
class AppConfig {
  static const apiUrl = String.fromEnvironment(
    'API_URL',
    defaultValue: 'https://default.example.com',
  );
}
```

---

## 3. File Config Terpisah

Kalau project mulai besar, buat file config seperti:

```text
lib/core/config/app_config.dart
lib/core/config/app_environment.dart
```

Contoh:

```dart
enum AppEnvironment { development, staging, production }

class AppConfig {
  final AppEnvironment environment;
  final String apiUrl;

  const AppConfig({
    required this.environment,
    required this.apiUrl,
  });
}
```

---

## 4. Flavor Android

Kalau butuh build berbeda untuk staging dan production, kamu bisa pakai flavor.

Contoh konsep:
- `app-dev`
- `app-staging`
- `app-prod`

Manfaat:
- icon berbeda,
- nama app berbeda,
- endpoint berbeda,
- dan mudah dibedakan saat install.

---

## 5. Environment File

Jika project memakai `.env`, struktur sederhananya:

```env
API_URL=https://api.example.com
APP_NAME=Study Jam App
FEATURE_NEW_HOME=true
```

Lalu baca dengan package seperti `flutter_dotenv`.

> Pastikan file `.env` tidak ikut terkirim ke repository publik kalau berisi secret.

---

## 6. Praktik yang Aman

- API secret jangan ditulis langsung di source code.
- Simpan perbedaan environment di file config atau `--dart-define`.
- Jangan campur staging dan production credentials.
- Pastikan build pipeline memakai variable yang sesuai.

---

## Checklist

- [ ] Tidak ada hardcode endpoint production
- [ ] Config staging dan production sudah dipisah
- [ ] `.env` atau `--dart-define` sudah dipakai konsisten
- [ ] Build release membaca config yang benar

---

*Study Jam 5 — Flutter Deployment*
