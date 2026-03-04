# Animation Pipeline: CSS vs JavaScript

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan intermediate 07-08.
  - [ ] Paham `transform/opacity`, `requestAnimationFrame`, dan dasar profiling frame time.
- Kamus mini:
  - `[baru]` animation pipeline: jalur kerja browser per frame saat animasi berjalan.
  - `[baru]` dropped frame: frame yang gagal dirender tepat waktu sesuai refresh rate.
  - `[ulang]` compositor-friendly property: properti yang cenderung aman di jalur compositing (`transform`, `opacity`).

## 1) Pengantar Singkat Topik
Topik ini membahas trade-off memilih animasi berbasis CSS atau JavaScript. Tujuannya adalah memilih pendekatan berdasarkan jenis efek dan budget performa, bukan preferensi gaya coding.

## 2) Big Picture
CSS animation/transitions biasanya lebih mudah menjaga jalur render stabil untuk efek standar, sementara JavaScript animation memberi kontrol logika yang lebih fleksibel. Masalah muncul saat kita memilih tool yang salah untuk problem yang dihadapi: efek sederhana dibuat terlalu kompleks di JS, atau efek dinamis dipaksa di CSS. Dengan memahami pipeline masing-masing, kita bisa menggabungkan keduanya secara terukur. Dampaknya adalah animasi yang halus sekaligus maintainable.

## 3) Small Picture
- Gunakan CSS untuk transisi state visual yang sederhana dan berulang.
- Gunakan JS (umumnya `requestAnimationFrame`) saat butuh orkestrasi, physics, atau sinkronisasi data real-time.
- Nilai pilihan dari trace: frame time, layout/paint cost, dan dropped frame.

## 4) Wireframe Alur Konsep
```text
[kebutuhan animasi] -> [pilih CSS/JS pipeline] -> [ukur frame + render cost] -> [iterasi]
```
- Alur utama: pilihan pipeline sesuai kebutuhan membuat animasi stabil.
- Alur jalan: mulai dari opsi paling sederhana, naikkan kompleksitas bila dibutuhkan.
- Alur error: JS animation update properti mahal (`top/left/width`) tanpa kontrol frame.

## 5) Analogi Dunia Nyata
Seperti memilih transmisi otomatis atau manual: otomatis cocok untuk pola umum yang konsisten, manual dipakai saat butuh kontrol detail pada kondisi khusus.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: micro-interaction, drag preview, progress visual, hero animation, dan motion antar state UI.
- Alasan pakai: membantu memilih pendekatan animasi dengan dampak render yang bisa diprediksi.
- Kapan tidak dipakai: jika bottleneck utama ada di decode media besar atau masalah non-animasi.

## 7) Contoh Sederhana + Bedah Output
```css
.chip {
  transition: transform 180ms ease, opacity 180ms ease;
}
.chip.enter {
  transform: translateY(0);
  opacity: 1;
}
```

```js
const chip = document.querySelector(".chip");
let x = 0;
function step() {
  x += 2;
  chip.style.transform = `translateX(${x}px)`;
  if (x < 200) requestAnimationFrame(step);
}
requestAnimationFrame(step);
```

Bedah output:
1. CSS cocok untuk state transition yang deklaratif dan sederhana.
2. JS via `requestAnimationFrame` cocok untuk kontrol langkah animasi per frame.
3. Keduanya bisa halus jika memakai properti ramah compositing dan tidak membebani main thread.

## 8) Jebakan Umum (Pitfalls)
- Menggunakan `setInterval` untuk animasi frame-based sehingga tidak sinkron dengan refresh display.
- Mengubah properti layout-heavy (`top/left`) di loop JS tanpa alasan kuat.
- Menumpuk terlalu banyak animasi bersamaan tanpa prioritas visual.
- Regression risk: penambahan efek baru mencampur CSS dan JS pada elemen sama hingga memicu konflik style dan frame drop.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const box = document.querySelector(".box");
let y = 0;
setInterval(() => {
  y += 4;
  box.style.top = `${y}px`;
}, 16);
```
Kunci 1:
- Output: box bergerak turun.
- Alasan: animasi berjalan, tetapi berisiko jank karena `setInterval` tidak sinkron frame dan `top` cenderung memicu layout/paint.

Drill 2:
```js
const box2 = document.querySelector(".box");
let y2 = 0;
function tick() {
  y2 += 4;
  box2.style.transform = `translateY(${y2}px)`;
  if (y2 < 400) requestAnimationFrame(tick);
}
requestAnimationFrame(tick);
```
Kunci 2:
- Output: box bergerak turun sampai 400px.
- Alasan: `requestAnimationFrame` sinkron dengan frame rendering dan `transform` biasanya lebih aman untuk jalur compositing.

## 10) Debug Story
- Gejala: animasi kartu promo mulus di desktop high-end, tetapi patah di laptop tim QA.
- Akar masalah: implementasi JS memakai `setInterval`, update `top/left`, plus kalkulasi sinkron setiap tick.
- Langkah debug:
  - Rekam trace selama animasi 3-5 detik di perangkat target.
  - Bandingkan versi CSS transition vs JS rAF dari sisi dropped frame dan time breakdown.
  - Refactor ke `transform/opacity`, pindahkan kalkulasi berat keluar loop animasi.
- Solusi: gabungan CSS transition untuk state change + JS rAF hanya untuk kebutuhan kontrol dinamis.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan kapan CSS animation lebih tepat daripada JS animation.
- [ ] Bisa menjelaskan kapan JS animation diperlukan dan bagaimana menjaganya tetap efisien.
- [ ] Bisa membuktikan pilihan dengan metrik dan trace.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: beginner 05 (compositing) dan intermediate 08 (main thread contention).
- Topik terkait: list rendering besar (modul 10) dan loading assets (modul 11).
- Referensi tambahan: dokumentasi Web Animations API dan requestAnimationFrame.

## A) Pipeline Breakdown
- Parse: tidak dominan saat animasi sudah berjalan.
- Style: dipicu saat class/state berubah pada CSS animation/transitions.
- Layout: meningkat jika animasi mengubah properti geometri.
- Paint: meningkat jika efek visual berat berubah tiap frame.
- Composite: tahap ideal untuk animasi `transform/opacity`.

## B) Perf Metric yang Relevan
- Metric: dropped frame ratio selama animasi.
- Target: < 5% dropped frame pada skenario target.
- Dampak jika buruk: motion terlihat patah dan kualitas UX turun.
- Metric: p95 frame time saat animasi aktif.
- Target: p95 <= 16.7 ms pada perangkat baseline tim.
- Dampak jika buruk: animasi tidak konsisten antar perangkat.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Rendering` (FPS meter)
2. Langkah observasi:
   - Rekam animasi versi CSS dan versi JS pada skenario identik.
   - Bandingkan `Layout/Paint/Composite` time breakdown serta dropped frame.
   - Evaluasi apakah kebutuhan kontrol animasi benar-benar butuh JS.
3. Indikator sukses:
   - Versi final memenuhi target frame time dan dropped frame ratio.
   - Trade-off maintainability vs performa terdokumentasi jelas untuk tim.

