# Race Condition dan Last Write Wins

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham Promise/fetch dan urutan microtask/macrotask.
  - [ ] Paham cancellation pattern dasar.
- Kamus mini:
  - `[baru]` race condition: hasil akhir bergantung urutan selesai yang tidak stabil.
  - `[baru]` last write wins (LWW): kebijakan bahwa update terakhir yang valid menjadi state final.
  - `[ulang]` stale response: respons lama yang datang terlambat.
  - `[baru]` request token: penanda identitas request untuk validasi hasil.

## 1) Pengantar Singkat Topik
Topik ini membahas cara mencegah data “mundur” akibat respons async datang tidak berurutan. Fokusnya adalah strategi `last write wins` untuk menjaga state konsisten.

## 2) Big Picture
Pada UI modern, user sering memicu request berturut-turut (search/filter/navigasi). Urutan selesai request tidak selalu sama dengan urutan start. Tanpa guard, respons lama bisa menimpa data baru. Dengan pola LWW (token/version check atau cancellation), kita memastikan hanya hasil terbaru yang boleh mengubah state.

## 3) Small Picture
- Setiap request diberi id versi/token yang meningkat.
- Simpan id request terbaru sebagai “otoritas state”.
- Saat respons datang, apply hanya jika id masih terbaru.
- Gabungkan dengan cancellation untuk efisiensi dan stabilitas.

## 4) Wireframe Alur Konsep
```text
[request #1 start]
[request #2 start]
   -> [#2 selesai dulu] -> [apply #2]
   -> [#1 selesai belakangan] -> [abaikan #1]
```
- Alur utama: hasil terbaru menang (LWW).
- Alur jalan: request lama tidak merusak state final.
- Alur error: tanpa guard, hasil lama overwrite hasil baru.

## 5) Analogi Dunia Nyata
Seperti papan informasi jadwal: jika ada revisi terbaru, versi lama harus diabaikan walaupun baru dikirim belakangan.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: live search, autocomplete, filter produk, detail panel dinamis.
- Alasan pakai: menjaga konsistensi state ketika async overlap.
- Kapan tidak dipakai: proses serial tunggal tanpa kemungkinan overlap.

## 7) Contoh Sederhana + Bedah Output
```js
let latestRequestId = 0;
let state = { items: [] };

async function search(keyword) {
  const requestId = ++latestRequestId;

  const res = await fetch(
    `https://jsonplaceholder.typicode.com/posts?title_like=${encodeURIComponent(keyword)}`
  );
  const data = await res.json();

  // Last write wins guard
  if (requestId !== latestRequestId) {
    console.log('ignore stale result:', requestId);
    return;
  }

  state.items = data;
  console.log('apply result for request:', requestId);
}

search('a');
search('ab');
```
Bedah output:
1. Dua request berjalan overlap.
2. Hanya request terbaru (`ab`) boleh apply ke state.
3. Respons lama dicatat sebagai stale dan diabaikan.

## 8) Jebakan Umum (Pitfalls)
- Mengandalkan urutan start request, bukan urutan selesai aktual.
- Tidak menyimpan `latestRequestId` secara terpusat.
- Mencampur kebijakan LWW dengan merge state acak tanpa aturan.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
let latest = 1;
const id = 1;
latest = 2;
console.log(id === latest ? 'apply' : 'ignore');
```
Kunci:
- Output: `ignore`
- Alasan: id respons sudah bukan request terbaru.

Drill 2:
```js
let latest = 0;
function makeReq() {
  latest += 1;
  return latest;
}
const r1 = makeReq();
const r2 = makeReq();
console.log(r1, r2, latest);
```
Kunci:
- Output: `1 2 2`
- Alasan: request kedua menjadi otoritas terbaru.

Drill 3:
```js
let latest = 3;
function shouldApply(id) {
  return id === latest;
}
console.log(shouldApply(2));
console.log(shouldApply(3));
```
Kunci:
- Output:
  - `false`
  - `true`
- Alasan: hanya id sama dengan `latest` yang valid untuk apply.

## 10) Debug Story
- Gejala: user mengetik cepat, hasil list sering kembali ke keyword lama.
- Akar masalah: setiap respons langsung di-apply tanpa verifikasi kebaruan.
- Langkah debug: log `requestId`, `latestRequestId`, dan waktu selesai tiap request.
- Solusi: terapkan guard LWW dan optional cancellation request lama.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan race condition pada alur request overlap.
- [ ] Bisa implementasi guard `last write wins` berbasis request id.
- [ ] Bisa membedakan kapan pakai LWW saja vs LWW + cancellation.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `intermediate/08-cancellation-pattern-abortcontroller.md`.
- Topik terkait: `intermediate/10-runtime-side-effects-dan-state-consistency.md`.
- Referensi tambahan: concurrency control sederhana di frontend state management.

## 13) Model Mental
- Entitas utama: request id, latest id, respons async.
- Urutan proses: start -> assign id -> selesai -> validasi id -> apply/ignore.
- Invariant (aturan yang selalu benar): respons dengan id lebih lama tidak boleh override state terbaru.

## 14) Failure Case
- Skenario gagal: request keyword `a` selesai setelah `ab`, lalu hasil `a` menimpa UI.
- Akar penyebab runtime: tidak ada validasi kebaruan sebelum update state.
- Perbaikan: gunakan request token/version guard (LWW), dan bila perlu abort request lama.

## 15) Spec Hint
- Section spec terkait (ringkas): event loop/async completion ordering tidak menjamin urutan selesai sesuai urutan start.
- Kata kunci pencarian spec: `fetch`, `promise resolution order`, `task completion`.
