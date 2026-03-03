# Call Stack, Task Queue, dan Microtask Queue

## Meta
- Level: `Beginner`
- Estimasi durasi: `30-45 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham function call dasar.
  - [ ] Paham Promise dan `setTimeout`.
- Kamus mini:
  - `[baru]` call stack: tumpukan eksekusi function sinkron.
  - `[baru]` microtask: antrian prioritas tinggi (contoh: `Promise.then`).
  - `[baru]` task/macrotask: antrian task umum (contoh: `setTimeout`).

## 1) Pengantar Singkat Topik
Topik ini menjelaskan alasan teknis di balik urutan output log pada kode async browser.

## 2) Big Picture
Tanpa mental model runtime, urutan log async terlihat acak dan sulit di-debug. Dengan memahami stack, microtask, dan macrotask, kita bisa memprediksi eksekusi sebelum menjalankan kode. Ini penting untuk menghindari race condition dasar dan bug timing.

## 3) Small Picture
- Kode sync dieksekusi dulu di call stack.
- Setelah stack kosong, browser memproses microtask.
- Baru kemudian task/macrotask diproses.

## 4) Wireframe Alur Konsep
```text
[sync code] -> [microtask drain] -> [next macrotask]
```
- Alur utama: sync selesai -> microtask -> macrotask.
- Alur jalan: callback async menunggu giliran queue.
- Alur error: asumsi urutan salah menyebabkan bug logika.

## 5) Analogi Dunia Nyata
Call stack seperti kasir yang harus melayani pelanggan sekarang, microtask seperti catatan prioritas di meja kasir, macrotask seperti antrean reguler berikutnya.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: membaca alur promise, timer, dan event UI.
- Alasan pakai: dasar untuk debugging async di browser.
- Kapan tidak dipakai: tidak perlu terlalu dalam saat logic murni sinkron.

## 7) Contoh Sederhana + Bedah Output
```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
```
Bedah output:
1. `A` dan `D` dieksekusi sinkron.
2. Microtask (`C`) diproses setelah stack kosong.
3. Macrotask timer (`B`) diproses setelah microtask.

## 8) Jebakan Umum (Pitfalls)
- Mengira `setTimeout(..., 0)` selalu paling cepat.
- Lupa bahwa semua microtask akan di-drain sebelum macrotask berikutnya.
- Tidak membedakan event callback dan promise callback.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
console.log('1');
Promise.resolve().then(() => console.log('2'));
setTimeout(() => console.log('3'), 0);
Promise.resolve().then(() => console.log('4'));
console.log('5');
```
Kunci:
- Output: `1`, `5`, `2`, `4`, `3`.
- Alasan: sync dulu, lalu microtask berurutan, lalu timer.

Drill 2:
```js
const logs = [];
setTimeout(() => logs.push('T'), 0);
logs.push('S');
Promise.resolve().then(() => logs.push('P'));
setTimeout(() => console.log(logs.join('-')), 0);
```
Kunci:
- Output: `S-P-T`
- Alasan: `S` sinkron dulu, `P` microtask setelah stack kosong, lalu timer `T`.

Drill 3:
```js
console.log('A');
setTimeout(() => {
  console.log('B');
  Promise.resolve().then(() => console.log('C'));
}, 0);
console.log('D');
```
Kunci:
- Output: `A`, `D`, `B`, `C`
- Alasan: `A/D` sinkron dulu, callback timer jalan berikutnya (`B`), lalu microtask di dalam callback (`C`).

## 10) Debug Story
- Gejala: state UI ter-update di urutan yang tidak diharapkan.
- Akar masalah: callback timer diasumsikan jalan sebelum `.then`.
- Langkah debug: tulis timestamp log dan label queue.
- Solusi: pindahkan logic kritikal ke alur microtask yang terkontrol.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membedakan sync, microtask, macrotask.
- [ ] Bisa memprediksi output contoh campuran promise dan timer.
- [ ] Bisa menjelaskan sumber bug urutan eksekusi.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: Promise dasar dan callback.
- Topik terkait: event loop detail.
- Referensi tambahan: HTML event loop overview.

## 13) Model Mental
- Entitas utama: call stack, microtask queue, task/macrotask queue.
- Urutan proses: eksekusi sinkron -> stack kosong -> drain microtask -> proses macrotask berikutnya.
- Invariant (aturan yang selalu benar): selama stack belum kosong, callback queue tidak dijalankan.

## 14) Failure Case
- Skenario gagal: developer mengira `setTimeout(..., 0)` selalu lebih cepat dari `.then`.
- Akar penyebab runtime: microtask memiliki prioritas sebelum macrotask setelah stack kosong.
- Perbaikan: tempatkan logika urutan kritis sesuai jenis queue dan verifikasi lewat prediksi output.

## 15) Spec Hint
- Section spec terkait (ringkas): event loop processing model dan microtask checkpoint.
- Kata kunci pencarian spec: `event loop`, `task queue`, `microtask`, `perform a microtask checkpoint`.
