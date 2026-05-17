# 8️⃣ Monitoring, Crash Reporting, dan Rollback Strategy

## Kenapa Setelah Deploy Masih Penting?

Rilis bukan akhir. Setelah aplikasi live, kita masih harus memantau:
- crash,
- error API,
- masalah performa,
- dan feedback user.

---

## 1. Monitoring Dasar

Yang perlu dilihat setelah release:
- jumlah install,
- rating aplikasi,
- crash rate,
- screen flow,
- dan event penting.

Tools yang umum dipakai:
- Firebase Crashlytics,
- Firebase Analytics,
- Sentry,
- atau dashboard observability lain.

---

## 2. Crash Reporting

Crash reporting membantu kita melihat error yang muncul di device user.

Contoh manfaat:
- tahu line error yang sering muncul,
- melihat device mana yang bermasalah,
- dan memprioritaskan bug fix.

Kalau pakai Firebase Crashlytics, alurnya biasanya:
1. install package,
2. aktifkan native config,
3. kirim error manual bila perlu,
4. cek dashboard.

---

## 3. Logging yang Berguna

Di production, logging harus seperlunya.

Contoh:

```dart
try {
  await api.fetchData();
} catch (e, stackTrace) {
  // kirim ke crash reporting
  reportError(e, stackTrace);
}
```

Simpan detail teknis ke monitoring tool, bukan ke UI user.

---

## 4. Rollback Strategy

Kalau release bermasalah, kita perlu cara mundur yang cepat.

Pilihan rollback:
- **Play Store staged rollout** dihentikan,
- publish hotfix versi baru,
- kembali ke versi sebelumnya jika hosting memungkinkan,
- atau matikan feature flag yang bermasalah.

---

## 5. Feature Flag

Feature flag sangat membantu saat deployment.

Contoh use case:
- fitur baru dimatikan kalau bermasalah,
- sebagian user saja yang melihat fitur,
- atau eksperimen UI dilakukan aman.

---

## 6. Post-Release Checklist

Setelah publish, cek:
- aplikasi bisa dibuka normal,
- login / navigasi utama jalan,
- analytics event masuk,
- crash rate aman,
- dan review awal user dipantau.

---

## Checklist Monitoring

- [ ] Crash reporting aktif
- [ ] Analytics event penting sudah dikirim
- [ ] Ada rencana rollback
- [ ] Feature flag dipakai untuk fitur berisiko
- [ ] Bug report dari user dipantau

---

*Study Jam 5 — Flutter Deployment*
