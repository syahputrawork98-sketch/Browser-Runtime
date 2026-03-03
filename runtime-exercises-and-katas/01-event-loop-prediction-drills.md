# Event Loop Prediction Drills

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham Promise dasar.
  - [ ] Paham `setTimeout`.
- Kamus mini:
  - `[ulang]` microtask: callback prioritas dari Promise.
  - `[ulang]` macrotask: callback task reguler seperti timer.

## 1) Pengantar Singkat Topik
Latihan ini mengasah ketepatan prediksi urutan eksekusi JavaScript di browser.

## 2) Big Picture
Kemampuan prediksi output membuat debugging async jauh lebih cepat. Drill terstruktur membantu membangun refleks teknis, bukan tebakan. Setelah latihan ini, kamu harus bisa menjelaskan urutan output dengan alasan runtime yang jelas.

## 3) Small Picture
- Baca kode dan tandai sync/microtask/macrotask.
- Prediksi output tanpa menjalankan kode.
- Validasi lalu jelaskan alasan.

## 4) Wireframe Alur Konsep
```text
[kode] -> [klasifikasi queue] -> [prediksi output] -> [verifikasi]
```
- Alur utama: prediksi benar dengan alasan tepat.
- Alur jalan: prediksi salah lalu koreksi mental model.
- Alur error: jawaban benar tetapi alasan tidak tepat.

## 5) Analogi Dunia Nyata
Seperti membaca jadwal layanan: ada prioritas cepat (microtask) dan antrean reguler (macrotask).

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: latihan interview, debugging bug async.
- Alasan pakai: menstabilkan reasoning runtime.
- Kapan tidak dipakai: saat fokus materi bukan eksekusi async.

## 7) Soal
1. 
```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
```
2.
```js
console.log(1);
Promise.resolve()
  .then(() => console.log(2))
  .then(() => console.log(3));
setTimeout(() => console.log(4), 0);
console.log(5);
```
3.
```js
setTimeout(() => {
  console.log('T1');
  Promise.resolve().then(() => console.log('P-in-T1'));
}, 0);
Promise.resolve().then(() => console.log('P1'));
console.log('S');
```
4.
```js
console.log('X');
queueMicrotask(() => console.log('M1'));
Promise.resolve().then(() => console.log('M2'));
setTimeout(() => console.log('T'), 0);
console.log('Y');
```
5.
```js
async function run() {
  console.log('R1');
  await 0;
  console.log('R2');
}
run();
console.log('R3');
```

## 8) Constraint
- Tidak menjalankan kode sebelum menulis prediksi.
- Jelaskan alasan urutan untuk setiap soal.
- Tidak boleh hanya menebak.

## 9) Hint Bertahap
- Hint 1: selesaikan semua kode sinkron dulu.
- Hint 2: prioritaskan microtask sebelum macrotask.
- Hint 3: perhatikan microtask yang dibuat dari dalam macrotask.

## 10) Rubrik Penilaian
- Correctness: urutan output benar.
- Readability: alasan ditulis jelas dan ringkas.
- Runtime reasoning: istilah queue dipakai tepat.

## 11) Kunci Jawaban
- Soal 1: `A D C B`
- Soal 2: `1 5 2 3 4`
- Soal 3: `S P1 T1 P-in-T1`
- Soal 4: `X Y M1 M2 T`
- Soal 5: `R1 R3 R2`

## 12) Debug Story
- Gejala: implementasi retry async berjalan di urutan tak terduga.
- Akar masalah: callback timer diasumsikan mendahului promise chain.
- Langkah debug: tambahkan label queue pada log.
- Solusi: susun ulang alur agar dependency berjalan di microtask yang tepat.

## 13) Checkpoint Kelulusan Topik
- [ ] Minimal 4/5 prediksi benar.
- [ ] Semua jawaban disertai alasan teknis.
- [ ] Bisa menjelaskan 1 kesalahan prediksi dan koreksinya.

## 14) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: event loop dasar.
- Topik terkait: call stack, task, microtask.
- Referensi tambahan: docs event loop browser.
