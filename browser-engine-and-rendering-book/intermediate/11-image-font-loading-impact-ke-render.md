# Image dan Font Loading Impact ke Render

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan intermediate 08-10.
  - [ ] Paham dasar LCP, CLS, dan jalur render awal halaman.
- Kamus mini:
  - `[baru]` render-blocking asset: aset yang menunda render tahap awal.
  - `[baru]` layout shift: pergeseran posisi elemen setelah initial render.
  - `[ulang]` LCP: metrik waktu elemen konten terbesar tampil.

## 1) Pengantar Singkat Topik
Topik ini membahas bagaimana strategi loading image dan font memengaruhi pipeline render, terutama saat first load dan scroll awal.

## 2) Big Picture
Image dan font sering dianggap masalah network saja, padahal dampaknya langsung ke style/layout/paint. Gambar tanpa dimensi dapat memicu layout shift, sedangkan font swap yang tidak terencana dapat mengubah metrik teks dan memaksa reflow. Dengan strategi loading yang tepat, kita bisa menurunkan jank visual dan membuat first render lebih stabil. Tujuan utama modul ini adalah menghubungkan keputusan asset loading dengan biaya render nyata.

## 3) Small Picture
- Tetapkan dimensi media untuk mencegah layout shift.
- Prioritaskan aset kritis (hero image/font penting), tunda aset non-kritis.
- Validasi hasil lewat metrik (LCP/CLS) dan trace render.

## 4) Wireframe Alur Konsep
```text
[request image/font] -> [decode + apply style] -> [layout/paint update] -> [stabilisasi frame]
```
- Alur utama: asset kritis termuat tepat waktu tanpa memicu shift besar.
- Alur jalan: placeholder menjaga geometri sebelum asset final siap.
- Alur error: asset telat atau tanpa reserve space memicu reflow dan paint ulang luas.

## 5) Analogi Dunia Nyata
Seperti memasang papan iklan: kalau ukuran frame sudah ditentukan sejak awal, isi poster bisa diganti tanpa menggeser struktur sekitarnya.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: halaman katalog, landing page visual-heavy, dashboard dengan chart/image card.
- Alasan pakai: mengurangi jank awal dan meningkatkan kualitas rendering saat load.
- Kapan tidak dipakai: ketika halaman minim media/font custom dan bottleneck utama ada di compute business logic.

## 7) Contoh Sederhana + Bedah Output
```html
<!-- Anti-pattern -->
<img src="/hero-large.jpg" alt="Hero" />

<!-- Better -->
<img
  src="/hero-large.jpg"
  alt="Hero"
  width="1200"
  height="630"
  loading="eager"
  fetchpriority="high"
/>
```

```css
@font-face {
  font-family: "BrandSans";
  src: url("/fonts/brand.woff2") format("woff2");
  font-display: swap;
}
```

Bedah output:
1. Dimensi image membantu browser reserve ruang sebelum decode selesai.
2. `fetchpriority="high"` membantu aset hero lebih cepat masuk jalur prioritas.
3. `font-display: swap` mengurangi text invisibility, tetapi tetap perlu kontrol agar shift teks tidak berlebihan.

## 8) Jebakan Umum (Pitfalls)
- Memuat semua gambar sebagai eager/high priority.
- Tidak menetapkan dimensi media sehingga layout berubah setelah image decode.
- Menggunakan banyak webfont variant tanpa prioritas yang jelas.
- Regression risk: redesign typography menambah font weight/variant berlebihan yang memperlambat render dan meningkatkan shift.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```html
<img src="/card-1.jpg" alt="Card image" />
<img src="/card-2.jpg" alt="Card image" />
```
Kunci 1:
- Output: gambar tampil saat termuat.
- Alasan: tanpa `width/height`, browser tidak tahu ruang final sejak awal sehingga risiko layout shift meningkat.

Drill 2:
```css
@font-face {
  font-family: "TextUI";
  src: url("/fonts/textui.woff2") format("woff2");
  font-display: optional;
}
```
Kunci 2:
- Output: teks bisa langsung tampil dengan fallback; font custom bisa jadi tidak diterapkan jika telat.
- Alasan: `optional` memprioritaskan stabilitas/kecepatan render dibanding kepastian swap font.

## 10) Debug Story
- Gejala: halaman produk terasa "loncat-loncat" saat pertama dibuka, walau JS utama ringan.
- Akar masalah: thumbnail tidak punya dimensi tetap, font heading swap terlambat, dan hero image terlalu besar tanpa prioritas jelas.
- Langkah debug:
  - Rekam load profile di `Performance` + pantau `Layout Shift` event.
  - Audit urutan request image/font di panel `Network`.
  - Terapkan reserve space image, atur font-display, dan prioritas asset kritis.
  - Uji ulang pada koneksi menengah.
- Solusi: stabilisasi geometri awal + prioritas asset kritis menurunkan shift dan meningkatkan kualitas render awal.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan hubungan image/font loading dengan layout shift.
- [ ] Bisa memilih strategi prioritas asset kritis vs non-kritis.
- [ ] Bisa memverifikasi dampak lewat metrik render.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: modul 08 (contention) dan 10 (large list rendering).
- Topik terkait: mini-project diagnosis intermediate (modul 12).
- Referensi tambahan: dokumentasi web.dev untuk image optimization, font loading, LCP/CLS.

## A) Pipeline Breakdown
- Parse: browser menemukan tag image dan rule font saat parsing dokumen/CSS.
- Style: font dan style final dihitung setelah resource tersedia atau fallback dipakai.
- Layout: terjadi saat dimensi media/metrics font final memengaruhi geometri.
- Paint: media dan glyph final digambar ke layar.
- Composite: hasil layer digabung; shift visual bisa terjadi jika layout berubah terlambat.

## B) Perf Metric yang Relevan
- Metric: `LCP` pada halaman media-heavy.
- Target: <= 2.5 detik pada baseline jaringan target.
- Dampak jika buruk: konten utama terlihat lambat dan perceived performance turun.
- Metric: `CLS` pada first load.
- Target: <= 0.1.
- Dampak jika buruk: UI terasa tidak stabil dan interaksi awal terganggu.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Network`
   - `Lighthouse`/`Performance Insights` (opsional)
2. Langkah observasi:
   - Rekam first load dengan throttling realistis.
   - Identifikasi event `Layout Shift`, urutan request image/font, dan waktu LCP.
   - Terapkan optimasi dimensi media, prioritas request, dan strategi font-display; rekam ulang.
3. Indikator sukses:
   - LCP membaik dan CLS menurun pada skenario yang sama.
   - Render awal lebih stabil tanpa lonjakan shift signifikan.

