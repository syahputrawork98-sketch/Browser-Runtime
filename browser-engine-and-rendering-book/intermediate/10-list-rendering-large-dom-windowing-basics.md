# List Rendering Large DOM dan Windowing Basics

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan intermediate 07-09.
  - [ ] Paham biaya style/layout/paint pada jumlah node besar.
- Kamus mini:
  - `[baru]` large DOM: kondisi jumlah node/elemen cukup besar sehingga biaya render meningkat signifikan.
  - `[baru]` windowing/virtualization: teknik merender hanya item yang terlihat (plus buffer) di viewport.
  - `[ulang]` viewport: area tampilan yang sedang terlihat oleh pengguna.

## 1) Pengantar Singkat Topik
Topik ini membahas kenapa list panjang sering jadi sumber jank, dan bagaimana windowing dasar mengurangi beban render tanpa mengubah pengalaman inti pengguna.

## 2) Big Picture
Merender ribuan item sekaligus membuat style, layout, dan paint membengkak, bahkan sebelum user berinteraksi. Saat scroll cepat, beban ini bertambah karena banyak elemen ikut diproses walau tidak terlihat. Windowing membatasi jumlah node aktif agar render cost proporsional dengan viewport, bukan total data. Hasilnya frame lebih stabil dan memori lebih terkendali.

## 3) Small Picture
- Batasi jumlah node aktif ke "visible window + overscan".
- Pisahkan data source dari node yang dirender.
- Ukur trade-off: performa vs kompleksitas implementasi.

## 4) Wireframe Alur Konsep
```text
[dataset besar] -> [hitung range visible] -> [render subset item] -> [update saat scroll]
```
- Alur utama: jumlah node DOM aktif tetap kecil walau data sangat besar.
- Alur jalan: scroll mengubah range visible lalu DOM subset diperbarui.
- Alur error: perhitungan range salah membuat blank gap atau lonjakan re-render.

## 5) Analogi Dunia Nyata
Seperti memajang buku perpustakaan di meja baca: hanya buku yang sedang dicari diletakkan di meja, sisanya tetap di rak arsip.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: activity feed, data table panjang, log viewer, infinite list.
- Alasan pakai: mengurangi biaya render dan memori pada dataset besar.
- Kapan tidak dipakai: ketika jumlah item kecil-menengah dan kompleksitas windowing tidak sepadan.

## 7) Contoh Sederhana + Bedah Output
```js
// Anti-pattern: render semua item sekaligus
const root = document.querySelector("#list");
const data = Array.from({ length: 10000 }, (_, i) => `Item ${i}`);
root.innerHTML = data.map((txt) => `<li class="row">${txt}</li>`).join("");
```

```js
// Windowing basics: render subset visible
const start = 200;
const size = 40;
const slice = data.slice(start, start + size);
root.innerHTML = slice.map((txt) => `<li class="row">${txt}</li>`).join("");
```

Bedah output:
1. Render semua item memperbesar biaya awal style/layout/paint.
2. Render subset menurunkan jumlah node aktif secara drastis.
3. Saat scroll, range subset diperbarui agar user tetap melihat list kontinyu.

## 8) Jebakan Umum (Pitfalls)
- Overscan terlalu besar sehingga manfaat windowing hilang.
- Mengupdate seluruh list tiap scroll tick tanpa diff sederhana.
- Mengabaikan tinggi item variatif sehingga perhitungan range meleset.
- Regression risk: fitur selection/highlight massal memaksa rerender banyak item di luar viewport.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const rows = Array.from({ length: 5000 }, (_, i) => `<li>${i}</li>`);
list.innerHTML = rows.join("");
```
Kunci 1:
- Output: semua item tampil sekaligus.
- Alasan: sederhana, tetapi berisiko meningkatkan biaya style/layout/paint saat load awal dan scroll berikutnya.

Drill 2:
```js
const total = 5000;
const visibleStart = 120;
const visibleCount = 30;
const rows2 = Array.from({ length: total }, (_, i) => `Item ${i}`);
const visible = rows2.slice(visibleStart, visibleStart + visibleCount);
list.innerHTML = visible.map((x) => `<li>${x}</li>`).join("");
```
Kunci 2:
- Output: hanya 30 item (range terlihat) dirender.
- Alasan: node DOM aktif lebih sedikit, sehingga render cost lebih ringan; butuh mekanisme tambahan untuk ilusi tinggi list total.

## 10) Debug Story
- Gejala: tabel log internal terasa berat saat data > 20k baris, terutama saat scroll cepat.
- Akar masalah: semua baris dirender permanen; filter dan highlight menyebabkan update luas.
- Langkah debug:
  - Rekam trace load awal dan scroll 10 detik.
  - Catat node count, layout spike, dan frame drop.
  - Terapkan windowing dasar + overscan kecil, lalu ulangi skenario sama.
- Solusi: windowing menurunkan node aktif dan menstabilkan frame time tanpa mengubah API data.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan kapan list besar butuh windowing.
- [ ] Bisa menghitung range visible sederhana untuk viewport.
- [ ] Bisa membaca dampak before/after pada trace performa.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: intermediate 08 (scroll jank) dan 09 (animation pipeline).
- Topik terkait: loading image/font (modul 11) dan mini-project diagnosis (modul 12).
- Referensi tambahan: dokumentasi virtualized list pattern.

## A) Pipeline Breakdown
- Parse: tidak dominan setelah struktur dasar list siap.
- Style: meningkat seiring jumlah node aktif.
- Layout: membengkak saat ribuan item ikut dihitung geometri.
- Paint: area repaint luas saat banyak node berubah.
- Composite: ikut terbebani jika update visual terjadi pada banyak elemen.

## B) Perf Metric yang Relevan
- Metric: active DOM node count pada list screen.
- Target: proporsional terhadap viewport (misal <= 3x visible items).
- Dampak jika buruk: memori naik, style/layout lambat, scroll jank.
- Metric: p95 frame time saat scroll list panjang.
- Target: p95 <= 16.7 ms pada perangkat baseline.
- Dampak jika buruk: scroll tersendat dan pengalaman baca data menurun.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Elements` (node count inspeksi)
2. Langkah observasi:
   - Rekam versi render-all vs windowing pada dataset yang sama.
   - Bandingkan node count aktif, durasi layout, dan dropped frame saat scroll.
   - Sesuaikan overscan lalu ulangi benchmark.
3. Indikator sukses:
   - Node aktif turun signifikan tanpa blank gap saat scroll.
   - Frame time lebih stabil pada skenario beban tinggi.

