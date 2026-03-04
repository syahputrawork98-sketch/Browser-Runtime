# Scroll Jank dan Main Thread Contention

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan intermediate 07 (layout thrashing).
  - [ ] Paham event loop dasar dan perbedaan tugas JS vs rendering pipeline.
- Kamus mini:
  - `[baru]` scroll jank: kondisi scroll terasa patah/tersendat karena frame tidak ter-render tepat waktu.
  - `[baru]` main thread contention: banyak pekerjaan bersaing di main thread sehingga tugas kritis terlambat.
  - `[ulang]` long task: pekerjaan di main thread yang berjalan terlalu lama dan menghambat respons UI.

## 1) Pengantar Singkat Topik
Topik ini membahas kenapa scroll bisa patah walau FPS sesekali terlihat tinggi. Fokusnya adalah memecah kompetisi kerja di main thread agar jalur scroll tetap lancar.

## 2) Big Picture
Saat user scroll, browser harus menangani input, update style/layout/paint, dan JavaScript hampir bersamaan. Jika handler scroll terlalu berat atau bersaing dengan task lain (parsing data, render list, logika sinkron), frame budget cepat habis. Dengan mengenali pola contention, kita bisa memindahkan, menunda, atau membagi pekerjaan non-kritis. Hasilnya adalah scroll yang lebih stabil di perangkat menengah ke bawah.

## 3) Small Picture
- Scroll handler harus ringan, hindari kerja sinkron berat di setiap event.
- Pekerjaan non-kritis dijadwalkan ulang (debounce/throttle/scheduler).
- Profiling harus melihat gabungan input, script, layout, dan paint, bukan satu layer saja.

## 4) Wireframe Alur Konsep
```text
[scroll input] -> [main thread queue] -> [script + render tasks] -> [frame commit]
```
- Alur utama: tugas kritis scroll selesai cepat lalu frame dikomit tepat waktu.
- Alur jalan: tugas non-kritis ditunda agar tidak mengganggu jalur interaksi.
- Alur error: task sinkron panjang mendorong render melewati budget 16.7 ms.

## 5) Analogi Dunia Nyata
Seperti satu loket pelayanan dipakai sekaligus untuk antrian cepat dan berkas panjang; jika berkas panjang tidak dipisah, semua antrian jadi lambat.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: feed scroll panjang, dashboard dengan widget banyak, dan halaman dengan analytics/event tracking intens.
- Alasan pakai: menjaga respons scroll pada kondisi beban campuran JS + rendering.
- Kapan tidak dipakai: jika bottleneck utama murni ada di network stall atau decode asset di luar main-thread contention.

## 7) Contoh Sederhana + Bedah Output
```js
const list = document.querySelector(".feed");

window.addEventListener("scroll", () => {
  // kerja berat tiap event scroll (anti-pattern)
  const items = list.querySelectorAll(".item");
  let total = 0;
  for (const item of items) {
    total += item.getBoundingClientRect().top;
  }
  list.dataset.metric = String(total);
});
```

Bedah output:
1. Handler scroll dipanggil sangat sering saat gesture berlangsung.
2. Query + read layout massal di tiap event meningkatkan contention di main thread.
3. Akibatnya frame commit terlambat dan scroll terasa jank.

## 8) Jebakan Umum (Pitfalls)
- Memasang logic analitik sinkron langsung di callback scroll.
- Menggabungkan update DOM berat dengan perhitungan data besar pada event yang sama.
- Hanya optimasi CSS padahal bottleneck utama di script task panjang.
- Regression risk: fitur baru menambah observer/listener yang memproses data berat di jalur scroll tanpa budget check.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
window.addEventListener("scroll", () => {
  const start = performance.now();
  while (performance.now() - start < 25) {
    // simulate heavy sync work
  }
});
```
Kunci 1:
- Output: tidak ada error, tetapi scroll sangat mungkin patah.
- Alasan: task sinkron 25 ms sudah melebihi budget frame 16.7 ms.

Drill 2:
```js
let scheduled = false;
window.addEventListener("scroll", () => {
  if (scheduled) return;
  scheduled = true;
  requestAnimationFrame(() => {
    document.body.dataset.y = String(window.scrollY);
    scheduled = false;
  });
});
```
Kunci 2:
- Output: nilai `data-y` tetap ter-update selama scroll.
- Alasan: update dibatasi satu kali per frame boundary, sehingga pressure main thread berkurang.

## 10) Debug Story
- Gejala: scroll feed terasa macet saat komponen chart di atas viewport aktif refresh data.
- Akar masalah: refresh chart menjalankan parsing sinkron + update DOM berat bersamaan dengan scroll handler.
- Langkah debug:
  - Rekam `Performance` saat scroll cepat 5-10 detik.
  - Cari `Long Task` dan kaitkan dengan call stack JS dominan.
  - Pisahkan kerja non-kritis ke jadwal terpisah, kurangi kerja per event scroll.
  - Ulangi trace pada skenario identik.
- Solusi: throttle update, pindahkan parsing non-kritis keluar jalur scroll, dan batasi DOM update ke boundary frame.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa mengidentifikasi contention utama dari trace (bukan asumsi).
- [ ] Bisa merancang ulang scroll handler agar kerja per frame terkontrol.
- [ ] Bisa menunjukkan before/after metric pada skenario scroll yang sama.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: intermediate 07 dan beginner 06 (audit trace).
- Topik terkait: animation pipeline CSS vs JS (modul 09).
- Referensi tambahan: dokumentasi Long Task API dan Performance panel.

## A) Pipeline Breakdown
- Parse: bukan jalur utama masalah pada kasus ini.
- Style: bisa terdampak jika scroll memicu class toggle luas.
- Layout: bisa melonjak jika handler melakukan read geometri massal.
- Paint: meningkat saat banyak visual update terjadi selama scroll.
- Composite: terlambat commit jika main thread terlalu padat oleh script/render task.

## B) Perf Metric yang Relevan
- Metric: jumlah `Long Task` saat skenario scroll.
- Target: 0 long task pada jalur scroll prioritas.
- Dampak jika buruk: input terasa delay dan frame drop konsisten.
- Metric: p95 frame time saat scroll kontinu.
- Target: p95 <= 16.7 ms pada perangkat target.
- Dampak jika buruk: scroll tidak halus dan pengalaman terasa berat.

## C) Verifikasi via DevTools
1. Panel yang dipakai:
   - `Performance`
   - `Performance Insights` (opsional)
2. Langkah observasi:
   - Rekam sesi scroll kontinu (5-10 detik) sebelum optimasi.
   - Catat long task, total script time, dan spike layout/paint saat scroll.
   - Terapkan pembagian kerja (throttle/rAF/scheduling) lalu rekam ulang.
3. Indikator sukses:
   - Long task berkurang atau hilang pada jalur scroll.
   - Frame time lebih stabil dan scroll terasa halus.

