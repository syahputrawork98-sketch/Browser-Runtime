# Render Pipeline Dasar di Browser

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham HTML/CSS dasar.
  - [ ] Pernah melihat DevTools.
- Kamus mini:
  - `[baru]` layout: proses hitung posisi dan ukuran elemen.
  - `[baru]` paint: proses menggambar piksel.
  - `[baru]` composite: penggabungan layer ke frame final.

## 1) Pengantar Singkat Topik
Topik ini membahas alur internal browser dari dokumen mentah sampai tampilan di layar.

## 2) Big Picture
UI lambat sering bukan karena JavaScript berat saja, tetapi karena biaya layout dan paint yang berulang. Dengan memahami pipeline rendering, kita bisa memilih teknik update UI yang lebih hemat biaya. Dampaknya adalah interaksi lebih halus dan frame drop lebih sedikit.

## 3) Small Picture
- Browser parse HTML/CSS.
- Browser hitung style dan layout.
- Browser paint lalu composite frame.

## 4) Wireframe Alur Konsep
```text
[HTML/CSS] -> [style+layout] -> [paint] -> [composite]
```
- Alur utama: perubahan kecil memicu kerja rendering minimal.
- Alur jalan: DOM update memicu tahap tertentu sesuai jenis perubahan.
- Alur error: perubahan tidak efisien memicu layout berulang.

## 5) Analogi Dunia Nyata
Seperti mencetak poster: rancang dulu, tentukan posisi, cetak warna, lalu tempel jadi hasil akhir.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: optimasi animasi, scroll, dan update UI besar.
- Alasan pakai: tahu biaya tiap operasi visual.
- Kapan tidak dipakai: untuk script non-UI murni.

## 7) Contoh Sederhana + Bedah Output
```js
const box = document.querySelector('.box');
box.style.width = '200px';
box.style.height = '200px';
box.style.background = 'tomato';
```
Bedah output:
1. Perubahan style membuat browser menilai ulang style.
2. Perubahan ukuran dapat memicu layout.
3. Perubahan visual dipaint lalu dikomposit.

## 8) Jebakan Umum (Pitfalls)
- Membaca layout (`offsetHeight`) setelah menulis style berulang.
- Animasi properti mahal seperti `width/height` di banyak elemen.
- Mengabaikan profiling DevTools saat optimasi.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
for (let i = 0; i < 1000; i++) {
  box.style.left = `${i}px`;
  // eslint-disable-next-line no-unused-expressions
  box.offsetHeight;
}
```
Kunci:
- Output: kode tetap jalan, tapi kemungkinan jank tinggi.
- Alasan: write-read layout berulang memicu layout thrashing.

## 10) Debug Story
- Gejala: animasi patah-patah saat drag.
- Akar masalah: update style memicu layout sinkron berulang.
- Langkah debug: rekam Performance panel dan lihat long frame.
- Solusi: batch update, hindari forced reflow, pakai transform untuk animasi.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan urutan pipeline rendering.
- [ ] Bisa identifikasi operasi UI yang mahal.
- [ ] Bisa memberi strategi dasar pengurangan jank.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: CSS layout dasar.
- Topik terkait: layout thrashing case study.
- Referensi tambahan: Rendering performance docs.
