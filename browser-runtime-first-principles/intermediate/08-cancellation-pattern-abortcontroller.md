# Cancellation Pattern dengan AbortController

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham fetch, Promise, dan error handling dasar.
  - [ ] Paham race condition sederhana pada request beruntun.
- Kamus mini:
  - `[baru]` cancellation: penghentian proses async yang sudah tidak relevan.
  - `[baru]` `AbortController`: API browser untuk mengirim sinyal pembatalan.
  - `[baru]` `AbortSignal`: sinyal yang dipantau operasi async (misalnya fetch).
  - `[ulang]` stale response: respons lama yang datang belakangan dan menimpa state terbaru.

## 1) Pengantar Singkat Topik
Topik ini membahas cara membatalkan pekerjaan async agar hasil lama tidak merusak state terbaru. Fokus utama ada pada `AbortController` dan pola “cancel previous request”.

## 2) Big Picture
Pada aplikasi interaktif, user bisa memicu request berkali-kali dalam waktu singkat (search, filter, navigasi). Tanpa cancellation, respons lama bisa datang terlambat lalu overwrite data terbaru. Ini menimbulkan bug yang terlihat acak. Dengan pola cancellation yang konsisten, alur async jadi lebih deterministik dan UI lebih stabil.

## 3) Small Picture
- Buat `AbortController` per request aktif.
- Saat request baru dimulai, batalkan request sebelumnya.
- Oper ke `fetch` melalui `signal`.
- Bedakan error abort vs error network biasa.

## 4) Wireframe Alur Konsep
```text
[request A start]
  -> [user trigger request B]
  -> [abort A]
  -> [run B]
  -> [apply hanya hasil B]
```
- Alur utama: request lama dibatalkan sebelum request baru aktif.
- Alur jalan: request selesai normal jika tidak ada trigger baru.
- Alur error: respons lama tetap diproses karena tidak ada cancel guard.

## 5) Analogi Dunia Nyata
Seperti memesan taksi dua kali: saat pesanan kedua dibuat, pesanan pertama dibatalkan agar tidak ada mobil lama yang tiba lalu membuat rute kacau.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: live search, pergantian tab cepat, detail data berdasarkan pilihan user.
- Alasan pakai: mencegah stale response dan hemat resource.
- Kapan tidak dipakai: proses singkat tunggal yang tidak mungkin overlap.

## 7) Contoh Sederhana + Bedah Output
```js
let currentController = null;

async function loadUser(userId) {
  if (currentController) {
    currentController.abort();
  }

  currentController = new AbortController();
  const { signal } = currentController;

  try {
    const res = await fetch(
      `https://jsonplaceholder.typicode.com/users/${userId}`,
      { signal }
    );
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    console.log('apply user:', data.id);
  } catch (err) {
    if (err.name === 'AbortError') {
      console.log('request aborted');
      return;
    }
    console.log('request failed:', err.message);
  }
}

loadUser(1);
loadUser(2);
```
Bedah output:
1. Request pertama dimulai lalu dibatalkan saat request kedua dipicu.
2. Log `request aborted` muncul untuk request lama.
3. Hanya hasil request kedua yang di-apply.

## 8) Jebakan Umum (Pitfalls)
- Membatalkan request lama tapi tetap memproses hasil lama saat promise resolve.
- Tidak membedakan `AbortError` dari error network lain.
- Reuse `AbortController` yang sudah di-abort untuk request baru.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const c = new AbortController();
c.abort();
console.log(c.signal.aborted);
```
Kunci:
- Output: `true`
- Alasan: `abort()` langsung menandai signal sebagai aborted.

Drill 2:
```js
let ctrl = new AbortController();
ctrl.abort();
ctrl = new AbortController();
console.log(ctrl.signal.aborted);
```
Kunci:
- Output: `false`
- Alasan: controller baru punya signal baru yang belum dibatalkan.

Drill 3:
```js
let active = null;
function start() {
  if (active) active.abort();
  active = new AbortController();
  console.log('new request, aborted old if any');
}
start();
start();
```
Kunci:
- Output dua kali `new request...` dan request lama dibatalkan pada panggilan kedua.
- Alasan: pola selalu cancel previous sebelum membuat request baru.

## 10) Debug Story
- Gejala: hasil pencarian sering “mundur” ke keyword lama saat user mengetik cepat.
- Akar masalah: request lama tidak dibatalkan atau hasil lama tetap di-apply.
- Langkah debug: beri request id + log status (`start/abort/success`) dan cek urutan selesai.
- Solusi: terapkan `AbortController` + guard apply hanya untuk request aktif.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan kapan cancellation diperlukan.
- [ ] Bisa implementasi pola `cancel previous request`.
- [ ] Bisa membedakan handling `AbortError` vs error lain.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/02-promise-jobs-dan-urutan-eksekusi.md`.
- Topik terkait: `intermediate/09-race-condition-dan-last-write-wins.md`.
- Referensi tambahan: Fetch API dan AbortController basics.

## 13) Model Mental
- Entitas utama: request aktif, abort signal, hasil request.
- Urutan proses: start new -> abort old -> await new -> apply result jika valid.
- Invariant (aturan yang selalu benar): request yang dibatalkan tidak boleh mengubah state final.

## 14) Failure Case
- Skenario gagal: user klik item A lalu B cepat; data A datang belakangan dan menimpa tampilan B.
- Akar penyebab runtime: tidak ada cancellation atau validasi request aktif.
- Perbaikan: batalkan request lama saat request baru dimulai dan validasi identitas request sebelum apply.

## 15) Spec Hint
- Section spec terkait (ringkas): AbortController/AbortSignal integration pada Fetch.
- Kata kunci pencarian spec: `AbortController`, `AbortSignal`, `fetch signal`, `abort steps`.
