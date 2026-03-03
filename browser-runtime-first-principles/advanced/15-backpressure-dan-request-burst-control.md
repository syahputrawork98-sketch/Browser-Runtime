# Backpressure dan Request Burst Control

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham cancellation, race guard, dan scheduling prioritas.
  - [ ] Paham dampak request overlap pada state/UI.
- Kamus mini:
  - `[baru]` backpressure: mekanisme menahan laju produksi kerja agar konsumen tidak kewalahan.
  - `[baru]` burst: lonjakan request dalam waktu singkat.
  - `[baru]` concurrency limit: batas jumlah request paralel yang boleh aktif.
  - `[ulang]` queue: antrean pekerjaan menunggu dieksekusi.

## 1) Pengantar Singkat Topik
Topik ini membahas strategi menahan ledakan request agar sistem tetap stabil. Fokusnya: membatasi concurrency, antri pekerjaan, dan memilih drop/cancel policy yang tepat.

## 2) Big Picture
Ketika user mengetik cepat atau event sering memicu fetch, request bisa meledak. Jika semua langsung ditembak, bandwidth, server, dan state management menjadi tidak stabil. Backpressure membantu menyeimbangkan throughput dan responsivitas. Tujuan akhirnya bukan “request sebanyak mungkin”, tapi “request secukupnya dengan hasil yang konsisten”.

## 3) Small Picture
- Batasi request aktif dengan concurrency limit.
- Simpan request berikutnya di queue atau gabungkan (coalesce) jika perlu.
- Tentukan kebijakan saat penuh: drop oldest, drop newest, atau cancel previous.
- Pastikan hasil apply mengikuti policy yang dipilih.

## 4) Wireframe Alur Konsep
```text
[event burst]
  -> [enqueue jobs]
  -> [run max N concurrent]
  -> [job selesai -> ambil job berikutnya]
  -> [state update terkontrol]
```
- Alur utama: beban tinggi tetap terkendali karena ada limit.
- Alur jalan: queue menahan lonjakan sementara tanpa overload.
- Alur error: tanpa limit, request overlap berlebihan memicu state chaos.

## 5) Analogi Dunia Nyata
Seperti gerbang tol dengan jumlah loket terbatas: kendaraan masuk antre, bukan semua dipaksa lewat satu titik sekaligus.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: live search, sync batch, upload banyak item, infinite scroll.
- Alasan pakai: menjaga stabilitas client/server saat trafik meledak.
- Kapan tidak dipakai: aplikasi kecil dengan request jarang dan non-overlap.

## 7) Contoh Sederhana + Bedah Output
```js
const queue = [];
let active = 0;
const LIMIT = 2;

function runNext() {
  if (active >= LIMIT) return;
  const job = queue.shift();
  if (!job) return;

  active += 1;
  job()
    .catch(() => {})
    .finally(() => {
      active -= 1;
      runNext();
    });
}

function scheduleRequest(task) {
  queue.push(task);
  runNext();
}

function fakeFetch(id, ms) {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log('done', id);
      resolve(id);
    }, ms);
  });
}

scheduleRequest(() => fakeFetch('A', 400));
scheduleRequest(() => fakeFetch('B', 300));
scheduleRequest(() => fakeFetch('C', 100));
scheduleRequest(() => fakeFetch('D', 200));
```
Bedah output:
1. Hanya dua request aktif bersamaan (`LIMIT = 2`).
2. Request berikutnya menunggu antrean.
3. Sistem tetap stabil meski burst beberapa tugas sekaligus.

## 8) Jebakan Umum (Pitfalls)
- Concurrency limit ada, tapi hasil apply tetap tidak dijaga (masih rawan stale).
- Queue tak terbatas tanpa policy drop/cancel saat tekanan tinggi.
- Limit terlalu rendah/tinggi tanpa observasi dampak.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const LIMIT = 1;
let active = 0;
console.log(active < LIMIT ? 'run' : 'wait');
```
Kunci:
- Output: `run`
- Alasan: masih ada slot concurrency.

Drill 2:
```js
const q = ['x', 'y', 'z'];
console.log(q.shift());
console.log(q.length);
```
Kunci:
- Output:
  - `x`
  - `2`
- Alasan: satu job keluar dari antrean.

Drill 3:
```js
let active = 2;
const LIMIT = 2;
console.log(active >= LIMIT ? 'queue it' : 'run now');
```
Kunci:
- Output: `queue it`
- Alasan: slot penuh, job baru harus antre.

## 10) Debug Story
- Gejala: saat user scroll cepat, request list menumpuk dan UI jadi tidak konsisten.
- Akar masalah: semua trigger langsung membuat fetch tanpa limit.
- Langkah debug: log `queue length`, `active count`, dan request lifecycle.
- Solusi: terapkan concurrency limit + queue policy + guard apply state.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan kapan backpressure dibutuhkan.
- [ ] Bisa implementasi concurrency limit sederhana.
- [ ] Bisa memilih policy burst handling yang sesuai konteks fitur.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `intermediate/08`, `intermediate/09`, `advanced/13`.
- Topik terkait: `advanced/16-observability-runtime-log-timing-trace.md`.
- Referensi tambahan: throughput vs latency trade-off di sistem interaktif.

## 13) Model Mental
- Entitas utama: producer (event), queue, worker slots, consumer (state/UI).
- Urutan proses: produce -> queue -> process with limit -> commit result.
- Invariant (aturan yang selalu benar): saat producer lebih cepat dari consumer, tanpa backpressure antrean/overlap akan tak terkendali.

## 14) Failure Case
- Skenario gagal: satu input memicu 20 request paralel dalam 1 detik.
- Akar penyebab runtime: tidak ada concurrency control atau cancel policy.
- Perbaikan: limit request aktif, antrekan sisanya, dan gunakan cancellation/merge pada request yang tidak relevan.

## 15) Spec Hint
- Section spec terkait (ringkas): fetch lifecycle, task scheduling, dan interaksi event loop dengan async completion.
- Kata kunci pencarian spec: `fetch`, `task queue`, `promise resolution`, `abort`.
