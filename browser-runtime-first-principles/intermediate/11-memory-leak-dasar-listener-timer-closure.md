# Memory Leak Dasar: Listener, Timer, dan Closure

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event listener, timer, dan closure dasar.
  - [ ] Paham lifecycle komponen/view sederhana pada UI browser.
- Kamus mini:
  - `[baru]` memory leak: objek tidak terpakai lagi tetapi tetap tertahan di memori.
  - `[baru]` retained reference: referensi yang membuat GC tidak bisa membersihkan objek.
  - `[ulang]` closure: function yang menyimpan akses ke scope luar.
  - `[baru]` cleanup: proses melepas listener/timer/referensi saat tidak dibutuhkan.

## 1) Pengantar Singkat Topik
Topik ini membahas kebocoran memori yang paling sering terjadi pada frontend: listener lupa dicabut, timer tidak dihentikan, dan closure menahan objek besar terlalu lama.

## 2) Big Picture
Leak memori biasanya tidak langsung terlihat saat fitur baru dibuat, tapi muncul setelah aplikasi dipakai lama: performa turun, UI patah, tab browser berat. Penyebab umumnya adalah side effect runtime yang tidak dibersihkan. Dengan discipline cleanup dan ownership referensi, kita bisa menjaga footprint memori stabil.

## 3) Small Picture
- Setiap `addEventListener` harus punya strategi `removeEventListener`.
- Setiap `setInterval`/timer berulang harus punya `clearInterval`/`clearTimeout`.
- Hindari closure menyimpan objek besar jika tidak perlu.
- Ikuti pola mount/unmount: setup -> use -> cleanup.

## 4) Wireframe Alur Konsep
```text
[mount view]
  -> [register listener + start timer]
  -> [view aktif]
  -> [unmount view]
  -> [cleanup listener + timer + refs]
```
- Alur utama: resource dibuat lalu dibersihkan tepat waktu.
- Alur jalan: lifecycle berulang tetap stabil karena cleanup konsisten.
- Alur error: setup berulang tanpa cleanup membuat leak menumpuk.

## 5) Analogi Dunia Nyata
Seperti menyewa ruang kerja harian: kalau setiap hari ambil kunci baru tapi tidak pernah mengembalikan, lama-lama gudang kunci penuh dan sistem kacau.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: komponen interaktif, polling berkala, dashboard yang hidup lama.
- Alasan pakai: menjaga performa jangka panjang dan mencegah degradasi memory.
- Kapan tidak dipakai: tidak ada; cleanup wajib pada resource yang bersifat persistent.

## 7) Contoh Sederhana + Bedah Output
```js
function mountWidget(root) {
  const state = { count: 0 };

  function onClick() {
    state.count += 1;
    console.log('clicked', state.count);
  }

  root.addEventListener('click', onClick);
  const intervalId = setInterval(() => {
    console.log('tick');
  }, 1000);

  // return cleanup
  return function unmountWidget() {
    root.removeEventListener('click', onClick);
    clearInterval(intervalId);
  };
}
```
Bedah output:
1. Saat mount, listener dan interval aktif.
2. Saat unmount dipanggil, keduanya dibersihkan.
3. Tanpa cleanup, click handler dan interval tetap hidup walau widget sudah tidak dipakai.

## 8) Jebakan Umum (Pitfalls)
- Menambah listener di tiap render tanpa melepas listener lama.
- Menyimpan data besar di closure callback interval.
- Menganggap browser “otomatis membersihkan semua” walau referensi masih ada.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
let id = setInterval(() => console.log('run'), 1000);
clearInterval(id);
console.log('stopped');
```
Kunci:
- Output: `stopped`
- Alasan: interval dihentikan sebelum sempat mencetak `run`.

Drill 2:
```js
const obj = { big: new Array(1000).fill('x') };
function make() {
  return () => obj.big.length;
}
const fn = make();
console.log(typeof fn);
```
Kunci:
- Output: `function`
- Alasan: closure tetap menyimpan akses ke `obj` sampai referensi `fn` dilepas.

Drill 3:
```js
const btn = document.createElement('button');
function h() {}
btn.addEventListener('click', h);
btn.removeEventListener('click', h);
console.log('clean');
```
Kunci:
- Output: `clean`
- Alasan: handler yang sama dicabut dengan benar, tidak tertinggal sebagai listener aktif.

## 10) Debug Story
- Gejala: setelah buka-tutup modal 20 kali, app makin lambat.
- Akar masalah: setiap buka modal menambah listener `keydown` dan interval tanpa cleanup saat modal ditutup.
- Langkah debug: profiling memory snapshot + cek jumlah listener aktif setelah unmount.
- Solusi: implement cleanup wajib pada close/unmount dan pastikan handler reference stabil.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa mengidentifikasi tiga sumber leak: listener, timer, closure.
- [ ] Bisa menulis pola setup/cleanup yang konsisten.
- [ ] Bisa menjelaskan kenapa referensi aktif menghalangi garbage collection.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: event listener dasar dan timer dasar.
- Topik terkait: `intermediate/12-mini-project-async-workflow-inspector.md`.
- Referensi tambahan: memory profiling tools di browser DevTools.

## 13) Model Mental
- Entitas utama: resource runtime (listener/timer), owner lifecycle, referensi aktif.
- Urutan proses: create resource -> resource hidup mengikuti owner -> owner selesai -> cleanup.
- Invariant (aturan yang selalu benar): selama referensi masih reachable, GC tidak akan membebaskan objek.

## 14) Failure Case
- Skenario gagal: komponen di-remount berkali-kali, tiap mount menambah interval baru tanpa clear.
- Akar penyebab runtime: interval lama tetap jalan karena id tidak pernah dibersihkan.
- Perbaikan: simpan id resource per-instance dan lakukan cleanup eksplisit saat unmount.

## 15) Spec Hint
- Section spec terkait (ringkas): perilaku event listener registry, timer APIs, dan reachability untuk garbage collection.
- Kata kunci pencarian spec: `EventTarget listeners`, `setInterval`, `clearInterval`, `garbage collection reachability`.
