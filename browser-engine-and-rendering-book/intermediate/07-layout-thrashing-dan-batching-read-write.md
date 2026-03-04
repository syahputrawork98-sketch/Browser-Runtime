# Layout Thrashing dan Batching Read/Write

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan beginner 01-06.
  - [ ] Paham reflow trigger, forced layout, dan dasar profiling `Performance`.
- Kamus mini:
  - `[ulang]` layout thrashing: pola write-read-write-read yang memicu layout sinkron berulang.
  - `[baru]` read phase: fase pengambilan data layout (`offsetWidth`, `getBoundingClientRect`) sebelum menulis style.
  - `[baru]` write phase: fase update style/DOM setelah semua data read terkumpul.

## 1) Pengantar Singkat Topik
Topik ini membahas cara mendiagnosis dan menghilangkan layout thrashing pada interaksi UI nyata. Fokusnya adalah pola eksekusi yang bisa direplikasi tim, bukan trik satu kali.

## 2) Big Picture
Di level intermediate, bottleneck sering datang dari kombinasi state update dan DOM access yang saling menyela. Kode terlihat "normal", tetapi urutan operasi membuat browser dipaksa menghitung layout berulang kali. Dengan memisahkan read phase dan write phase, kamu menurunkan biaya layout tanpa mengorbankan behavior fitur. Dampaknya paling terasa pada list besar, drag, resize, dan sticky UI.

## 3) Small Picture
- Semua read layout dilakukan dulu, lalu write dilakukan dalam batch.
- Hindari read layout di dalam loop yang sedang menulis style.
- Gunakan `requestAnimationFrame` sebagai boundary commit visual saat perlu.

## 4) Wireframe Alur Konsep
```text
[event/update state] -> [collect read] -> [batch write] -> [single layout impact]
```
- Alur utama: satu interaksi menghasilkan layout cost terkontrol.
- Alur jalan: data geometri dikumpulkan dulu, perubahan DOM dieksekusi belakangan.
- Alur error: util helper melakukan read tersembunyi saat write phase sehingga thrashing kembali muncul.

## 5) Analogi Dunia Nyata
Seperti audit gudang: catat ukuran semua rak dulu, baru pindahkan barang sekaligus. Jika tiap pindah barang kamu ukur ulang seluruh gudang, pekerjaan jadi lambat.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: virtualized list sederhana, auto-resize panel, sticky header, dan drag interaction.
- Alasan pakai: mengurangi forced layout spike yang sering muncul di jalur interaksi kritis.
- Kapan tidak dipakai: ketika bottleneck utama sudah terbukti di paint-heavy effect atau network wait.

## 7) Contoh Sederhana + Bedah Output
```js
// Anti-pattern: write lalu read di setiap iterasi
const rows = Array.from(document.querySelectorAll(".row"));
for (const row of rows) {
  row.style.width = "320px"; // write
  // eslint-disable-next-line no-unused-expressions
  row.getBoundingClientRect().height; // read (berpotensi memaksa layout sinkron)
}
```

```js
// Better pattern: read phase -> write phase
const rows2 = Array.from(document.querySelectorAll(".row"));
const heights = rows2.map((row) => row.getBoundingClientRect().height); // read phase
rows2.forEach((row, i) => {
  row.style.width = `${300 + Math.min(heights[i], 40)}px`; // write phase
});
```

Bedah output:
1. Versi anti-pattern cenderung memicu banyak forced layout saat iterasi.
2. Versi batch memindahkan read sebelum write sehingga layout bisa lebih stabil.
3. Hasil visual tetap sama, tetapi profile timeline biasanya lebih ringan.

## 8) Jebakan Umum (Pitfalls)
- Mengira helper util "aman", padahal di dalamnya ada `offsetWidth`.
- Memecah update ke banyak callback kecil sehingga urutan read/write campur lagi.
- Melakukan batch write, tetapi masih menyisipkan read di callback animasi yang sama.
- Regression risk: penambahan fitur baru (tooltip/sticky) menyisipkan read layout di jalur render yang sebelumnya sudah bersih.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const cards = document.querySelectorAll(".card");
for (const card of cards) {
  card.style.height = "140px";
  // eslint-disable-next-line no-unused-expressions
  card.offsetHeight;
}
```
Kunci 1:
- Output: style tetap diterapkan, tetapi berisiko jank saat jumlah card banyak.
- Alasan: pola write lalu read di loop memicu forced layout berulang.

Drill 2:
```js
const cards2 = Array.from(document.querySelectorAll(".card"));
const sizes = cards2.map((c) => c.offsetHeight);

requestAnimationFrame(() => {
  cards2.forEach((c, i) => {
    c.style.height = `${sizes[i] + 8}px`;
  });
});
```
Kunci 2:
- Output: tinggi card naik seragam sesuai data awal.
- Alasan: read dilakukan sebelum write; write dibatch pada boundary frame yang jelas, sehingga layout thrash berkurang.

## 10) Debug Story
- Gejala: saat user scroll sambil filter list, frame drop naik tajam.
- Akar masalah: handler filter menulis style item lalu memanggil helper yang membaca `getBoundingClientRect` per item.
- Langkah debug:
  - Rekam `Performance` saat skenario scroll + filter.
  - Cari pola berulang `Layout` dekat call stack handler list.
  - Pisahkan read phase dan write phase, lalu rekam ulang skenario identik.
- Solusi: memindahkan seluruh read ke awal siklus update dan commit write dalam batch, menurunkan spike `Layout`.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menemukan lokasi thrashing dari timeline + call stack.
- [ ] Bisa memisahkan read/write pada kode interaksi nyata.
- [ ] Bisa membuktikan penurunan layout spike lewat before/after trace.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: beginner 03 (reflow trigger) dan 06 (audit dasar).
- Topik terkait: scroll jank dan animation pipeline di modul 08-09.
- Referensi tambahan: dokumentasi forced synchronous layout dan rendering performance.

## A) Pipeline Breakdown
- Parse: umumnya tidak jadi bottleneck pada kasus thrashing.
- Style: update style tetap terjadi, tapi bukan selalu biaya terbesar.
- Layout: stage utama yang melonjak saat read/write bercampur.
- Paint: mengikuti perubahan visual setelah layout commit.
- Composite: menggabungkan hasil frame; dampaknya sekunder terhadap thrashing.

## B) Perf Metric yang Relevan
- Metric: jumlah forced layout event per interaksi kritis.
- Target: 0 pada jalur utama interaksi (filter/drag/resize).
- Dampak jika buruk: jank muncul meski JavaScript execution time terlihat pendek.
- Metric: p95 durasi `Layout` pada trace interaksi.
- Target: p95 < 4 ms pada skenario baseline perangkat tim.
- Dampak jika buruk: frame budget terpotong besar dan FPS tidak stabil.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Performance Insights` (jika tersedia)
2. Langkah observasi:
   - Rekam skenario interaksi sebelum refactor (30-60 detik).
   - Identifikasi event `Layout` berulang dan call stack JS pemicunya.
   - Terapkan batching read/write, lalu rekam ulang skenario yang sama.
3. Indikator sukses:
   - Jumlah forced layout menurun signifikan.
   - p95 durasi `Layout` turun dan frame time lebih stabil.

