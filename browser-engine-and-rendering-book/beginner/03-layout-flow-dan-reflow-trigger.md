# Layout Flow dan Reflow Trigger

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Sudah paham pipeline render dasar.
  - [ ] Paham perbedaan perubahan style visual vs perubahan geometri.
- Kamus mini:
  - `[baru]` reflow/layout: proses hitung ulang posisi dan ukuran elemen.
  - `[baru]` forced synchronous layout: layout dipaksa dieksekusi saat JavaScript masih berjalan karena ada operasi baca ukuran.
  - `[ulang]` layout thrashing: pola write-read layout berulang yang membuat performa turun.

## 1) Pengantar Singkat Topik
Topik ini membahas kapan browser harus melakukan layout ulang dan kenapa beberapa pola kode membuat layout jadi mahal. Ini penting untuk mencegah jank saat interaksi UI.

## 2) Big Picture
Tidak semua perubahan CSS memicu layout, tetapi perubahan ukuran dan posisi biasanya memicunya. Masalah muncul saat kita menulis style lalu langsung membaca ukuran berkali-kali dalam loop. Dengan memahami reflow trigger, kita bisa memisahkan fase baca dan tulis supaya browser bekerja lebih efisien. Dampaknya frame lebih stabil, terutama pada animasi, scroll, dan list update.

## 3) Small Picture
- Properti yang mengubah geometri (`width`, `height`, `top`, `left`) berpotensi memicu layout.
- Membaca `offsetWidth/getBoundingClientRect()` setelah write style dapat memaksa layout sinkron.
- Batch `read` dulu, lalu `write`, hindari pola bolak-balik.

## 4) Wireframe Alur Konsep
```text
[ubah geometri] -> [layout recalculation] -> [paint/composite]
```
- Alur utama: perubahan geometri dihitung sekali untuk kumpulan update.
- Alur jalan: browser menggabungkan update sebelum commit frame.
- Alur error: kode write-read berulang memaksa layout sinkron berkali-kali.

## 5) Analogi Dunia Nyata
Seperti mengatur kursi aula: lebih cepat kalau ukur semua posisi dulu lalu geser sekaligus, bukan geser satu kursi lalu ukur ulang seluruh aula setiap langkah.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: optimasi interaksi drag, resize panel, virtual list, dan animasi elemen banyak.
- Alasan pakai: mengurangi biaya layout sinkron yang sering tidak terlihat dari log JavaScript biasa.
- Kapan tidak dipakai: jika bottleneck utama ada di render gambar berat atau komputasi data non-UI.

## 7) Contoh Sederhana + Bedah Output
```js
const items = document.querySelectorAll(".item");

for (const el of items) {
  el.style.width = "200px"; // write
  // eslint-disable-next-line no-unused-expressions
  el.offsetWidth; // read -> bisa memaksa layout sinkron
}
```
Bedah output:
1. Write style mengubah geometri elemen.
2. Read ukuran langsung setelah write bisa memicu forced layout.
3. Jika pola ini terjadi berulang, total frame time meningkat dan UI tersendat.

## 8) Jebakan Umum (Pitfalls)
- Mengukur layout di dalam loop yang juga menulis style.
- Menggunakan animasi `top/left/width/height` untuk elemen banyak tanpa profiling.
- Menganggap semua operasi DOM murah karena JavaScript terlihat singkat.
- Regression risk: fitur baru menambah read ukuran di loop render tanpa batch read/write.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const box = document.querySelector(".box");

box.style.width = "300px";
const w1 = box.offsetWidth;
box.style.width = "320px";
const w2 = box.offsetWidth;

console.log(w1, w2);
```
Kunci:
- Output: dua angka lebar terbaru elemen (contoh `300 320` sesuai CSS box model).
- Alasan: setiap read ukuran setelah write berpotensi memaksa layout sinkron agar nilai akurat.

## 10) Debug Story
- Gejala: saat accordion dibuka-tutup cepat, animasi patah-patah.
- Akar masalah: setiap toggle menjalankan write height lalu read ukuran berkali-kali untuk elemen anak.
- Langkah debug: rekam `Performance`, cari frame lambat dengan dominasi `Layout`, lalu telusuri call stack JS pemicunya.
- Solusi: kumpulkan semua read di awal, lakukan write di batch berikutnya (misalnya dengan `requestAnimationFrame`), dan prioritaskan `transform` bila memungkinkan.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menyebutkan operasi yang umum memicu reflow.
- [ ] Bisa jelaskan kenapa read setelah write dapat memaksa layout sinkron.
- [ ] Bisa menerapkan pola batch read/write sederhana.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: box model, positioning, dan ukuran elemen di CSS.
- Topik terkait: style recalculation dan selector cost.
- Referensi tambahan: dokumentasi forced reflow dan profiling rendering.

## A) Pipeline Breakdown
- Parse: browser membangun struktur dokumen dari HTML/CSS.
- Style: browser menentukan computed style final tiap elemen.
- Layout: browser menghitung geometri elemen; ini tahap utama yang dibahas di topik ini.
- Paint: browser menggambar hasil layout ke layer.
- Composite: layer digabung untuk ditampilkan sebagai frame.

## B) Perf Metric yang Relevan
- Metric: durasi `Layout` per interaksi.
- Target: p95 < 5 ms pada skenario interaksi dasar.
- Dampak jika buruk: respons UI melambat dan frame drop meningkat.
- Metric: jumlah forced layout event per aksi pengguna.
- Target: 0 forced layout pada jalur kritis interaksi umum.
- Dampak jika buruk: jank terjadi walau beban JavaScript kecil.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Rendering` (opsional untuk melihat frame rendering)
2. Langkah observasi:
   - Rekam interaksi yang dicurigai (expand/collapse, drag, resize).
   - Cari event `Layout` panjang dan cek urutan JS `write -> read`.
   - Refactor ke pola batch read/write, lalu rekam ulang skenario yang sama.
3. Indikator sukses:
   - Durasi layout menurun konsisten pada rekaman ulang.
   - Tidak ada lonjakan frame > 16.7 ms pada interaksi normal.

