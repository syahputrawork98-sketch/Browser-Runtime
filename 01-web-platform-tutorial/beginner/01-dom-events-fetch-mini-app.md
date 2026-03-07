# DOM Events + Fetch Mini App

## Meta
- Level: `Beginner`
- Estimasi durasi: `30-45 menit`
- Status drill: `opsional ringan`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham seleksi DOM dasar (`querySelector`).
  - [ ] Paham async dasar dengan `Promise`.
- Kamus mini:
  - `[baru]` event listener: fungsi yang dijalankan saat event terjadi.
  - `[baru]` fetch: API browser untuk HTTP request.

## 1) Pengantar Singkat Topik
Topik ini membangun mini app browser yang merespons klik user lalu mengambil data dari API.

## 2) Big Picture
Bug UI dasar sering muncul karena event tidak terpasang benar atau data async dipakai sebelum siap. Dengan memahami alur event dan fetch, kita bisa membuat interaksi yang stabil dan mudah didiagnosis. Setelah topik ini, kamu bisa membangun fitur browser kecil end-to-end.

## 3) Small Picture
- Ambil elemen DOM.
- Pasang listener klik.
- Jalankan `fetch`, tunggu respons, lalu render hasil.

## 4) Wireframe Alur Konsep
```text
[klik tombol] -> [fetch data] -> [render ke DOM]
```
- Alur utama: klik -> request sukses -> tampilkan data.
- Alur jalan: klik -> loading -> data tampil.
- Alur error: klik -> request gagal -> tampilkan pesan error.

## 5) Analogi Dunia Nyata
User seperti menekan tombol pesan antar, fetch seperti kurir mengambil pesanan, dan DOM update seperti pesanan tiba di meja.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: fitur daftar data, pencarian, detail item.
- Alasan pakai: pola paling umum di UI browser modern.
- Kapan tidak dipakai: saat data sudah tersedia lokal tanpa request.

## 7) Contoh Sederhana + Bedah Output
```html
<button id="loadBtn">Load User</button>
<pre id="out"></pre>
<script>
  const btn = document.querySelector('#loadBtn');
  const out = document.querySelector('#out');

  btn.addEventListener('click', async () => {
    out.textContent = 'Loading...';
    try {
      const res = await fetch('https://jsonplaceholder.typicode.com/users/1');
      const user = await res.json();
      out.textContent = `${user.id}: ${user.name}`;
    } catch (err) {
      out.textContent = `Error: ${err.message}`;
    }
  });
</script>
```
Bedah output:
1. Klik tombol mengubah teks jadi `Loading...`.
2. Saat respons berhasil, data user ditampilkan.
3. Jika gagal network, teks berubah jadi pesan error.

## 8) Jebakan Umum (Pitfalls)
- Lupa `await` saat parsing `res.json()`.
- Tidak menangani error network.
- Update DOM sebelum elemen tersedia.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
console.log('start');
fetch('https://jsonplaceholder.typicode.com/todos/1')
  .then(() => console.log('fetch done'));
console.log('end');
```
Kunci:
- Output: `start`, `end`, `fetch done`.
- Alasan: callback `.then` jalan async setelah stack sync selesai.

## 10) Debug Story
- Gejala: UI tetap `Loading...`.
- Akar masalah: promise reject tidak ditangani.
- Langkah debug: cek tab Network dan tambahkan `try/catch`.
- Solusi: tampilkan fallback error pada UI.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa pasang event listener dan update DOM.
- [ ] Bisa ambil data API dan render hasil.
- [ ] Bisa handling error request.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: promise dan async JavaScript dasar.
- Topik terkait: event loop dasar.
- Referensi tambahan: MDN DOM events dan Fetch API.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: `index.html` dengan satu tombol dan satu elemen output.

## B) Langkah Praktik
1. Buat elemen tombol dan area output.
2. Pasang event listener `click` pada tombol.
3. Jalankan `fetch` dan tampilkan hasil sukses/error ke DOM.

## C) Expected Output
- UI yang diharapkan: saat klik, muncul status loading lalu data user.
- Output console/network yang diharapkan: request `200 OK` atau error yang bisa dibaca.

## D) Troubleshooting
- Masalah umum 1 + solusi: CORS/error network, cek endpoint dan koneksi.
- Masalah umum 2 + solusi: output tidak berubah, pastikan selector elemen benar.
