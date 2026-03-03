# UI Jank Karena Layout Thrashing

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham render pipeline dasar.
  - [ ] Pernah pakai Performance panel di DevTools.
- Kamus mini:
  - `[baru]` jank: frame tidak stabil sehingga animasi patah.
  - `[baru]` layout thrashing: pola baca-tulis layout berulang yang mahal.

## 1) Pengantar Singkat Topik
Case study ini membahas incident performa UI patah-patah akibat akses layout sinkron berulang.

## 2) Big Picture
Bug performa sering lolos karena aplikasi tetap "berfungsi", tapi pengalaman pengguna buruk. Studi kasus membantu melatih proses investigasi berbasis bukti, bukan asumsi. Dengan pola ini, kita bisa bergerak dari gejala ke akar masalah secara sistematis.

## 3) Small Picture
- Identifikasi gejala dan dampak.
- Rekam profil performa.
- Temukan root cause, lakukan fix, lalu pasang guard regresi.

## 4) Wireframe Alur Konsep
```text
[symptom] -> [investigation] -> [root cause] -> [fix] -> [regression guard]
```
- Alur utama: evidence mengarah ke akar masalah tunggal.
- Alur jalan: ada beberapa hipotesis lalu dieliminasi.
- Alur error: perbaikan sementara menutup gejala tapi akar masalah belum selesai.

## 5) Analogi Dunia Nyata
Seperti mesin kendaraan bergetar: bukan cukup menambah oli, tapi cari komponen pemicu getaran lalu pasang inspeksi berkala.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: postmortem performa frontend.
- Alasan pakai: melatih problem-solving production mindset.
- Kapan tidak dipakai: untuk bug kecil yang tidak butuh investigasi mendalam.

## 7) Context
- Produk/fitur: dashboard dengan list kartu bisa drag.
- Lingkungan: browser desktop menengah.
- Batasan: tidak boleh mengganti library UI besar.

## 8) Symptom
- Gejala utama: FPS turun saat drag cepat.
- Dampak user: interaksi terasa patah.
- Dampak bisnis/teknis: waktu penggunaan fitur menurun.

## 9) Investigation
- Data yang dikumpulkan: rekaman Performance, long task, dan timeline rendering.
- Hipotesis: handler drag terlalu banyak memicu reflow sinkron.
- Hasil verifikasi: pola write-style lalu read-layout berulang ditemukan.

## 10) Root Cause
Handler `mousemove` menulis properti posisi elemen lalu langsung membaca `offsetHeight` dalam loop yang sama, memaksa layout berulang setiap frame.

## 11) Fix
- Perubahan yang dilakukan: pisahkan fase read dan write, batching via `requestAnimationFrame`, dan pindah animasi ke `transform`.
- Kenapa valid: mengurangi forced layout dan memindahkan kerja ke jalur compositing.

## 12) Regression Guard
- Test/monitoring: baseline FPS dan budget long frame pada skenario drag.
- Trigger alert: jika frame > 16ms melebihi ambang yang disepakati.

## 13) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan gejala, akar masalah, dan fix.
- [ ] Bisa menunjukkan bukti dari DevTools.
- [ ] Bisa mendesain guard regresi performa.

## 14) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: render pipeline dasar.
- Topik terkait: forced reflow dan compositing.
- Referensi tambahan: performance profiling guide.
