# Timer, Delay Minimum, dan Clamping

## Meta
- Level: `Beginner`
- Estimasi durasi: `35-50 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event loop dasar dan macrotask.
  - [ ] Paham `setTimeout` dan callback function.
- Kamus mini:
  - `[baru]` delay minimum: waktu tunggu minimal sebelum callback timer eligible dieksekusi.
  - `[baru]` clamping: browser menaikkan delay timer ke batas minimum tertentu.
  - `[ulang]` macrotask: task queue umum tempat callback timer dijadwalkan.

## 1) Pengantar Singkat Topik
Topik ini menjelaskan kenapa `setTimeout(fn, 0)` tidak berarti callback berjalan "sekarang". Kamu akan memahami delay minimum, antrian task, dan clamping pada timer berantai.

## 2) Big Picture
Bug timing sering muncul karena asumsi bahwa angka delay sama dengan waktu eksekusi aktual. Di browser, callback timer hanya menjadi eligible setelah delay lewat dan setelah call stack + microtask kosong. Pada kondisi tertentu, browser menerapkan clamping sehingga delay efektif jadi lebih besar. Memahami ini penting untuk debugging animasi, polling, dan logic debounce manual.

## 3) Small Picture
- `setTimeout` mendaftarkan callback ke task queue, bukan mengeksekusi langsung.
- Delay adalah batas paling cepat callback boleh diproses, bukan janji waktu pasti.
- Callback timer tetap menunggu giliran setelah sync dan microtask selesai.
- Timer berantai dapat mengalami clamping sehingga delay minimum meningkat.

## 4) Wireframe Alur Konsep
```text
[setTimeout(fn, n)]
  -> [timer countdown >= n]
  -> [enqueue macrotask]
  -> [stack kosong + microtask drain]
  -> [jalankan callback]
```
- Alur utama: delay terpenuhi, callback dijalankan pada giliran macrotask berikutnya.
- Alur jalan: callback tertunda lebih lama karena antrian padat.
- Alur error: asumsi `0ms` langsung jalan menyebabkan urutan state salah.

## 5) Analogi Dunia Nyata
Seperti nomor antrean layanan: setelah nomor dipanggil (delay selesai), kamu tetap menunggu loket benar-benar kosong untuk dilayani.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: penjadwalan aksi berkala, retry sederhana, delay UI non-kritis.
- Alasan pakai: memberi jeda eksekusi tanpa blocking main thread.
- Kapan tidak dipakai: untuk akurasi waktu tinggi yang butuh sinkronisasi frame presisi.

## 7) Contoh Sederhana + Bedah Output
```js
console.log('start');

setTimeout(() => {
  console.log('timer-0');
}, 0);

Promise.resolve().then(() => {
  console.log('microtask');
});

console.log('end');
```
Bedah output:
1. `start` dan `end` keluar dulu karena sinkron.
2. `microtask` berjalan sebelum timer.
3. `timer-0` baru dieksekusi pada giliran macrotask.

## 8) Jebakan Umum (Pitfalls)
- Mengira `setTimeout(..., 0)` lebih prioritas dari microtask.
- Menjadikan timer sebagai mekanisme "sinkronisasi pasti".
- Mengabaikan clamping saat timer dipanggil berulang cepat.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
setTimeout(() => console.log('T1'), 0);
console.log('S1');
```
Kunci:
- Output: `S1`, `T1`
- Alasan: callback timer masuk macrotask dan menunggu stack sinkron kosong.

Drill 2:
```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
```
Kunci:
- Output: `A`, `D`, `C`, `B`
- Alasan: microtask (`C`) diproses sebelum macrotask timer (`B`).

Drill 3:
```js
let i = 0;
function tick() {
  i += 1;
  console.log('tick', i);
  if (i < 3) setTimeout(tick, 0);
}
setTimeout(tick, 0);
console.log('ready');
```
Kunci:
- Output: `ready`, `tick 1`, `tick 2`, `tick 3`
- Alasan: tiap timer callback dijadwalkan ulang sebagai macrotask berikutnya, tidak sekaligus.

## 10) Debug Story
- Gejala: indikator loading hilang terlambat walau timer di-set `0ms`.
- Akar masalah: ada microtask chain panjang yang terus di-drain sebelum macrotask timer mendapat giliran.
- Langkah debug: beri log label `sync/microtask/timer` dan ukur timestamp sederhana.
- Solusi: kurangi chain microtask berat, atau jadwalkan ulang logic dengan strategi yang lebih tepat.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan kenapa `setTimeout(..., 0)` tetap tertunda.
- [ ] Bisa membedakan delay minimal vs waktu eksekusi aktual.
- [ ] Bisa memprediksi urutan log saat timer bercampur microtask.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/01-call-stack-task-microtask.md`.
- Topik terkait: `beginner/05-render-tick-dan-ui-update-timing.md`.
- Referensi tambahan: event loop dan timer behavior di browser.

## 13) Model Mental
- Entitas utama: timer scheduler, macrotask queue, microtask queue, call stack.
- Urutan proses: register timer -> delay terpenuhi -> enqueue macrotask -> tunggu giliran -> eksekusi callback.
- Invariant (aturan yang selalu benar): timer callback tidak memotong eksekusi sinkron yang sedang berjalan.

## 14) Failure Case
- Skenario gagal: developer menaruh urutan kritis pada `setTimeout(fn, 0)` dan berharap callback selalu langsung setelah baris berikutnya.
- Akar penyebab runtime: callback timer harus menunggu microtask drain dan antrian macrotask.
- Perbaikan: desain alur berdasarkan prioritas queue (sync -> microtask -> macrotask), bukan angka delay semata.

## 15) Spec Hint
- Section spec terkait (ringkas): HTML timer task source dan event loop task processing.
- Kata kunci pencarian spec: `timers`, `task queue`, `minimum delay`, `timer nesting level`.
