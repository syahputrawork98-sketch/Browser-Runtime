# Scheduling Prioritas dan Trade-Off

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham microtask/macrotask, fairness, cancellation, dan race guard.
  - [ ] Paham prinsip state consistency dan cleanup side effects.
- Kamus mini:
  - `[baru]` scheduling priority: urutan prioritas kerja berdasarkan dampak ke UX.
  - `[baru]` latency-sensitive work: pekerjaan yang harus cepat terlihat user (input/render feedback).
  - `[baru]` throughput work: pekerjaan batch yang tidak harus real-time per aksi.
  - `[ulang]` trade-off: keputusan yang mengorbankan satu sisi untuk sisi lain.

## 1) Pengantar Singkat Topik
Topik ini membahas bagaimana menentukan prioritas kerja di main thread agar UI tetap responsif sekaligus produktif. Fokusnya bukan “semua cepat”, tapi “yang penting dulu, sisanya terjadwal”.

## 2) Big Picture
Di aplikasi nyata, main thread sering menampung banyak pekerjaan bersamaan: update UI, parsing data, logging, sinkronisasi storage, analytics. Jika semua diperlakukan sama, pekerjaan non-kritis bisa menahan interaksi user. Dengan scheduling prioritas, kita memisahkan tugas kritis dari tugas background. Hasilnya UX lebih stabil dan sistem lebih mudah dioptimasi.

## 3) Small Picture
- Klasifikasikan tugas: kritis-UI, penting-fungsional, background.
- Eksekusi cepat untuk pekerjaan latency-sensitive.
- Pekerjaan berat dibagi jadi chunk dan dijadwalkan ulang.
- Pantau dampak scheduling lewat log/timing sederhana.

## 4) Wireframe Alur Konsep
```text
[user input]
  -> [update state + render feedback cepat]
  -> [queue pekerjaan non-kritis]
  -> [idle/chunk process]
  -> [finalize background tasks]
```
- Alur utama: user mendapat feedback cepat, background tetap jalan bertahap.
- Alur jalan: batch besar dipecah agar frame tidak drop.
- Alur error: tugas background memblokir jalur interaksi.

## 5) Analogi Dunia Nyata
Seperti dapur restoran: pesanan meja yang menunggu di depan diprioritaskan, sementara prep stok bumbu dilakukan saat jeda.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: dashboard interaktif, list besar, aplikasi dengan banyak side effect.
- Alasan pakai: menjaga responsiveness tanpa menghentikan pekerjaan penting lain.
- Kapan tidak dipakai: aplikasi kecil dengan beban kerja sangat rendah.

## 7) Contoh Sederhana + Bedah Output
```js
const queue = [];
let scheduled = false;

function scheduleBackground(task) {
  queue.push(task);
  if (!scheduled) {
    scheduled = true;
    setTimeout(flushBackground, 0); // yield ke event loop
  }
}

function flushBackground() {
  const start = performance.now();
  while (queue.length > 0 && performance.now() - start < 8) {
    const task = queue.shift();
    task();
  }
  if (queue.length > 0) {
    setTimeout(flushBackground, 0);
  } else {
    scheduled = false;
  }
}

function onUserInput(value) {
  console.log('critical: update UI', value); // prioritas tinggi
  scheduleBackground(() => console.log('bg: analytics', value));
  scheduleBackground(() => console.log('bg: cache sync', value));
}
```
Bedah output:
1. Jalur kritis (`update UI`) dijalankan langsung.
2. Tugas background diproses bertahap dalam batch kecil.
3. Event loop diberi ruang agar interaksi lain tetap responsif.

## 8) Jebakan Umum (Pitfalls)
- Menganggap semua task penting sehingga tidak ada prioritas nyata.
- Chunk terlalu besar sehingga masih mengunci main thread.
- Menjadwalkan ulang tanpa batas dan tanpa monitoring antrean.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
console.log('critical');
setTimeout(() => console.log('bg'), 0);
console.log('done');
```
Kunci:
- Output: `critical`, `done`, `bg`
- Alasan: tugas background dijadwalkan sebagai macrotask berikutnya.

Drill 2:
```js
const tasks = [1, 2, 3];
while (tasks.length) {
  console.log('work', tasks.shift());
}
console.log('ui');
```
Kunci:
- Output `work ...` selesai dulu baru `ui` log berikutnya.
- Alasan: loop sinkron tanpa yield memonopoli thread.

Drill 3:
```js
let i = 0;
function chunk() {
  i += 1;
  console.log('chunk', i);
  if (i < 3) setTimeout(chunk, 0);
}
chunk();
console.log('after start');
```
Kunci:
- Output: `chunk 1`, `after start`, lalu `chunk 2`, `chunk 3`
- Alasan: chunk berikutnya dijadwalkan ulang sehingga memberi fairness antar giliran.

## 10) Debug Story
- Gejala: input terasa lag saat sinkronisasi data besar berjalan.
- Akar masalah: sinkronisasi diproses sinkron penuh di jalur yang sama dengan interaksi user.
- Langkah debug: tandai task kritis vs background dan ukur durasi blok kerja.
- Solusi: pindahkan non-kritis ke scheduler background dengan chunk kecil.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa mengklasifikasikan tugas berdasarkan prioritas UX.
- [ ] Bisa menerapkan chunking untuk task berat.
- [ ] Bisa menjelaskan trade-off latency vs throughput.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `intermediate/07` sampai `intermediate/12`.
- Topik terkait: `advanced/14-batching-dom-read-write-untuk-runtime-stability.md`.
- Referensi tambahan: event loop scheduling patterns dan long-task mitigation.

## 13) Model Mental
- Entitas utama: task kritis, task background, event loop slots.
- Urutan proses: layani kritis -> jadwalkan background -> proses bertahap -> evaluasi dampak.
- Invariant (aturan yang selalu benar): pekerjaan non-kritis tidak boleh mengorbankan feedback interaktif user.

## 14) Failure Case
- Skenario gagal: parsing data besar dijalankan penuh setiap keypress.
- Akar penyebab runtime: task throughput ditempatkan di jalur latency-sensitive.
- Perbaikan: debounce input + chunk parsing + background scheduling.

## 15) Spec Hint
- Section spec terkait (ringkas): task queue processing, microtask checkpoint, dan rendering opportunity.
- Kata kunci pencarian spec: `event loop task`, `microtask checkpoint`, `long task`.
