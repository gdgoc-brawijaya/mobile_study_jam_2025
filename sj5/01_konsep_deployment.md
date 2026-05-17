# 1️⃣ Konsep Deployment & Release Flow

## Apa Itu Deployment?

**Deployment** adalah proses membawa aplikasi dari environment development ke environment yang bisa dipakai user, misalnya:
- **Local development** → aplikasi di laptop kita
- **Staging** → versi uji sebelum rilis
- **Production** → versi yang dipakai user nyata

Kalau di Flutter, deployment bukan cuma "klik upload". Kita perlu memastikan aplikasi:
- sudah di-build dengan mode yang benar,
- aman untuk production,
- punya konfigurasi environment yang tepat,
- dan bisa dipantau setelah dirilis.

---

## Tahapan Umum Release

```text
Code -> Test -> Build -> Sign -> Upload -> Review -> Release -> Monitor
```

Penjelasan singkat:

1. **Code**: fitur sudah selesai di branch tertentu.
2. **Test**: unit test, widget test, dan smoke test dijalankan.
3. **Build**: Flutter menghasilkan file release.
4. **Sign**: aplikasi Android ditandatangani dengan keystore.
5. **Upload**: file APK/AAB diunggah.
6. **Review**: Play Console memproses artefak.
7. **Release**: aplikasi bisa diakses user.
8. **Monitor**: pantau crash, error, dan performa.

---

## Mode di Flutter

Flutter punya tiga mode utama:

| Mode | Tujuan | Ciri |
|---|---|---|
| **Debug** | Development harian | Hot reload, lebih lambat, banyak assert |
| **Profile** | Performance profiling | Cocok untuk cek performa |
| **Release** | Production | Optimized, minim log, siap rilis |

Contoh:

```bash
flutter run              # biasanya debug
flutter run --profile    # profiling
flutter build apk --release
```

---

## Target Deployment Android

Untuk study jam ini, fokus utama kita adalah:
- **Android release**
- **APK / AAB distribution**
- **Play Console release flow**
- **Pipeline release otomatis**

---

## Kenapa Release Harus Direncanakan?

Kalau deployment dilakukan tanpa persiapan, biasanya masalahnya muncul di:
- app crash setelah install,
- API endpoint salah,
- build gagal karena signing,
- file konfigurasi ikut ter-commit,
- atau user melihat versi yang belum siap.

Karena itu, workflow deployment yang sehat perlu:
- branch strategy,
- environment config,
- versioning,
- release checklist,
- dan monitoring.

---

## Checklist Awal

Sebelum mulai build release, pastikan:

- [x] App sudah jalan stabil di mode debug
- [ ] Tidak ada error di console saat membuka halaman utama
- [ ] Config API / Firebase / endpoint sudah siap untuk production
- [ ] Icon, splash screen, dan nama app sudah sesuai
- [ ] Versi app sudah dinaikkan
- [ ] Screenshot, listing, dan deskripsi store sudah disiapkan jika perlu publish

---

## Output yang Akan Kita Buat

Di modul-modul berikutnya, kita akan menyiapkan:

- release build Android,
- signing dengan keystore,
- distribusi Android,
- environment config per stage,
- dan CI/CD sederhana dengan GitHub Actions.

---

*Study Jam 5 — Flutter Deployment*
