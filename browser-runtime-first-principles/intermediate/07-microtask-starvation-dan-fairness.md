# Microtask Starvation dan Fairness

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event loop dasar: sync, microtask, macrotask.
  - [ ] Paham Promise chain dan timer behavior.
- Kamus mini:
  - `[baru]` starvation: kondisi saat satu jenis pekerjaan terus tertunda karena queue lain mendominasi.
  - `[baru]` fairness: keseimbangan agar berbagai jenis task mendapat giliran eksekusi.
  - `[ulang]` microtask drain: proses browser mengeksekusi microtask sampai habis sebelum lanjut fase berikutnya.

## 1) Pengantar Singkat Topik
Topik ini membahas risiko nyata saat microtask diproduksi tanpa kontrol. Kamu akan belajar kenapa UI bisa terasa “freeze” meski tidak ada loop sinkron berat.

## 2) Big Picture
Promise dan microtask sangat berguna, tapi bisa jadi jebakan kalau dipakai sebagai loop tidak terbatas. Karena microtask di-drain terus sebelum macrotask dan rendering, callback lain bisa kelaparan. Dampaknya: input user terasa lag, timer tertunda, dan update visual tidak sempat dipaint. Dengan fairness strategy, kita menjaga app tetap responsif di bawah beban async tinggi.

## 3) Small Picture
- Microtask punya prioritas tinggi dan di-drain sampai kosong.
- Menambah microtask baru di dalam microtask bisa memperpanjang drain tanpa akhir praktis.
- Fairness dicapai dengan sesekali “yield” ke macrotask/frame.
- Jangan jadikan Promise chain tak terbatas sebagai scheduler tunggal.

## 4) Wireframe Alur Konsep
```text
[enqueue microtask]
  -> [run microtask]
  -> [enqueue microtask lagi]
  -> [drain berulang]
  -> [macrotask/render tertunda]
```
- Alur utama: microtask berjalan cepat, tapi tetap harus dikendalikan.
- Alur jalan: sistem tetap responsif jika ada yield berkala.
- Alur error: microtask loop membuat timer/render kelaparan.

## 5) Analogi Dunia Nyata
Seperti antrean prioritas di rumah sakit: jika pasien prioritas datang terus tanpa jeda, antrean reguler tidak pernah dipanggil.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: desain scheduler ringan, batching async, optimasi alur Promise.
- Alasan pakai: mencegah UI starvation pada aplikasi interaktif.
- Kapan tidak dipakai: flow sederhana tanpa produksi microtask berantai.

## 7) Contoh Sederhana + Bedah Output
```js
let count = 0;

function microtaskLoop() {
  count += 1;
  if (count % 1000 === 0) {
    console.log('microtask count:', count);
  }
  if (count < 5000) {
    Promise.resolve().then(microtaskLoop);
  }
}

setTimeout(() => {
  console.log('timer fired');
}, 0);

Promise.resolve().then(microtaskLoop);
console.log('start');
```
Bedah output:
1. `start` muncul dulu (sync).
2. Microtask loop berjalan panjang sampai kondisi berhenti.
3. `timer fired` baru muncul setelah microtask drain selesai.

## 8) Jebakan Umum (Pitfalls)
- Mengira Promise chain panjang “gratis” untuk responsivitas.
- Tidak ada batas iterasi atau mekanisme yield.
- Men-debug lag UI hanya dari network, padahal penyebab di queue starvation.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
setTimeout(() => console.log('T'), 0);
Promise.resolve().then(() => {
  console.log('P1');
  Promise.resolve().then(() => console.log('P2'));
});
console.log('S');
```
Kunci:
- Output: `S`, `P1`, `P2`, `T`
- Alasan: semua microtask (`P1`, `P2`) di-drain sebelum timer.

Drill 2:
```js
let i = 0;
function run() {
  i += 1;
  if (i < 3) Promise.resolve().then(run);
  console.log('M', i);
}
setTimeout(() => console.log('T'), 0);
Promise.resolve().then(run);
```
Kunci:
- Output: `M 1`, `M 2`, `M 3`, `T`
- Alasan: microtask berantai habis dulu sebelum macrotask timer.

Drill 3:
```js
let n = 0;
function fairRun() {
  n += 1;
  console.log('step', n);
  if (n < 4) {
    if (n % 2 === 0) setTimeout(fairRun, 0); // yield ke macrotask
    else Promise.resolve().then(fairRun);
  }
}
fairRun();
```
Kunci:
- Output urutan step tetap naik 1..4, tetapi eksekusi tidak sepenuhnya menahan macrotask.
- Alasan: ada yield periodik via `setTimeout`, memberi fairness ke task lain.

## 10) Debug Story
- Gejala: tombol UI tidak responsif saat proses async “ringan” berjalan.
- Akar masalah: Promise chain rekursif membuat microtask drain sangat panjang.
- Langkah debug: tambahkan metrik jumlah microtask per detik dan cek delay timer sederhana.
- Solusi: batasi batch microtask dan lakukan yield periodik ke macrotask atau frame.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan microtask starvation dengan contoh nyata.
- [ ] Bisa merancang strategi fairness sederhana (yield berkala).
- [ ] Bisa memprediksi efek chain Promise terhadap timer/render.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/02-promise-jobs-dan-urutan-eksekusi.md`.
- Topik terkait: `intermediate/10-runtime-side-effects-dan-state-consistency.md`.
- Referensi tambahan: event loop processing model dan task scheduling basics.

## 13) Model Mental
- Entitas utama: microtask queue, macrotask queue, render opportunity.
- Urutan proses: sync -> microtask drain (sampai kosong) -> macrotask -> render.
- Invariant (aturan yang selalu benar): jika microtask terus menambah microtask tanpa jeda, macrotask/render akan tertunda.

## 14) Failure Case
- Skenario gagal: scheduler internal memakai Promise recursion tak terbatas untuk “mempercepat” proses.
- Akar penyebab runtime: microtask mendominasi event loop dan menghambat fairness.
- Perbaikan: gunakan batching + yield periodik (`setTimeout`/`requestAnimationFrame`) agar antrian lain mendapat giliran.

## 15) Spec Hint
- Section spec terkait (ringkas): event loop step yang mewajibkan microtask checkpoint sebelum lanjut task/render.
- Kata kunci pencarian spec: `perform a microtask checkpoint`, `event loop`, `task queue`.
