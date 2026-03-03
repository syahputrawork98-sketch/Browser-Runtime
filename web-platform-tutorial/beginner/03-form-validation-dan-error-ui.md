# Form Validation dan Error UI Dasar

## Meta
- Level: `Beginner`
- Estimasi durasi: `30-45 menit`
- Status drill: `opsional ringan`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event `submit`.
  - [ ] Paham update teks di DOM.
- Kamus mini:
  - `[baru]` validation: proses cek input sebelum diproses.
  - `[baru]` error state: kondisi UI saat input tidak valid.

## 1) Pengantar Singkat Topik
Topik ini membahas cara memvalidasi input form dan menampilkan pesan error yang jelas di UI.

## 2) Big Picture
Form tanpa validasi membuat data kacau dan user bingung saat submit gagal. Dengan validasi sederhana dan error UI yang jelas, user tahu apa yang salah dan bagaimana memperbaikinya. Ini fondasi penting sebelum form menjadi lebih kompleks.

## 3) Small Picture
- Tangkap event submit.
- Cek aturan validasi input.
- Tampilkan error jika gagal, proses data jika valid.

## 4) Wireframe Alur Konsep
```text
[submit form] -> [validasi input] -> [error UI atau sukses]
```
- Alur utama: input valid -> submit sukses -> tampilkan status sukses.
- Alur jalan: input kosong/salah -> tampilkan pesan error.
- Alur error: error lama tidak di-reset sehingga user bingung.

## 5) Analogi Dunia Nyata
Seperti petugas loket memeriksa formulir: jika kolom wajib kosong, petugas langsung beri tahu bagian yang harus diperbaiki.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: login form, contact form, registrasi sederhana.
- Alasan pakai: mencegah data invalid masuk dan meningkatkan UX.
- Kapan tidak dipakai: tidak ada input user (halaman statis).

## 7) Contoh Sederhana + Bedah Output
```html
<form id="signupForm">
  <input id="nameInput" placeholder="Nama" />
  <input id="emailInput" placeholder="Email" />
  <button type="submit">Submit</button>
</form>
<p id="errorText"></p>
<p id="successText"></p>

<script>
  const form = document.querySelector('#signupForm');
  const nameInput = document.querySelector('#nameInput');
  const emailInput = document.querySelector('#emailInput');
  const errorText = document.querySelector('#errorText');
  const successText = document.querySelector('#successText');

  form.addEventListener('submit', (event) => {
    event.preventDefault();
    errorText.textContent = '';
    successText.textContent = '';

    const name = nameInput.value.trim();
    const email = emailInput.value.trim();

    if (!name) {
      errorText.textContent = 'Nama wajib diisi.';
      return;
    }

    if (!email.includes('@')) {
      errorText.textContent = 'Format email tidak valid.';
      return;
    }

    successText.textContent = `Data valid untuk ${name}.`;
  });
</script>
```
Bedah output:
1. Submit tanpa nama menampilkan `Nama wajib diisi.`.
2. Submit dengan email tanpa `@` menampilkan `Format email tidak valid.`.
3. Input valid menampilkan pesan sukses.

## 8) Jebakan Umum (Pitfalls)
- Lupa `event.preventDefault()` sehingga halaman reload.
- Error lama tidak dihapus sebelum validasi ulang.
- Pesan error tidak spesifik sehingga user tidak tahu yang salah.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill (opsional):
```js
const email = 'userexample.com';
console.log(email.includes('@') ? 'valid' : 'invalid');
```
Kunci:
- Output: `invalid`
- Alasan: string tidak mengandung karakter `@`.

## 10) Debug Story
- Gejala: form selalu gagal walau input terlihat benar.
- Akar masalah: nilai input punya spasi awal/akhir, tapi tidak di-`trim()`.
- Langkah debug: log nilai mentah input dengan tanda pembatas.
- Solusi: gunakan `trim()` sebelum validasi.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membuat validasi dasar untuk field wajib.
- [ ] Bisa menampilkan error UI yang jelas.
- [ ] Bisa membedakan jalur submit valid vs tidak valid.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: DOM event submit.
- Topik terkait: state dan render dasar.
- Referensi tambahan: MDN form validation basics.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: `index.html` berisi form 2 input + area pesan.

## B) Langkah Praktik
1. Buat form dengan input `nama` dan `email`.
2. Pasang event `submit`, lalu hentikan reload default.
3. Tambahkan validasi bertahap dan render pesan error/sukses.

## C) Expected Output
- UI yang diharapkan: pesan error tampil saat input invalid, pesan sukses saat valid.
- Output console/network yang diharapkan: tidak ada error JS saat submit.

## D) Troubleshooting
- Masalah umum 1 + solusi: halaman reload saat submit, pastikan `preventDefault()` dipanggil.
- Masalah umum 2 + solusi: error tidak berubah, pastikan pesan di-reset sebelum validasi baru.
