# Observability Runtime: Log, Timing, dan Trace

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham scheduling, batching, dan backpressure pada runtime browser.
  - [ ] Paham cara membaca state transition dasar.
- Kamus mini:
  - `[baru]` observability: kemampuan memahami perilaku sistem dari sinyal runtime.
  - `[baru]` trace id: penanda unik untuk mengikuti satu alur kerja end-to-end.
  - `[baru]` latency budget: batas waktu target untuk menjaga UX tetap baik.
  - `[ulang]` instrumentation: penambahan log/metric/trace untuk analisis.

## 1) Pengantar Singkat Topik
Topik ini membahas cara membuat runtime browser “terlihat” lewat log, timing, dan trace. Tujuannya supaya debugging tidak bergantung intuisi, tetapi berbasis bukti.

## 2) Big Picture
Bug runtime sering sulit direproduksi karena urutan event async tidak selalu sama. Tanpa observability, kita hanya menebak sumber masalah. Dengan trace id, pengukuran timing, dan log terstruktur, kita bisa menelusuri akar masalah lebih cepat. Hasilnya: perbaikan lebih presisi, regresi lebih mudah dideteksi.

## 3) Small Picture
- Gunakan format log konsisten (`timestamp`, `traceId`, `phase`, `message`).
- Ukur durasi operasi penting (start/end dan elapsed time).
- Kaitkan event terkait dengan trace id yang sama.
- Pisahkan log debug lokal dari log penting produksi.

## 4) Wireframe Alur Konsep
```text
[action start]
  -> [generate traceId]
  -> [log start + timing start]
  -> [async steps]
  -> [log each phase]
  -> [timing end + result]
```
- Alur utama: satu action punya jejak lengkap dari awal sampai akhir.
- Alur jalan: trace memudahkan melihat bottleneck tiap phase.
- Alur error: tanpa trace id, log bercampur antar request dan sulit dianalisis.

## 5) Analogi Dunia Nyata
Seperti nomor resi pengiriman: setiap paket punya ID sehingga perjalanan bisa dilacak dari gudang sampai tujuan.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: investigasi race, latency, timeout, dan performa alur async.
- Alasan pakai: mempercepat RCA (root cause analysis).
- Kapan tidak dipakai: prototipe sekali pakai yang tidak butuh pemeliharaan.

## 7) Contoh Sederhana + Bedah Output
```js
function now() {
  return performance.now();
}

function createTraceId() {
  return Math.random().toString(36).slice(2, 10);
}

async function observedFetch(url) {
  const traceId = createTraceId();
  const t0 = now();
  console.log(`[${traceId}] start fetch ${url}`);

  try {
    const res = await fetch(url);
    const t1 = now();
    console.log(`[${traceId}] response status=${res.status} elapsed=${(t1 - t0).toFixed(1)}ms`);
    const data = await res.json();
    const t2 = now();
    console.log(`[${traceId}] parse done elapsed=${(t2 - t0).toFixed(1)}ms`);
    return data;
  } catch (err) {
    const te = now();
    console.log(`[${traceId}] error=${err.message} elapsed=${(te - t0).toFixed(1)}ms`);
    throw err;
  }
}
```
Bedah output:
1. Setiap operasi punya `traceId` unik.
2. Durasi tiap fase (`response`, `parse`) terlihat jelas.
3. Error path tetap memiliki jejak timing lengkap.

## 8) Jebakan Umum (Pitfalls)
- Log terlalu banyak tanpa struktur sehingga noise tinggi.
- Tidak memasukkan trace id, membuat korelasi lintas fase sulit.
- Hanya log sukses, tapi error path tidak diinstrumentasi.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const t0 = 10;
const t1 = 25;
console.log(`${t1 - t0}ms`);
```
Kunci:
- Output: `15ms`
- Alasan: durasi adalah selisih waktu akhir dan awal.

Drill 2:
```js
const id = 'abc123';
console.log(`[${id}] start`);
console.log(`[${id}] end`);
```
Kunci:
- Output dua log dengan trace id yang sama.
- Alasan: satu trace id harus melekat pada satu workflow.

Drill 3:
```js
const logs = [];
function log(phase) { logs.push(phase); }
log('start');
log('fetch');
log('apply');
console.log(logs.join(' > '));
```
Kunci:
- Output: `start > fetch > apply`
- Alasan: urutan phase membantu rekonstruksi workflow.

## 10) Debug Story
- Gejala: user melaporkan “kadang loading lama”, tapi tim tidak bisa reproduksi konsisten.
- Akar masalah: tidak ada timing per phase, sehingga bottleneck tidak terlihat.
- Langkah debug: tambah trace id + timing di phase `start`, `fetch`, `parse`, `apply`.
- Solusi: identifikasi phase paling lambat, lalu optimasi spesifik (caching, batching, cancel, dsb).

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa mendesain format log runtime yang konsisten.
- [ ] Bisa menambahkan timing end-to-end dan per phase.
- [ ] Bisa menggunakan trace id untuk investigasi bug async.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `advanced/13`, `advanced/14`, `advanced/15`.
- Topik terkait: `advanced/17-testing-runtime-ordering-dan-timing.md`.
- Referensi tambahan: structured logging dan performance instrumentation basics.

## 13) Model Mental
- Entitas utama: trace id, phase log, timer marker, outcome.
- Urutan proses: assign trace -> catat phase -> ukur durasi -> simpulkan bottleneck.
- Invariant (aturan yang selalu benar): observability yang baik harus konsisten di sukses dan error path.

## 14) Failure Case
- Skenario gagal: dua request overlap menulis log tanpa trace id, tim salah mengaitkan phase.
- Akar penyebab runtime: log tidak punya konteks korelasi.
- Perbaikan: wajibkan trace id per workflow dan sertakan pada semua phase log.

## 15) Spec Hint
- Section spec terkait (ringkas): timing/event loop ordering dan callback execution model.
- Kata kunci pencarian spec: `performance.now`, `event loop`, `task`, `promise jobs`.
