# Testing Runtime Ordering dan Timing

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event loop, microtask/macrotask, dan scheduling prioritas.
  - [ ] Paham observability runtime (log, timing, trace id).
- Kamus mini:
  - `[baru]` deterministic test: test dengan hasil urutan yang konsisten.
  - `[baru]` fake timer: kontrol waktu virtual untuk mengetes timeout/interval.
  - `[baru]` flakiness: test kadang pass kadang gagal karena faktor non-deterministik.
  - `[ulang]` assertion: pengecekan hasil terhadap ekspektasi.

## 1) Pengantar Singkat Topik
Topik ini membahas cara mengetes perilaku runtime async secara stabil, terutama urutan eksekusi dan timing. Fokusnya adalah membuat test yang bisa dipercaya, bukan test yang kebetulan lolos.

## 2) Big Picture
Bug runtime sering muncul dari urutan eksekusi yang tidak sesuai asumsi. Kalau test hanya cek hasil akhir tanpa cek ordering, banyak bug timing lolos ke produksi. Dengan strategi testing ordering dan timing, kita bisa mengunci kontrak perilaku async. Hasilnya: regresi lebih cepat terdeteksi dan debugging lebih terarah.

## 3) Small Picture
- Pisahkan test urutan eksekusi dari test konten data.
- Gunakan log sequence sederhana untuk memvalidasi ordering.
- Kontrol waktu dengan fake timer saat menguji timeout/retry.
- Hindari assert yang bergantung ke delay real-time yang rapuh.

## 4) Wireframe Alur Konsep
```text
[setup test]
  -> [trigger async flow]
  -> [capture event sequence]
  -> [advance timer / flush microtask]
  -> [assert ordering + assert timing]
```
- Alur utama: test memeriksa urutan event dan batas waktu.
- Alur jalan: waktu dikontrol agar hasil deterministik.
- Alur error: tanpa kontrol waktu, test flakey dan sulit dipercaya.

## 5) Analogi Dunia Nyata
Seperti mengecek SOP dapur restoran: bukan hanya rasa akhir, tapi urutan proses masak juga harus benar supaya hasil konsisten.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: validasi debounce/throttle, retry dengan backoff, cancellation, dan race guard.
- Alasan pakai: mencegah bug timing yang sulit direproduksi manual.
- Kapan tidak dipakai: script sekali jalan tanpa alur async kompleks.

## 7) Contoh Sederhana + Bedah Output
```js
const trace = [];

function flow() {
  trace.push('sync:start');

  Promise.resolve().then(() => {
    trace.push('microtask:then');
  });

  setTimeout(() => {
    trace.push('task:timeout');
    console.log(trace.join(' -> '));
  }, 0);

  trace.push('sync:end');
}

flow();
```
Bedah output:
1. Urutan awal sinkron: `sync:start` lalu `sync:end`.
2. Microtask berjalan sebelum timeout: `microtask:then`.
3. Macrotask timeout terakhir: `task:timeout`.

## 8) Jebakan Umum (Pitfalls)
- Menulis test dengan `setTimeout` real-time tanpa fake timer.
- Hanya assert nilai akhir, tidak assert sequence langkah.
- Mencampur banyak concern (ordering, data, error) dalam satu test.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const out = [];
setTimeout(() => out.push('T'), 0);
Promise.resolve().then(() => out.push('P'));
out.push('S');
setTimeout(() => console.log(out.join('')), 0);
```
Kunci:
- Output: `SPT`
- Alasan: `S` sinkron dulu, `P` microtask, lalu `T` dari timeout pertama.

Drill 2:
```js
let done = false;
setTimeout(() => { done = true; }, 100);
console.log(done);
```
Kunci:
- Output: `false`
- Alasan: timer belum dieksekusi saat log sinkron berjalan.

Drill 3:
```js
const seq = [];
Promise.resolve().then(() => seq.push(1));
Promise.resolve().then(() => seq.push(2));
setTimeout(() => console.log(seq.join(',')), 0);
```
Kunci:
- Output: `1,2`
- Alasan: microtask queue mempertahankan urutan pendaftaran callback.

## 10) Debug Story
- Gejala: test retry kadang gagal meski kode tidak berubah.
- Akar masalah: test mengandalkan waktu nyata (real timer) dan kondisi mesin CI.
- Langkah debug: pecah test jadi sequence log, aktifkan fake timer, lalu advance waktu bertahap.
- Solusi: test dibuat deterministik dengan kontrol timer + assertion ordering per fase.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membedakan test ordering vs test hasil data.
- [ ] Bisa merancang test async yang deterministik.
- [ ] Bisa mendeteksi dan mengurangi flakiness pada test timing.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `advanced/13`, `advanced/15`, `advanced/16`.
- Topik terkait: `advanced/18-capstone-runtime-debugging-lab.md`.
- Referensi tambahan: fake timers, microtask flushing, dan async test best practices.

## 13) Model Mental
- Entitas utama: trigger, queue (microtask/task), clock (real/fake), assertion.
- Urutan proses: trigger flow -> catat sequence -> jalankan queue sesuai giliran -> validasi kontrak.
- Invariant (aturan yang selalu benar): test runtime yang baik harus mengunci urutan kritis, bukan hanya output akhir.

## 14) Failure Case
- Skenario gagal: test debounce pass di laptop lokal, tapi gagal acak di CI.
- Akar penyebab runtime: ketergantungan pada timing real dan beban mesin.
- Perbaikan: gunakan fake timer, pisahkan assertion per fase, dan kurangi asumsi berbasis delay absolut.

## 15) Spec Hint
- Section spec terkait (ringkas): event loop task processing, microtask checkpoint, timer scheduling.
- Kata kunci pencarian spec: `event loop`, `perform a microtask checkpoint`, `setTimeout`, `queue a task`.
