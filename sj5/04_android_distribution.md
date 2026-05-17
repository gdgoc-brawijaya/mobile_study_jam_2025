# 4️⃣ Distribusi Android: Internal Testing, Closed Testing, dan Release

## Alur Distribusi

Untuk Android, aplikasi biasanya tidak langsung dipublish ke production. Alur yang lebih aman adalah:

```text
Build APK/AAB -> Upload ke Play Console -> Uji testing track -> Release production
```

---

## 1. Build Artifact untuk Distribusi

### APK untuk testing cepat
```bash
flutter build apk --release
```

APK cocok untuk:
- instalasi manual di device,
- pengujian internal,
- demo cepat ke tim.

### AAB untuk Play Store
```bash
flutter build appbundle --release
```

AAB adalah format utama untuk distribusi resmi di Play Console.

---

## 2. Internal Testing Track

Internal testing cocok untuk:
- tim kecil,
- cek build sebelum masuk review,
- validasi UI dan flow penting.

Langkah umum di Play Console:
1. Buka app di Play Console.
2. Masuk ke menu **Testing**.
3. Pilih **Internal testing**.
4. Upload file AAB.
5. Tambahkan tester melalui email atau Google Group.

---

## 3. Closed Testing

Closed testing berguna untuk:
- alpha/beta tester,
- validasi fitur baru,
- cek stabilitas sebelum production.

Keuntungannya:
- akses tester dibatasi,
- feedback lebih terkontrol,
- lebih aman dibanding production langsung.

---

## 4. Production Release

Setelah build stabil, lanjutkan ke production.

Hal yang perlu dicek:
- app icon sudah benar,
- nama package tidak berubah,
- build number naik,
- signing release valid,
- deskripsi dan screenshot sudah diisi.

---

## 5. Android App Bundle Checklist

Sebelum upload AAB:
- [ ] `versionCode` naik
- [ ] `versionName` sesuai changelog
- [ ] `key.properties` aman dan tidak di-commit
- [ ] Release build sudah dites di device
- [ ] Tidak ada error pada permission atau dependency

---

## 6. Troubleshooting Distribusi

| Masalah | Penyebab | Solusi |
|---|---|---|
| Upload ditolak | versionCode sama | Naikkan build number |
| App crash di test track | Bug di release mode | Jalankan build release di device |
| Signing error | Keystore salah | Cek konfigurasi signing |
| Reviewer menolak | Metadata kurang lengkap | Lengkapi store listing dan policy |

---

## Ringkasan

Kalau target kita Android, jalur aman yang disarankan adalah:
- build APK untuk testing cepat,
- build AAB untuk Play Console,
- gunakan internal/closed testing sebelum production,
- lalu monitor hasil rilis.

---

*Study Jam 5 — Flutter Android Deployment*
