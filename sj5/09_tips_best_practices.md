# 9️⃣ Tips, Checklist, dan Best Practices Deployment

## 1. Jangan Deploy dari Branch Sembarangan

Gunakan branch strategy yang jelas:
- `main` untuk production
- `staging` untuk uji final
- feature branch untuk development

Kalau tim masih kecil, minimal pastikan:
- pull request sudah di-review,
- dan build release diuji dulu sebelum tag release.

---

## 2. Selalu Naikkan Version Code

Kalau upload ke Play Console, build number harus naik.

```yaml
version: 1.0.0+1
version: 1.0.0+2
```

Jangan sampai upload ulang file dengan build number yang sama.

---

## 3. Jangan Commit Secret

File atau data berikut jangan masuk repo publik:
- keystore,
- `key.properties`,
- `.env`,
- API key sensitif,
- service account credential.

Contoh `.gitignore`:

```gitignore
android/key.properties
android/app/*.jks
android/app/*.keystore
.env
*.env
```

---

## 4. Uji Release di Device Asli

Debug build sering terasa aman, tapi release bisa beda.

Wajib cek:
- app launch,
- login,
- navigation,
- API call,
- dan media / asset loading.

Kalau ada fitur berat, test juga di device low-end.

---

## 5. Hindari Print Berlebihan

Di release, jangan bergantung pada `print()` untuk debugging.

Gunakan logger yang bisa dimatikan atau monitoring tool.

---

## 6. Buat Release Notes

Release notes membantu user dan tim paham apa yang berubah.

Contoh singkat:
- memperbaiki login,
- menambah halaman profil,
- meningkatkan performa loading,
- memperbaiki crash pada Android 14.

---

## 7. Checklist Sebelum Release

- [ ] Build debug sudah stabil
- [ ] Versi app naik
- [ ] Signing Android aman
- [ ] Environment production benar
- [ ] Screenshot dan metadata siap
- [ ] Android build / AAB berhasil
- [ ] Monitoring aktif
- [ ] Rollback plan ada

---

## 8. Checklist Setelah Release

- [ ] Install pertama berhasil
- [ ] Crash rate tidak naik drastis
- [ ] Event analytics masuk
- [ ] Feedback user dipantau
- [ ] Jika ada bug kritis, siapkan hotfix

---

## 9. Prinsip Utama

Deployment yang baik itu bukan cuma "aplikasi berhasil di-upload".
Yang penting adalah:
- aman,
- dapat dipantau,
- mudah di-rollback,
- dan siap dipelihara.

---

## Penutup

Setelah modul ini, peserta sudah punya gambaran end-to-end tentang:
- mempersiapkan project,
- build release,
- sign dan upload,
- distribusi Android,
- otomatisasi pipeline,
- sampai monitoring setelah rilis.

---

*Study Jam 5 — Flutter Deployment*
