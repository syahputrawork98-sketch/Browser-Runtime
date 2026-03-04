# Paint Cost dan Layer Basics

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham perbedaan style, layout, dan paint.
  - [ ] Pernah profiling sederhana di DevTools Performance.
- Kamus mini:
  - `[baru]` paint: proses menggambar piksel berdasarkan hasil layout dan style.
  - `[baru]` paint invalidation: area yang ditandai perlu digambar ulang.
  - `[baru]` layer: permukaan terpisah yang bisa dikomposit tanpa selalu repaint seluruh halaman.

## 1) Pengantar Singkat Topik
Topik ini membahas kenapa sebagian perubahan visual memakan biaya paint tinggi, dan kapan layer bisa membantu mengurangi kerja ulang. Ini penting untuk menjaga animasi dan interaksi tetap halus.

## 2) Big Picture
UI sering terasa berat bukan karena layout, tapi karena area repaint terlalu besar atau terlalu sering. Efek visual seperti shadow besar, blur, atau perubahan background luas bisa menaikkan biaya paint. Dengan memahami paint cost dan layer basics, kita bisa memilih teknik visual yang tetap enak dilihat tetapi lebih hemat render. Hasilnya frame lebih stabil dan penurunan FPS berkurang.

## 3) Small Picture
- Perubahan visual yang tidak mengubah geometri tetap bisa mahal di tahap paint.
- Area invalidation yang luas berarti lebih banyak piksel harus digambar ulang.
- Layer terpisah dapat mengisolasi update visual agar repaint tidak menyebar.

## 4) Wireframe Alur Konsep
```text
[perubahan visual] -> [paint invalidation] -> [raster/paint ulang] -> [composite]
```
- Alur utama: repaint terjadi pada area kecil yang relevan.
- Alur jalan: browser menandai region invalid lalu menggambar ulang region itu.
- Alur error: perubahan visual memicu repaint besar berulang pada frame berdekatan.

## 5) Analogi Dunia Nyata
Seperti mengecat tembok: lebih cepat retouch satu sudut yang rusak daripada mengecat ulang seluruh ruangan setiap kali ada goresan kecil.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: optimasi animasi elemen visual kaya efek, hover states kompleks, dan update list besar.
- Alasan pakai: mengendalikan biaya piksel-level yang sering tidak terlihat dari log JavaScript.
- Kapan tidak dipakai: jika bottleneck utama sudah terbukti di network latency atau operasi data berat.

## 7) Contoh Sederhana + Bedah Output
```css
.card {
  width: 240px;
  height: 120px;
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.25);
  transition: box-shadow 150ms ease;
}

.card:hover {
  box-shadow: 0 24px 56px rgba(0, 0, 0, 0.35);
}
```

Bedah output:
1. Hover tidak mengubah layout, tetapi mengubah visual sehingga memicu paint.
2. Shadow besar meningkatkan area dan kompleksitas repaint.
3. Jika banyak card berubah bersamaan, total paint cost bisa melonjak.

## 8) Jebakan Umum (Pitfalls)
- Menggunakan efek blur/shadow berat pada banyak elemen interaktif.
- Menganggap perubahan warna/background selalu murah pada area besar.
- Memaksa layer di terlalu banyak elemen tanpa cek dampak memori.
- Regression risk: penambahan efek visual baru memperluas area repaint di komponen list/grid.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const panel = document.querySelector(".panel");
let on = false;

setInterval(() => {
  on = !on;
  panel.style.background = on
    ? "linear-gradient(90deg, #111, #444)"
    : "linear-gradient(90deg, #222, #666)";
}, 100);
```
Kunci:
- Output: background panel berganti terus tiap 100 ms.
- Alasan: perubahan visual berulang memicu paint berkala; jika area panel besar, biaya paint bisa signifikan.

## 10) Debug Story
- Gejala: hover grid produk terasa tersendat di laptop kelas menengah.
- Akar masalah: tiap hover mengubah `box-shadow` tebal pada banyak kartu dalam viewport.
- Langkah debug: rekam `Performance`, lihat dominasi `Paint`, aktifkan highlight `Paint flashing` untuk melihat area repaint.
- Solusi: sederhanakan efek shadow, batasi elemen yang berubah bersamaan, gunakan layer hanya pada elemen prioritas.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membedakan bottleneck paint vs layout.
- [ ] Bisa menjelaskan kenapa area repaint memengaruhi performa.
- [ ] Bisa menyebutkan kapan layer membantu dan kapan berisiko boros.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: dasar properti visual CSS (color, shadow, filter, gradient).
- Topik terkait: compositing dan strategi animasi transform/opacity.
- Referensi tambahan: dokumentasi DevTools paint profiling.

## A) Pipeline Breakdown
- Parse: browser memproses HTML/CSS menjadi struktur kerja internal.
- Style: browser menghitung computed style elemen.
- Layout: dipakai jika ada perubahan geometri; tidak selalu terjadi pada kasus paint-only.
- Paint: piksel digambar ulang pada region invalid.
- Composite: layer hasil paint digabung untuk frame akhir.

## B) Perf Metric yang Relevan
- Metric: durasi `Paint` per interaksi/frame.
- Target: p95 < 4 ms untuk interaksi hover/transition umum.
- Dampak jika buruk: hover/scroll terasa tersendat walau layout kecil.
- Metric: luas area repaint (indikasi dari paint flashing dan trace).
- Target: area repaint terbatas pada komponen yang berubah.
- Dampak jika buruk: frame budget cepat habis karena repaint menyebar.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Rendering` -> `Paint flashing`
2. Langkah observasi:
   - Rekam interaksi (hover, toggle theme, animasi visual).
   - Cek event `Paint` dan total durasinya di timeline.
   - Aktifkan `Paint flashing` untuk melihat apakah repaint terlalu luas.
3. Indikator sukses:
   - Durasi paint turun setelah optimasi efek visual.
   - Area repaint lebih terlokalisasi ke komponen target.

