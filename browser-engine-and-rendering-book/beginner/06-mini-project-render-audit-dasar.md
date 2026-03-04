# Mini Project: Render Audit Dasar

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan modul 01-05 di level beginner.
  - [ ] Bisa menjalankan contoh HTML/CSS/JS sederhana di browser.
- Kamus mini:
  - `[baru]` baseline trace: rekaman performa awal sebelum optimasi.
  - `[baru]` render budget: batas waktu render per frame agar UI tetap halus.
  - `[ulang]` bottleneck: tahap paling mahal yang paling memengaruhi kelancaran interaksi.

## 1) Pengantar Singkat Topik
Mini project ini melatih kamu melakukan audit performa rendering dari awal sampai validasi hasil. Fokusnya bukan sekadar "terasa lebih cepat", tetapi ada bukti lewat metrik.

## 2) Big Picture
Di proyek nyata, masalah performa sering campuran antara style, layout, paint, dan composite. Tanpa alur audit yang rapi, optimasi sering salah sasaran. Mini project ini memberi kerangka kerja sederhana: rekam baseline, pilih satu bottleneck prioritas, lakukan perbaikan kecil, lalu ukur ulang. Setelah menguasainya, kamu bisa masuk level intermediate dengan kebiasaan optimasi berbasis evidence.

## 3) Small Picture
- Rekam baseline sebelum ubah kode.
- Pilih satu masalah terbesar dulu (jangan optimasi semua sekaligus).
- Bandingkan hasil before/after pada skenario interaksi yang sama.

## 4) Wireframe Alur Konsep
```text
[baseline trace] -> [identifikasi bottleneck] -> [optimasi terarah] -> [retest + bandingkan metric]
```
- Alur utama: satu hipotesis optimasi menghasilkan perbaikan metric yang terukur.
- Alur jalan: jika metric belum membaik, revisi hipotesis dan ulangi.
- Alur error: banyak perubahan sekaligus membuat penyebab peningkatan tidak jelas.

## 5) Analogi Dunia Nyata
Seperti servis motor: cek performa awal, ganti komponen yang paling bermasalah dulu, lalu test ride dengan rute yang sama untuk lihat efeknya.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: audit performa UI kecil (dashboard, list interaktif, panel filter).
- Alasan pakai: memberi kebiasaan debugging render yang sistematis dan bisa direplikasi.
- Kapan tidak dipakai: saat bug utama bukan rendering (misal API timeout, data corrupt).

## 7) Contoh Sederhana + Bedah Output
```js
// Baseline: simulasi update UI berulang
const cards = document.querySelectorAll(".card");
for (const card of cards) {
  card.style.width = "260px";
  // eslint-disable-next-line no-unused-expressions
  card.offsetWidth;
  card.style.boxShadow = "0 20px 40px rgba(0,0,0,0.25)";
}
```
Bedah output:
1. Kode memicu kombinasi biaya layout (`offsetWidth`) dan paint (`boxShadow`).
2. Pada jumlah card besar, interaksi cenderung jank.
3. Ini cocok jadi baseline sebelum mencoba optimasi batch read/write dan efek visual lebih ringan.

## 8) Jebakan Umum (Pitfalls)
- Mengubah banyak variabel sekaligus sehingga hasil tidak bisa dianalisis.
- Tidak menyamakan skenario uji before vs after.
- Menilai performa hanya dari perasaan tanpa trace/metrik.
- Regression risk: optimasi yang lolos di satu skenario gagal pada flow user lain karena tidak diuji ulang.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
function runTest() {
  const modal = document.querySelector(".modal");
  modal.classList.toggle("open");
}

for (let i = 0; i < 50; i++) runTest();
```
Kunci:
- Output: class `open` berganti berulang, UI bisa tampak flicker/berubah cepat.
- Alasan: toggle cepat dapat memicu style/layout/paint berulang; tanpa trace, sulit tahu bottleneck dominan.

## 10) Debug Story
- Gejala: saat user membuka filter panel dan scroll list, UI terasa patah-patah.
- Akar masalah: panel animasi memakai `top`, list item punya shadow berat, dan ada read ukuran di loop update.
- Langkah debug:
  - Rekam `Performance` untuk skenario "open filter + scroll 3 detik".
  - Catat stage paling mahal (layout vs paint).
  - Terapkan 1-2 perubahan terarah (contoh: `transform` untuk panel, kurangi shadow).
  - Rekam ulang skenario sama.
- Solusi: hasil terbaik datang dari kombinasi animasi compositor-friendly + pengurangan paint area.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membuat baseline trace dan menyebut bottleneck utamanya.
- [ ] Bisa menjelaskan alasan teknis perubahan optimasi yang dipilih.
- [ ] Bisa menunjukkan perbandingan metric before/after.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: modul 02 (style), 03 (layout), 04 (paint), 05 (composite).
- Topik terkait: layout thrashing dan animation pipeline di intermediate.
- Referensi tambahan: dokumentasi Chrome DevTools Performance + Rendering.

## A) Pipeline Breakdown
- Parse: asumsi sudah stabil dari dokumen aplikasi.
- Style: dievaluasi saat class/state berubah selama interaksi.
- Layout: dievaluasi saat ada perubahan geometri atau forced read.
- Paint: dievaluasi saat properti visual berubah (warna, shadow, gradient).
- Composite: dievaluasi saat layer digabung pada animasi/interaksi.

## B) Perf Metric yang Relevan
- Metric: p95 frame time pada skenario uji utama.
- Target: p95 <= 16.7 ms pada jalur interaksi prioritas.
- Dampak jika buruk: interaksi terasa tidak responsif dan FPS turun.
- Metric: durasi stage dominan (`Recalculate Style`/`Layout`/`Paint`) sebelum vs sesudah optimasi.
- Target: turun minimal 20% pada stage bottleneck utama.
- Dampak jika buruk: perubahan kode tidak memberi dampak nyata ke pengalaman pengguna.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Rendering` (`Paint flashing`, FPS meter)
2. Langkah observasi:
   - Rekam baseline 2-3 kali pada skenario interaksi yang sama.
   - Catat nilai p95 frame time dan stage paling mahal.
   - Terapkan optimasi kecil, lalu rekam ulang dengan langkah identik.
3. Indikator sukses:
   - Stage bottleneck turun konsisten pada beberapa rekaman.
   - p95 frame time membaik dan interaksi terasa lebih halus.

