# Promise Jobs dan Urutan Eksekusi

## Meta
- Level: `Beginner`
- Estimasi durasi: `35-50 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham dasar `Promise` (`resolve`/`reject`).
  - [ ] Paham call stack, microtask, dan macrotask dari modul 01.
- Kamus mini:
  - `[baru]` promise job: pekerjaan microtask yang dijadwalkan oleh `then/catch/finally`.
  - `[ulang]` microtask queue: antrian prioritas tinggi yang diproses setelah stack kosong.
  - `[baru]` chain: rangkaian `then/catch/finally` yang saling meneruskan hasil.

## 1) Pengantar Singkat Topik
Topik ini membahas kapan callback `Promise.then/catch/finally` benar-benar dieksekusi. Fokusnya adalah urutan nyata saat kode sinkron, promise job, dan timer bercampur.

## 2) Big Picture
Bug async sering muncul karena asumsi yang salah: "kalau Promise sudah resolve, callback langsung jalan sekarang". Faktanya callback Promise selalu dijadwalkan sebagai job di microtask queue. Dengan model ini, kita bisa memprediksi output log, mencegah race sederhana, dan menulis flow async yang konsisten.

## 3) Small Picture
- Callback `then/catch/finally` tidak jalan sinkron, selalu jadi promise job.
- Semua promise job yang sudah terjadwal akan di-drain sebelum macrotask berikutnya.
- Urutan chain mengikuti urutan pendaftaran dan hasil dari step sebelumnya.

## 4) Wireframe Alur Konsep
```text
[sync code]
  -> [promise resolved]
  -> [enqueue promise jobs]
  -> [stack kosong]
  -> [drain microtask]
  -> [next macrotask]
```
- Alur utama: sync selesai dulu, baru callback Promise berjalan.
- Alur jalan: chain `then` berjalan bertahap dari hasil step sebelumnya.
- Alur error: asumsi callback Promise sinkron menyebabkan urutan state salah.

## 5) Analogi Dunia Nyata
Seperti loket yang mencatat "pekerjaan prioritas berikutnya" di papan kecil. Pekerjaan itu tidak dikerjakan saat ini juga, tetapi segera setelah pekerjaan saat ini selesai.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: flow fetch, validasi async, transform data bertahap.
- Alasan pakai: promise job memberi urutan eksekusi async yang terstruktur.
- Kapan tidak dipakai: logic murni sinkron tanpa ketergantungan async.

## 7) Contoh Sederhana + Bedah Output
```js
console.log('sync-1');

Promise.resolve('ok')
  .then((value) => {
    console.log('then-1', value);
    return 'next';
  })
  .then((value) => {
    console.log('then-2', value);
  });

console.log('sync-2');
```
Bedah output:
1. `sync-1` dan `sync-2` keluar dulu karena sinkron.
2. `then-1 ok` berjalan saat microtask drain.
3. `then-2 next` berjalan setelah `then-1` selesai.

## 8) Jebakan Umum (Pitfalls)
- Mengira `Promise.resolve().then(...)` dieksekusi sinkron.
- Mencampur update state sinkron dan promise job tanpa urutan yang jelas.
- Lupa `return` pada chain `then`, sehingga nilai berikutnya jadi `undefined`.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
console.log('A');
Promise.resolve().then(() => console.log('B'));
console.log('C');
```
Kunci:
- Output: `A`, `C`, `B`
- Alasan: `B` adalah promise job (microtask), jadi jalan setelah stack sinkron kosong.

Drill 2:
```js
Promise.resolve(1)
  .then((x) => x + 1)
  .then((x) => console.log(x));
console.log('done');
```
Kunci:
- Output: `done`, `2`
- Alasan: chain Promise berjalan sebagai microtask setelah kode sinkron selesai.

Drill 3:
```js
Promise.resolve()
  .then(() => {
    console.log('p1');
    setTimeout(() => console.log('t1'), 0);
  })
  .then(() => console.log('p2'));
```
Kunci:
- Output: `p1`, `p2`, `t1`
- Alasan: `p2` tetap microtask, diproses sebelum timer macrotask `t1`.

## 10) Debug Story
- Gejala: status UI berubah ke "ready" sebelum data turunan selesai diproses.
- Akar masalah: update status sinkron dilakukan sebelum chain Promise selesai.
- Langkah debug: beri label log per fase (`sync`, `then-1`, `then-2`, `timer`) dan cek urutan aktual.
- Solusi: pindahkan update status akhir ke `then/finally` terakhir.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa jelaskan kapan callback `then/catch/finally` dieksekusi.
- [ ] Bisa memprediksi urutan log saat Promise bercampur timer.
- [ ] Bisa menyebutkan dampak lupa `return` di chain Promise.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/01-call-stack-task-microtask.md`.
- Topik terkait: `beginner/03-timer-delay-minimum-dan-clamping.md`.
- Referensi tambahan: konsep Promise jobs dan microtask queue.

## 13) Model Mental
- Entitas utama: call stack, promise job (microtask), macrotask queue.
- Urutan proses: sync -> enqueue promise jobs -> drain microtask -> lanjut macrotask.
- Invariant (aturan yang selalu benar): callback Promise tidak pernah jalan sinkron di frame kode saat ini.

## 14) Failure Case
- Skenario gagal: developer menulis validasi akhir sinkron tepat setelah `then`, berharap data sudah siap.
- Akar penyebab runtime: callback `then` belum dieksekusi karena masih menunggu microtask drain.
- Perbaikan: pindahkan validasi akhir ke `then` terakhir atau `finally`.

## 15) Spec Hint
- Section spec terkait (ringkas): JavaScript Promise reaction jobs dan konsep microtask checkpoint.
- Kata kunci pencarian spec: `PromiseJobs`, `microtask checkpoint`, `then reaction`.
