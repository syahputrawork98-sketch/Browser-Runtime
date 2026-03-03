# Render Tick dan Timing Update UI

## Meta
- Level: `Beginner`
- Estimasi durasi: `35-50 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event loop dasar (sync, microtask, macrotask).
  - [ ] Paham update DOM sederhana (`textContent`, `classList`).
- Kamus mini:
  - `[baru]` render tick: siklus browser untuk menghitung dan menggambar ulang UI.
  - `[baru]` paint: proses menampilkan piksel hasil update ke layar.
  - `[baru]` `requestAnimationFrame` (rAF): callback sebelum browser melakukan paint frame berikutnya.
  - `[ulang]` reflow/layout: perhitungan ulang ukuran/posisi elemen.

## 1) Pengantar Singkat Topik
Topik ini menjelaskan kenapa perubahan DOM tidak selalu langsung terlihat “saat itu juga”. Fokusnya adalah kapan browser melakukan render dan bagaimana menempatkan logic agar sinkron dengan frame UI.

## 2) Big Picture
Banyak bug UI muncul dari asumsi bahwa setelah mengubah DOM, pengguna pasti sudah melihat hasilnya. Faktanya browser mengatur rendering per tick/frame. Jika update state, kerja berat JS, dan pembacaan layout dicampur tanpa urutan, UI terasa lag atau flicker. Dengan memahami timing render tick, kamu bisa membuat interaksi terasa mulus dan prediktif.

## 3) Small Picture
- Perubahan DOM dapat terjadi cepat, tapi visual tampil saat browser paint.
- Microtask yang panjang bisa menunda kesempatan render berikutnya.
- `requestAnimationFrame` cocok untuk kerja yang harus sinkron dengan frame.
- Hindari kerja berat tepat sebelum paint jika ingin UI cepat terlihat.

## 4) Wireframe Alur Konsep
```text
[event/callback]
  -> [update state + DOM]
  -> [microtask drain]
  -> [render tick: style/layout/paint]
  -> [frame baru terlihat]
```
- Alur utama: update DOM lalu browser paint pada tick berikutnya.
- Alur jalan: callback rAF dieksekusi tepat sebelum paint.
- Alur error: JS/microtask terlalu lama sehingga render tertunda.

## 5) Analogi Dunia Nyata
Seperti papan skor digital: operator bisa mengubah data cepat, tapi layar auditorium memperbarui tampilan pada siklus refresh tertentu.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: loading indicator, animasi sederhana, update list responsif.
- Alasan pakai: menjaga persepsi kecepatan UI dan menghindari flicker.
- Kapan tidak dipakai: logic backend murni tanpa dampak visual langsung.

## 7) Contoh Sederhana + Bedah Output
```html
<button id="runBtn">Run</button>
<p id="statusText">Idle</p>

<script>
  const runBtn = document.querySelector('#runBtn');
  const statusText = document.querySelector('#statusText');

  runBtn.addEventListener('click', () => {
    statusText.textContent = 'Loading...';
    console.log('DOM updated: Loading...');

    requestAnimationFrame(() => {
      console.log('rAF before next paint');
      statusText.textContent = 'Done';
    });
  });
</script>
```
Bedah output:
1. Klik tombol langsung mengubah DOM ke `Loading...`.
2. Callback `rAF` berjalan sebelum paint frame berikutnya.
3. Status berubah ke `Done` dalam siklus render berikutnya.

## 8) Jebakan Umum (Pitfalls)
- Mengukur ukuran/layout berulang setelah tiap write DOM kecil.
- Menaruh kerja JS berat panjang di satu frame sehingga UI terasa freeze.
- Mengira `setTimeout(..., 0)` setara dengan sinkronisasi frame render.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
console.log('A');
requestAnimationFrame(() => console.log('B'));
console.log('C');
```
Kunci:
- Output: `A`, `C`, lalu `B`
- Alasan: callback rAF tidak sinkron; ia dijadwalkan untuk frame berikutnya.

Drill 2:
```js
Promise.resolve().then(() => console.log('microtask'));
requestAnimationFrame(() => console.log('raf'));
console.log('sync');
```
Kunci:
- Output: `sync`, `microtask`, lalu `raf`
- Alasan: microtask di-drain setelah sync, rAF terjadi pada fase frame berikutnya.

Drill 3:
```js
status.textContent = 'Loading...';
for (let i = 0; i < 1e8; i += 1) {}
status.textContent = 'Done';
```
Kunci:
- Output visual kemungkinan langsung terlihat `Done` (tanpa sempat tampak `Loading...`)
- Alasan: loop berat menahan main thread sehingga paint tertunda sampai loop selesai.

## 10) Debug Story
- Gejala: spinner loading kadang tidak pernah terlihat saat proses berat dimulai.
- Akar masalah: thread utama langsung dipakai kerja berat setelah set spinner.
- Langkah debug: ukur durasi blok JS dan cek timeline di tab Performance.
- Solusi: pecah kerja berat, beri kesempatan browser paint dulu (mis. rAF atau chunking kerja).

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan kenapa update DOM tidak selalu langsung terlihat di layar.
- [ ] Bisa membedakan peran microtask vs rAF terhadap timing UI.
- [ ] Bisa mengidentifikasi kasus render tertunda karena blok JS.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/01` dan `beginner/03`.
- Topik terkait: `advanced/14-batching-dom-read-write-untuk-runtime-stability.md`.
- Referensi tambahan: konsep browser rendering pipeline dan `requestAnimationFrame`.

## 13) Model Mental
- Entitas utama: main thread JS, microtask queue, render tick, paint.
- Urutan proses: jalankan JS -> drain microtask -> browser proses frame (termasuk rAF) -> paint.
- Invariant (aturan yang selalu benar): paint butuh kesempatan frame; jika main thread sibuk, paint tertunda.

## 14) Failure Case
- Skenario gagal: developer set teks `Loading...` lalu langsung jalankan proses berat sinkron panjang.
- Akar penyebab runtime: main thread tidak memberi ruang browser untuk paint sebelum proses berat selesai.
- Perbaikan: pisah kerja berat per chunk atau jadwalkan tahap berikutnya setelah frame berjalan.

## 15) Spec Hint
- Section spec terkait (ringkas): event loop rendering opportunity dan animation frame callbacks.
- Kata kunci pencarian spec: `rendering opportunity`, `requestAnimationFrame`, `update the rendering`.
