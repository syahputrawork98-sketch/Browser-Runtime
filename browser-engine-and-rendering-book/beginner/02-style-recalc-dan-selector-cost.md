# Style Recalculation dan Selector Cost

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham dasar DOM tree dan CSS selector.
  - [ ] Pernah membuka tab `Performance` dan `Elements` di DevTools.
- Kamus mini:
  - `[baru]` style recalculation: proses browser menghitung ulang style final elemen setelah ada perubahan.
  - `[baru]` selector cost: biaya pencocokan selector CSS terhadap elemen di DOM.
  - `[ulang]` render pipeline: alur parse -> style -> layout -> paint -> composite.

## 1) Pengantar Singkat Topik
Topik ini membahas kenapa CSS selector tertentu bisa membuat tahap style di pipeline menjadi lebih mahal. Ini penting karena bottleneck style sering tersembunyi saat UI terasa lambat.

## 2) Big Picture
Saat DOM sering berubah, browser harus memutuskan elemen mana yang style-nya perlu dihitung ulang. Selector yang terlalu kompleks atau terlalu luas memperbesar area kerja ini. Dengan paham selector cost, kita bisa mendesain CSS yang lebih stabil saat data dan interaksi meningkat. Dampaknya, frame time lebih konsisten dan jank berkurang.

## 3) Small Picture
- Perubahan class/attribute dapat memicu style recalculation.
- Selector yang luas (contoh: descendant chain panjang) cenderung menaikkan biaya matching.
- Batasi scope selector agar invalidation area tidak melebar.

## 4) Wireframe Alur Konsep
```text
[DOM/class berubah] -> [selector matching + style recalculation] -> [style baru siap untuk layout/paint]
```
- Alur utama: update class pada elemen target hanya memengaruhi subtree yang relevan.
- Alur jalan: browser mencari elemen yang terdampak lalu hitung style finalnya.
- Alur error: selector terlalu generik menyebabkan banyak elemen ikut direcalc.

## 5) Analogi Dunia Nyata
Seperti mengecek seragam sekolah: jika aturan ditulis "semua orang di kota", proses cek jadi lama; jika ditulis "siswa kelas 6 di sekolah A", proses cek lebih cepat.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: merancang CSS yang tetap efisien saat komponen dan data bertambah.
- Alasan pakai: mengurangi biaya style stage sebelum masalah layout/paint ikut membesar.
- Kapan tidak dipakai: jika bottleneck utama terbukti ada di network atau JavaScript compute berat.

## 7) Contoh Sederhana + Bedah Output
```html
<div id="app">
  <div class="content">
    <ul class="list" id="list">
      <li class="item"><span class="item-label">A</span></li>
      <li class="item"><span class="item-label">B</span></li>
    </ul>
  </div>
</div>
```

```css
/* Lebih mahal saat DOM besar */
#app .content .list .item span {
  color: #333;
}

/* Lebih terarah */
.item-label {
  color: #333;
}
```

```js
const list = document.querySelector("#list");
list.classList.toggle("highlighted");
```

Bedah output:
1. Toggle class memicu style recalculation.
2. Selector yang kompleks butuh proses matching lebih banyak pada DOM besar.
3. Jika scope selector sempit, biaya style biasanya lebih kecil dan stabil.

## 8) Jebakan Umum (Pitfalls)
- Mengandalkan selector chain panjang untuk semua komponen.
- Mengubah class pada node root terlalu sering tanpa scope yang jelas.
- Optimasi selector tanpa mengukur dampak di DevTools.
- Regression risk: desain CSS baru memperluas selector global tanpa observasi invalidation area.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const app = document.querySelector("#app");
for (let i = 0; i < 200; i++) {
  app.classList.toggle("theme-dark");
}
```
Kunci:
- Output: UI tetap berubah, tetapi style recalculation bisa mahal jika selector bertumpu pada `.theme-dark` secara luas.
- Alasan: perubahan class berulang pada root bisa membuat banyak node harus dievaluasi ulang.

## 10) Debug Story
- Gejala: saat filter list diketik cepat, UI terasa tersendat.
- Akar masalah: class root diganti tiap keystroke, selector descendant panjang memicu style recalculation luas.
- Langkah debug: rekam `Performance`, cari event input yang diikuti durasi style tinggi, cek rule matching di `Elements`.
- Solusi: pindahkan class ke scope komponen, sederhanakan selector, kurangi perubahan class global.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa jelaskan hubungan perubahan class dengan style recalculation.
- [ ] Bisa mengidentifikasi selector yang berpotensi mahal.
- [ ] Bisa menyebutkan strategi mempersempit scope invalidation.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: CSS selector dasar, specificity, dan struktur DOM.
- Topik terkait: layout thrashing dan batching DOM update.
- Referensi tambahan: dokumentasi DevTools Performance panel.

## A) Pipeline Breakdown
- Parse: HTML/CSS diubah menjadi struktur yang bisa diproses browser.
- Style: browser melakukan selector matching dan menghitung computed style.
- Layout: berjalan jika hasil style memengaruhi geometri.
- Paint: berjalan jika ada perubahan visual.
- Composite: layer digabung menjadi frame akhir.

## B) Perf Metric yang Relevan
- Metric: durasi `Recalculate Style` per interaksi.
- Target: p95 < 4 ms untuk interaksi umum.
- Dampak jika buruk: input/scroll terasa tersendat walau JavaScript ringan.
- Metric: `Long Animation Frame` terkait style stage.
- Target: 0 long frame (>16.7 ms) pada skenario interaksi dasar.
- Dampak jika buruk: frame drop meningkat dan pengalaman terasa tidak halus.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Elements` (untuk audit selector dan scope class)
2. Langkah observasi:
   - Rekam aksi berulang (contoh: typing pada search/filter).
   - Lihat flame chart dan durasi `Recalculate Style`.
   - Bandingkan sebelum vs sesudah penyederhanaan selector.
3. Indikator sukses:
   - Durasi style stage turun konsisten pada skenario yang sama.
   - Tidak ada lonjakan frame time saat interaksi cepat.

