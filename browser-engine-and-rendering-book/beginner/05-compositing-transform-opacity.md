# Compositing, Transform, dan Opacity

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham dasar paint cost dan layer.
  - [ ] Paham properti CSS `transform` dan `opacity`.
- Kamus mini:
  - `[baru]` compositing: tahap penggabungan layer menjadi frame final.
  - `[baru]` compositor-friendly animation: animasi yang dominan berjalan di tahap composite tanpa layout/paint berat.
  - `[baru]` layer promotion: elemen dipisah ke layer tersendiri agar update visual bisa lebih terisolasi.

## 1) Pengantar Singkat Topik
Topik ini menjelaskan kenapa animasi `transform` dan `opacity` sering lebih halus dibanding `top/left/width/height`. Ini penting untuk membuat interaksi terasa responsif tanpa membebani pipeline render.

## 2) Big Picture
Animasi yang memicu layout atau paint tiap frame mudah menghabiskan frame budget 16.7 ms. Dengan memindahkan jenis animasi ke properti yang ramah compositor, browser bisa mengurangi kerja di main thread. Namun, strategi ini tidak gratis karena layer tambahan juga memakai memori GPU. Tujuan topik ini adalah seimbang: halus saat bergerak, tetap hemat resource.

## 3) Small Picture
- `transform` dan `opacity` sering bisa diproses pada tahap compositing.
- `top/left/width/height` cenderung memicu layout/paint lebih sering.
- Layer tambahan perlu dikendalikan agar tidak memicu pressure memori.

## 4) Wireframe Alur Konsep
```text
[ubah transform/opacity] -> [update layer] -> [composite frame]
```
- Alur utama: animasi bergerak di jalur compositor dengan biaya stabil.
- Alur jalan: elemen dipromosikan ke layer lalu frame dikomposisikan ulang.
- Alur error: terlalu banyak layer atau efek berat tetap membuat performa turun.

## 5) Analogi Dunia Nyata
Seperti menggeser kaca film di atas meja daripada menggambar ulang meja setiap detik; gerakan jadi lebih cepat karena objek dasarnya tidak digambar ulang penuh.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: animasi hover card, drawer, modal, toast, drag preview.
- Alasan pakai: mengurangi kebutuhan layout/paint berulang pada animasi.
- Kapan tidak dipakai: ketika animasi tetap memerlukan perubahan geometri nyata atau ketika layer tambahan menyebabkan konsumsi memori berlebih.

## 7) Contoh Sederhana + Bedah Output
```css
.panel {
  transform: translateY(16px);
  opacity: 0;
  transition: transform 180ms ease, opacity 180ms ease;
}

.panel.is-open {
  transform: translateY(0);
  opacity: 1;
}
```
Bedah output:
1. Saat class `.is-open` ditambahkan, panel bergerak dan memudar secara halus.
2. Perubahan properti dominan terjadi di compositing jika kondisi layer mendukung.
3. Jika elemen banyak atau efek lain berat, hasil tetap perlu diverifikasi di DevTools.

## 8) Jebakan Umum (Pitfalls)
- Menganggap `transform/opacity` selalu gratis pada semua kondisi.
- Menambahkan `will-change` pada terlalu banyak elemen secara permanen.
- Menggabungkan animasi compositor-friendly dengan efek paint berat di elemen sama.
- Regression risk: layer promotion diterapkan global dan memicu lonjakan penggunaan memori GPU.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const item = document.querySelector(".item");

item.style.transition = "top 200ms ease";
item.style.top = "120px";
```
Kunci:
- Output: elemen berpindah ke bawah dengan animasi.
- Alasan: animasi `top` biasanya melibatkan layout/paint, jadi lebih berisiko jank dibanding `transform: translateY(...)`.

## 10) Debug Story
- Gejala: animasi daftar notifikasi patah-patah saat banyak item masuk bersamaan.
- Akar masalah: animasi memakai `top` + shadow berat, sehingga layout dan paint naik tiap frame.
- Langkah debug: rekam `Performance`, lihat komposisi `Layout/Paint/Composite`, cek layer di `Layers` panel.
- Solusi: ganti animasi posisi ke `transform`, sederhanakan efek visual, dan batasi `will-change` hanya saat transisi aktif.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa jelaskan kenapa `transform/opacity` sering lebih smooth.
- [ ] Bisa sebutkan risiko penggunaan layer berlebihan.
- [ ] Bisa memilih properti animasi sesuai tujuan performa.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: paint cost, layer basics, dan properti animasi CSS.
- Topik terkait: layer strategy (memory vs smoothness) di level advanced.
- Referensi tambahan: dokumentasi compositing dan animasi performa.

## A) Pipeline Breakdown
- Parse: browser membaca HTML/CSS.
- Style: browser menghitung style final elemen.
- Layout: dapat dilewati pada animasi `transform/opacity` murni.
- Paint: dapat diminimalkan jika elemen sudah ada di layer yang sesuai.
- Composite: tahap utama untuk menggabungkan layer tiap frame animasi.

## B) Perf Metric yang Relevan
- Metric: persentase frame di bawah 16.7 ms saat animasi berjalan.
- Target: >= 95% frame on-time pada perangkat target.
- Dampak jika buruk: animasi terasa patah, respons interaksi menurun.
- Metric: jumlah layer aktif selama skenario animasi.
- Target: hanya elemen penting yang dipromosikan ke layer.
- Dampak jika buruk: memori naik, bisa memicu stutter saat scrolling/transition lain.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Layers` (atau `Rendering` untuk inspeksi compositing)
2. Langkah observasi:
   - Rekam animasi sebelum optimasi (`top/left`) lalu sesudah (`transform/opacity`).
   - Bandingkan komposisi waktu `Layout/Paint` terhadap `Composite Layers`.
   - Audit jumlah layer aktif selama animasi berlangsung.
3. Indikator sukses:
   - Waktu `Layout/Paint` turun pada jalur animasi utama.
   - Animasi lebih stabil dan jumlah frame drop berkurang.

