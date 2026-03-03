# Batching DOM Read/Write untuk Runtime Stability

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham render tick dan timing update UI.
  - [ ] Paham dasar layout/reflow dan dampak operasi DOM berulang.
- Kamus mini:
  - `[baru]` DOM read: operasi membaca info layout/DOM (mis. `offsetHeight`, `getBoundingClientRect`).
  - `[baru]` DOM write: operasi mengubah DOM/style.
  - `[baru]` layout thrashing: pola read-write-read-write yang memicu reflow berulang.
  - `[ulang]` batching: mengelompokkan operasi sejenis agar overhead runtime berkurang.

## 1) Pengantar Singkat Topik
Topik ini membahas cara menjaga stabilitas runtime saat memanipulasi DOM intensif. Kunci utamanya adalah memisahkan fase read dan write supaya browser tidak dipaksa menghitung layout berkali-kali.

## 2) Big Picture
Banyak masalah performa UI muncul bukan karena algoritma berat, melainkan urutan operasi DOM yang buruk. Ketika read dan write bercampur di loop, browser dipaksa melakukan kalkulasi layout berulang. Ini memicu jank dan frame drop. Dengan batching read/write, kita menurunkan biaya layout dan membuat perilaku runtime lebih prediktif.

## 3) Small Picture
- Kumpulkan semua data yang perlu dibaca terlebih dahulu (read phase).
- Lakukan semua perubahan DOM/style setelahnya (write phase).
- Hindari read layout tepat setelah write pada frame yang sama.
- Untuk update besar, gunakan rAF sebagai batas frame write.

## 4) Wireframe Alur Konsep
```text
[collect elements]
  -> [READ: measure]
  -> [compute updates in JS]
  -> [WRITE: apply styles/content]
  -> [next frame]
```
- Alur utama: read lalu write sekali per batch.
- Alur jalan: update besar tetap mulus karena phase terpisah.
- Alur error: read/write campur acak memicu layout thrashing.

## 5) Analogi Dunia Nyata
Seperti tim survei bangunan: semua pengukuran dilakukan dulu, baru tim konstruksi mengubah struktur. Jika ukur dan bongkar selang-seling, proses jadi lambat dan kacau.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: virtual list sederhana, resize handler, animasi posisi, dashboard kartu dinamis.
- Alasan pakai: menurunkan reflow berulang dan menjaga frame stabil.
- Kapan tidak dipakai: update DOM sangat kecil dan jarang (biaya batching tidak signifikan).

## 7) Contoh Sederhana + Bedah Output
```js
const cards = Array.from(document.querySelectorAll('.card'));

function updateCards() {
  // READ phase
  const measurements = cards.map((el) => ({
    el,
    width: el.getBoundingClientRect().width
  }));

  // COMPUTE phase
  const nextWidths = measurements.map((m) => ({
    el: m.el,
    width: Math.max(180, m.width * 0.9)
  }));

  // WRITE phase
  for (const item of nextWidths) {
    item.el.style.width = `${item.width}px`;
  }
}

requestAnimationFrame(updateCards);
```
Bedah output:
1. Browser hanya perlu satu siklus read yang konsisten.
2. Update style dilakukan terkelompok pada write phase.
3. Risiko reflow berulang menurun dibanding pola read/write berselang-seling.

## 8) Jebakan Umum (Pitfalls)
- Memanggil `getBoundingClientRect()` di dalam loop setelah setiap `style` write.
- Menganggap batching hanya soal “kode lebih rapi”, padahal ini soal biaya runtime.
- Menjalankan write terlalu sering di luar kontrol frame.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
el.style.width = '200px';       // write
console.log(el.offsetWidth);    // read
el.style.width = '220px';       // write
console.log(el.offsetWidth);    // read
```
Kunci:
- Dampak: berpotensi memicu reflow berulang.
- Alasan: read setelah write memaksa browser sinkronkan layout berkali-kali.

Drill 2:
```js
const w = el.offsetWidth;       // read
const h = el.offsetHeight;      // read
el.style.width = `${w + 10}px`; // write
el.style.height = `${h + 10}px`;// write
```
Kunci:
- Dampak: lebih stabil dibanding read/write campur.
- Alasan: read phase selesai sebelum write phase dimulai.

Drill 3:
```js
requestAnimationFrame(() => {
  const w = el.getBoundingClientRect().width;
  el.style.transform = `translateX(${w}px)`;
});
```
Kunci:
- Pola: read + write terikat frame callback, umumnya lebih terkontrol.
- Alasan: rAF memberi titik sinkron alami sebelum paint frame.

## 10) Debug Story
- Gejala: scroll list terasa patah-patah setelah fitur resize otomatis ditambahkan.
- Akar masalah: setiap item melakukan read/write berselang-seling di handler scroll.
- Langkah debug: rekam Performance tab, cari banyak layout/recalculate style berulang.
- Solusi: refactor ke batching read->compute->write dan throttle pemicu.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa mengidentifikasi pola layout thrashing.
- [ ] Bisa memisahkan read phase dan write phase.
- [ ] Bisa menjelaskan kenapa batching meningkatkan runtime stability.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/05-render-tick-dan-ui-update-timing.md`.
- Topik terkait: `advanced/15-backpressure-dan-request-burst-control.md`.
- Referensi tambahan: layout/reflow basics dan performance profiling browser.

## 13) Model Mental
- Entitas utama: DOM read cost, DOM write cost, layout engine, frame budget.
- Urutan proses: read terkelompok -> compute -> write terkelompok -> paint.
- Invariant (aturan yang selalu benar): semakin sering read/write saling menyela, semakin besar risiko reflow berulang.

## 14) Failure Case
- Skenario gagal: dalam loop 100 elemen, tiap iterasi melakukan `write -> read -> write`.
- Akar penyebab runtime: browser dipaksa melakukan kalkulasi layout berkali-kali per iterasi.
- Perbaikan: kumpulkan measurement dulu untuk semua elemen, lalu apply write dalam satu batch.

## 15) Spec Hint
- Section spec terkait (ringkas): update rendering pipeline (style/layout/paint) dan timing frame.
- Kata kunci pencarian spec: `update the rendering`, `layout`, `paint`, `animation frame callbacks`.
