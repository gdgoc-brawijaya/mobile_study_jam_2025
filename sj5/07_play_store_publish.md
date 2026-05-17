# 7️⃣ Publish ke Google Play Console

## Alur Publikasi

Setelah AAB siap, langkah umum untuk publish ke Play Store adalah:

```text
Build AAB -> Upload ke Play Console -> Isi metadata -> Review -> Release
```

---

## 1. Buat Akun Google Play Console

Kamu butuh:
- akun Google,
- biaya pendaftaran developer,
- dan akses ke dashboard Play Console.

---

## 2. Buat Aplikasi Baru

Di Play Console:
1. Klik **Create app**
2. Isi nama aplikasi
3. Pilih bahasa default
4. Tentukan apakah aplikasi gratis atau berbayar
5. Isi deklarasi dasar yang diminta

---

## 3. Siapkan Artifact Release

Build AAB:

```bash
flutter build appbundle --release
```

File hasil build:

```text
build/app/outputs/bundle/release/app-release.aab
```

---

## 4. App Signing

Google Play umumnya menggunakan **Play App Signing**.

Yang perlu diperhatikan:
- upload key harus aman,
- keystore tidak boleh hilang,
- dan build number harus naik setiap upload.

Kalau build number sama, upload biasanya ditolak.

---

## 5. Isi Store Listing

Sebelum publish, kamu biasanya perlu menyiapkan:
- nama aplikasi,
- deskripsi singkat,
- deskripsi panjang,
- screenshot,
- icon 512x512,
- feature graphic,
- kategori aplikasi,
- dan contact email.

---

## 6. Testing Track

Sebelum production, gunakan track:
- **Internal testing**
- **Closed testing**
- **Open testing**

Manfaatnya:
- cek build benar-benar jalan,
- user tester bisa memberi feedback,
- bug bisa ditemukan lebih awal.

---

## 7. Release Bertahap

Jangan langsung full rollout kalau app baru pertama kali rilis.

Lebih aman memakai:
- staged rollout,
- misalnya 10% dulu,
- lalu naik bertahap kalau stabil.

---

## 8. Hal yang Sering Lupa

- Versi build belum dinaikkan.
- Keystore tidak dicadangkan.
- Screenshot store belum lengkap.
- Permission declaration belum diisi.
- Privacy policy belum disiapkan jika aplikasi butuh data sensitif.

---

## Checklist Publish

- [ ] AAB sudah berhasil dibuat
- [ ] Signature release sudah benar
- [ ] Store listing sudah diisi
- [ ] Testing track sudah lewat
- [ ] Build number sudah unik
- [ ] Privacy policy siap jika dibutuhkan

---

*Study Jam 5 — Flutter Deployment*
