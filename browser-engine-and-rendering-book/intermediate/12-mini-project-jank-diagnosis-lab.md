# Mini Project: Jank Diagnosis Lab

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan intermediate 07-11.
  - [ ] Sudah terbiasa rekam dan membaca trace di DevTools.
- Kamus mini:
  - `[baru]` diagnosis loop: siklus baseline -> hipotesis -> eksperimen -> validasi.
  - `[baru]` performance budget guardrail: batas metrik yang dipakai sebagai syarat lolos perubahan.
  - `[ulang]` jank: frame rendering terlambat sehingga interaksi terlihat tersendat.

## 1) Pengantar Singkat Topik
Mini project ini mensimulasikan workflow nyata ketika UI mengalami jank campuran (scroll, animasi, dan list update). Fokus utamanya adalah pengambilan keputusan berbasis evidence, bukan optimasi acak.

## 2) Big Picture
Di sistem produksi, jank jarang disebabkan satu faktor tunggal. Biasanya ada kombinasi handler berat, layout spike, paint luas, dan asset loading timing yang kurang tepat. Lab ini memaksa kita membedah masalah secara sistematis dan menetapkan prioritas perbaikan. Hasil akhirnya adalah playbook diagnosis yang bisa dipakai ulang tim untuk regression prevention.

## 3) Small Picture
- Tetapkan skenario uji yang repeatable sebelum mengubah kode.
- Pilih bottleneck tertinggi, lakukan perubahan minimal, ukur ulang.
- Dokumentasikan hasil gagal/sukses agar keputusan teknik bisa diaudit.

## 4) Wireframe Alur Konsep
```text
[baseline scenario] -> [trace + isolate bottleneck] -> [targeted fix] -> [retest + compare budget]
```
- Alur utama: tiap iterasi menghasilkan perbaikan metrik terukur.
- Alur jalan: jika perbaikan tidak signifikan, revisi hipotesis dan ulangi.
- Alur error: terlalu banyak perubahan sekaligus membuat dampak tidak bisa diatribusikan.

## 5) Analogi Dunia Nyata
Seperti diagnosa mesin pabrik: ukur output awal, ganti satu komponen kritis, ukur ulang di kondisi sama sebelum lanjut ke komponen berikutnya.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: melatih incident-style debugging performa pada UI interaktif.
- Alasan pakai: membangun kebiasaan keputusan berbasis trace dan metric guardrail.
- Kapan tidak dipakai: saat kamu hanya butuh demo konsep singkat tanpa target kualitas produksi.

## 7) Contoh Sederhana + Bedah Output
```js
// Simulasi beban campuran pada interaksi scroll + filter
const list = document.querySelector(".list");
window.addEventListener("scroll", () => {
  const items = list.querySelectorAll(".item");
  for (const item of items) {
    item.style.top = `${window.scrollY % 100}px`;
    // eslint-disable-next-line no-unused-expressions
    item.offsetHeight;
  }
});
```

Bedah output:
1. Kode mencampur write + read dalam loop saat scroll, berisiko layout thrashing.
2. Properti `top` menambah potensi biaya layout/paint.
3. Ini kandidat baseline yang baik untuk latihan diagnosis dan refactor bertahap.

## 8) Jebakan Umum (Pitfalls)
- Mengubah beberapa area kode sekaligus tanpa checkpoint metric.
- Mengklaim perbaikan hanya dari subjektif "terasa lebih halus".
- Mengabaikan variasi perangkat/jaringan saat validasi.
- Regression risk: optimasi lokal memperbaiki satu flow tetapi merusak flow lain karena guardrail tidak diverifikasi lintas skenario.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const start = performance.now();
while (performance.now() - start < 20) {}
requestAnimationFrame(() => {
  document.body.classList.toggle("active");
});
```
Kunci 1:
- Output: class tetap toggle, tetapi frame bisa terlambat.
- Alasan: long sync work sebelum `requestAnimationFrame` mengurangi waktu render frame saat ini.

Drill 2:
```js
let dirty = false;
window.addEventListener("scroll", () => {
  dirty = true;
});
function loop() {
  if (dirty) {
    dirty = false;
    document.body.style.setProperty("--scroll-y", String(window.scrollY));
  }
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
```
Kunci 2:
- Output: nilai CSS custom property ter-update mengikuti scroll.
- Alasan: pekerjaan dibatasi di frame loop tunggal; ini biasanya lebih stabil daripada kerja berat di tiap event scroll.

## 10) Debug Story
- Gejala: halaman katalog macet saat user scroll sambil membuka filter dan hover kartu produk.
- Akar masalah: kombinasi scroll handler berat, animasi non-compositor-friendly, dan list DOM terlalu besar.
- Langkah debug:
  - Definisikan 3 skenario tetap: scroll-only, scroll+filter, scroll+hover.
  - Rekam baseline tiap skenario dan catat metric kunci (`p95 frame`, long task, layout spike).
  - Terapkan perbaikan bertahap: batching read/write, transform animation, windowing list, stabilisasi image/font.
  - Rekam ulang dan bandingkan terhadap budget guardrail.
- Solusi: jank turun setelah bottleneck terbesar ditangani lebih dulu dan validasi dilakukan lintas skenario, bukan satu flow.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menyiapkan diagnosis loop lengkap dari baseline sampai retest.
- [ ] Bisa memilih prioritas perbaikan berdasarkan data trace.
- [ ] Bisa menunjukkan hasil sebelum/sesudah terhadap budget yang disepakati.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: intermediate 07-11.
- Topik terkait: advanced 13 (render budget) dan 16 (measurement playbook).
- Referensi tambahan: dokumentasi Performance panel dan Web Vitals debugging.

## A) Pipeline Breakdown
- Parse: umumnya stabil, bukan fokus utama lab ini.
- Style: dapat melonjak saat banyak class/state berubah bersamaan.
- Layout: sering jadi bottleneck saat read/write bercampur.
- Paint: membesar saat area visual berubah luas.
- Composite: jalur ideal untuk animasi ringan; akan terganggu jika stage sebelumnya terlalu berat.

## B) Perf Metric yang Relevan
- Metric: p95 frame time untuk tiap skenario diagnosis.
- Target: <= 16.7 ms pada perangkat baseline.
- Dampak jika buruk: jank masih terasa pada interaksi prioritas.
- Metric: jumlah long task (>50 ms) selama 10 detik interaksi.
- Target: 0 pada flow utama setelah optimasi.
- Dampak jika buruk: input delay dan animasi drop frame tetap tinggi.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Rendering` (FPS meter, paint flashing)
   - `Network` (untuk cek kontribusi asset loading saat first interaction)
2. Langkah observasi:
   - Tetapkan script uji manual yang sama untuk tiap iterasi.
   - Rekam baseline dan simpan metric per skenario.
   - Terapkan satu kelas perubahan per iterasi, lalu rekam ulang dan bandingkan.
3. Indikator sukses:
   - Semua skenario utama lolos budget guardrail yang disepakati.
   - Tidak ada regresi signifikan pada skenario sampingan setelah optimasi utama.

