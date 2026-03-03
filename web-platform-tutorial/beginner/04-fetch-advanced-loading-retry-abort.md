# Fetch Lanjutan: Loading, Retry, dan Abort

## Meta
- Level: `Beginner`
- Estimasi durasi: `35-50 menit`
- Status drill: `opsional ringan`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham fetch dasar dan `try/catch`.
  - [ ] Paham update UI lewat DOM.
- Kamus mini:
  - `[baru]` loading state: status UI saat request berjalan.
  - `[baru]` retry: mencoba request ulang saat gagal.
  - `[baru]` abort: membatalkan request yang tidak diperlukan lagi.

## 1) Pengantar Singkat Topik
Topik ini membahas pola fetch yang lebih tahan gangguan: menampilkan loading, menangani kegagalan dengan retry, dan membatalkan request lewat `AbortController`.

## 2) Big Picture
Request jaringan di dunia nyata tidak selalu mulus. Jika UI tidak mengelola loading, retry, dan abort, user bisa melihat data usang, proses menggantung, atau klik berulang yang bikin kacau. Pola ini membuat pengalaman user lebih stabil dan aplikasi lebih aman dari kondisi race sederhana.

## 3) Small Picture
- Saat request mulai: set loading `true`, update UI.
- Saat gagal: retry dengan batas percobaan.
- Saat user pindah konteks: abort request lama.

## 4) Wireframe Alur Konsep
```text
[klik load] -> [loading on] -> [fetch]
  -> [sukses -> render data]
  -> [gagal -> retry <= batas]
  -> [dibatalkan -> tampilkan status cancel]
```
- Alur utama: request sukses dalam 1-2 kali percobaan.
- Alur jalan: request pertama gagal, retry berhasil.
- Alur error: request lama tidak di-abort dan hasilnya menimpa request terbaru.

## 5) Analogi Dunia Nyata
Seperti memesan ojek: aplikasi menampilkan "mencari driver" (loading), mencoba cari lagi kalau gagal (retry), dan membatalkan pesanan lama saat kamu ganti tujuan (abort).

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: search data, load detail, refresh list.
- Alasan pakai: membuat fetch lebih robust untuk kondisi jaringan nyata.
- Kapan tidak dipakai: request internal yang pasti cepat/stabil di lingkungan demo sederhana.

## 7) Contoh Sederhana + Bedah Output
```html
<button id="loadBtn">Load Todo</button>
<button id="cancelBtn">Cancel</button>
<p id="statusText"></p>
<pre id="resultText"></pre>

<script>
  const loadBtn = document.querySelector('#loadBtn');
  const cancelBtn = document.querySelector('#cancelBtn');
  const statusText = document.querySelector('#statusText');
  const resultText = document.querySelector('#resultText');

  let controller = null;

  async function fetchWithRetry(url, retries, signal) {
    for (let i = 0; i <= retries; i++) {
      try {
        const res = await fetch(url, { signal });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return await res.json();
      } catch (err) {
        if (signal.aborted) throw err;
        if (i === retries) throw err;
      }
    }
  }

  loadBtn.addEventListener('click', async () => {
    if (controller) controller.abort();
    controller = new AbortController();

    statusText.textContent = 'Loading...';
    resultText.textContent = '';

    try {
      const data = await fetchWithRetry(
        'https://jsonplaceholder.typicode.com/todos/1',
        1,
        controller.signal
      );
      statusText.textContent = 'Success';
      resultText.textContent = JSON.stringify(data, null, 2);
    } catch (err) {
      if (controller.signal.aborted) {
        statusText.textContent = 'Request dibatalkan';
      } else {
        statusText.textContent = `Gagal: ${err.message}`;
      }
    }
  });

  cancelBtn.addEventListener('click', () => {
    if (controller) controller.abort();
  });
</script>
```
Bedah output:
1. Klik `Load Todo` menampilkan status `Loading...`.
2. Jika sukses, status jadi `Success` dan data tampil.
3. Jika dibatalkan, status jadi `Request dibatalkan`.
4. Jika gagal terus, status menampilkan pesan `Gagal: ...`.

## 8) Jebakan Umum (Pitfalls)
- Tidak reset loading state saat error/cancel.
- Retry tanpa batas sehingga request spam.
- Tidak membedakan error biasa dan abort.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill (opsional):
```js
const controller = new AbortController();
controller.abort();
console.log(controller.signal.aborted ? 'cancelled' : 'running');
```
Kunci:
- Output: `cancelled`
- Alasan: `abort()` mengubah `signal.aborted` menjadi `true`.

## 10) Debug Story
- Gejala: data tiba-tiba "mundur" setelah user klik load beberapa kali.
- Akar masalah: hasil request lama datang belakangan dan menimpa data request terbaru.
- Langkah debug: beri ID request di log, bandingkan urutan kirim vs selesai.
- Solusi: abort request lama sebelum kirim request baru.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menampilkan loading state yang jelas.
- [ ] Bisa menambahkan retry dengan batas percobaan.
- [ ] Bisa membatalkan request menggunakan `AbortController`.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: fetch mini app dasar.
- Topik terkait: form validation dan state-render.
- Referensi tambahan: MDN Fetch API dan AbortController.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: `index.html` dengan tombol load, cancel, status, dan area hasil.

## B) Langkah Praktik
1. Buat state UI dasar: loading/success/error/cancel.
2. Implementasi fungsi `fetchWithRetry` dengan batas retry.
3. Tambahkan `AbortController` dan tombol cancel.

## C) Expected Output
- UI yang diharapkan: loading jelas, hasil sukses tampil, cancel/error terbaca.
- Output console/network yang diharapkan: request ter-cancel terlihat sebagai aborted.

## D) Troubleshooting
- Masalah umum 1 + solusi: loading tidak hilang, pastikan semua jalur `try/catch` update status akhir.
- Masalah umum 2 + solusi: cancel tidak bekerja, pastikan `fetch` memakai `signal` dari controller.
