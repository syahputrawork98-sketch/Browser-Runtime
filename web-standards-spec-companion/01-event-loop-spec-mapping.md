# Event Loop: Mapping Praktik ke Spec

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham Promise, `setTimeout`, dan urutan output dasar.
  - [ ] Paham istilah task dan microtask.
- Kamus mini:
  - `[baru]` job: istilah formal ECMAScript untuk unit kerja async tertentu.
  - `[baru]` task: istilah HTML event loop untuk task queue.

## 1) Pengantar Singkat Topik
Topik ini menjembatani istilah sehari-hari developer dengan istilah formal pada spesifikasi web.

## 2) Big Picture
Perdebatan teknis sering buntu karena istilah dipakai campur aduk. Dengan mapping ke spec, kita punya rujukan objektif untuk memvalidasi perilaku runtime. Dampaknya adalah diskusi teknis lebih presisi dan keputusan implementasi lebih kuat.

## 3) Small Picture
- Ambil klaim perilaku runtime.
- Petakan ke istilah dan clause spec.
- Tulis interpretasi yang bisa dipakai di kode.

## 4) Wireframe Alur Konsep
```text
[klaim runtime] -> [istilah spec] -> [clause] -> [catatan implementasi]
```
- Alur utama: klaim cocok dengan clause.
- Alur jalan: klaim butuh konteks lintas spec.
- Alur error: klaim tidak punya dukungan spec yang tepat.

## 5) Analogi Dunia Nyata
Seperti menerjemahkan bahasa teknisi lapangan ke dokumen standar resmi agar semua tim sepakat definisi.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: validasi asumsi runtime dan code review teknis.
- Alasan pakai: mengurangi debat berbasis opini.
- Kapan tidak dipakai: untuk contoh belajar yang sangat pemula.

## 7) Contoh Sederhana + Bedah Output
```js
console.log('start');
Promise.resolve().then(() => console.log('microtask'));
setTimeout(() => console.log('task'), 0);
console.log('end');
```
Bedah output:
1. `start`, `end` berjalan sinkron.
2. Callback promise masuk jalur microtask/job.
3. Timer berjalan pada task berikutnya.

## 8) Spec Links
- HTML Standard (event loop, task, microtask): https://html.spec.whatwg.org/
- ECMAScript (jobs/promise reaction jobs): https://tc39.es/ecma262/

## 9) Clause Mapping
- Klaim teknis: promise callback dieksekusi sebelum timer callback.
- Clause spec terkait: HTML microtask checkpoint + ECMAScript PromiseJobs.
- Catatan interpretasi: browser menguras microtask sebelum melanjutkan task berikutnya.

## 10) Terminology Mapping
- Istilah praktis di kode: "microtask".
- Istilah formal di spec: "job queue/PromiseJobs" (ECMAScript) dan "microtask queue" (HTML integration layer).

## 11) Implementation Notes
- Perilaku engine/browser: secara umum konsisten pada browser modern.
- Perbedaan lintas browser: detail scheduling non-standar bisa berbeda untuk API tertentu, jadi validasi tetap diperlukan.

## 12) Checkpoint Kelulusan Topik
- [ ] Bisa memetakan 1 perilaku runtime ke clause spec.
- [ ] Bisa menjelaskan istilah praktis vs istilah formal.
- [ ] Bisa menulis catatan interpretasi yang tidak ambigu.

## 13) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: event loop dasar.
- Topik terkait: call stack/task/microtask.
- Referensi tambahan: dokumentasi MDN tentang event loop.
