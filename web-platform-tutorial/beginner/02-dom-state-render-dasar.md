# DOM State dan Render Dasar

## Meta
- Level: `Beginner`
- Estimasi durasi: `30-45 menit`
- Status drill: `opsional ringan`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham seleksi elemen DOM dasar.
  - [ ] Paham event `click`.
- Kamus mini:
  - `[baru]` state: data sumber kebenaran UI saat ini.
  - `[baru]` render: proses menampilkan state ke DOM.

## 1) Pengantar Singkat Topik
Topik ini membahas cara menyimpan data di state sederhana lalu merender ulang UI saat state berubah.

## 2) Big Picture
Banyak bug UI terjadi karena data dan tampilan tidak sinkron. Dengan pola state -> render, perubahan data selalu punya satu jalur update ke DOM. Pola ini jadi fondasi sebelum masuk ke fitur yang lebih kompleks.

## 3) Small Picture
- Simpan nilai di object state.
- Buat fungsi `render()` untuk mencerminkan state ke UI.
- Setiap event yang mengubah state harus memanggil `render()`.

## 4) Wireframe Alur Konsep
```text
[event user] -> [update state] -> [render] -> [UI terbaru]
```
- Alur utama: klik tombol -> state berubah -> UI ikut berubah.
- Alur jalan: banyak event tetap memakai jalur state -> render yang sama.
- Alur error: state berubah tapi lupa `render()`, UI jadi tidak sinkron.

## 5) Analogi Dunia Nyata
State seperti catatan stok di kasir, render seperti papan display harga. Kalau stok berubah tapi display tidak di-update, informasi jadi salah.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: counter, toggle, filter sederhana, daftar kecil.
- Alasan pakai: membuat alur update UI lebih konsisten dan mudah di-debug.
- Kapan tidak dipakai: script sekali jalan tanpa perubahan interaktif.

## 7) Contoh Sederhana + Bedah Output
```html
<h2 id="countText"></h2>
<button id="plusBtn">Tambah</button>
<button id="resetBtn">Reset</button>

<script>
  const state = { count: 0 };
  const countText = document.querySelector('#countText');
  const plusBtn = document.querySelector('#plusBtn');
  const resetBtn = document.querySelector('#resetBtn');

  function render() {
    countText.textContent = `Count: ${state.count}`;
  }

  plusBtn.addEventListener('click', () => {
    state.count += 1;
    render();
  });

  resetBtn.addEventListener('click', () => {
    state.count = 0;
    render();
  });

  render();
</script>
```
Bedah output:
1. Saat awal, UI menampilkan `Count: 0`.
2. Klik `Tambah` menaikkan state dan UI menjadi `Count: 1`, `Count: 2`, dst.
3. Klik `Reset` mengembalikan state dan UI ke `Count: 0`.

## 8) Jebakan Umum (Pitfalls)
- Mengubah state tapi lupa memanggil `render()`.
- Mengubah DOM manual di banyak tempat tanpa jalur render tunggal.
- Menyimpan state tersebar di banyak variabel global.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill (opsional):
```js
const state = { count: 1 };
function render() {
  console.log(`Count: ${state.count}`);
}
state.count += 2;
render();
```
Kunci:
- Output: `Count: 3`
- Alasan: state naik dari `1` ke `3`, lalu render mencetak nilai terbaru.

## 10) Debug Story
- Gejala: angka counter tidak berubah di layar.
- Akar masalah: event sudah update state, tapi fungsi render tidak dipanggil.
- Langkah debug: log nilai state setelah klik dan log saat `render()` dipanggil.
- Solusi: jadikan aturan wajib "setiap update state -> render".

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan peran state dan render.
- [ ] Bisa membuat UI counter sederhana berbasis state.
- [ ] Bisa menemukan bug saat state dan UI tidak sinkron.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: DOM event dasar.
- Topik terkait: form validation dan error UI.
- Referensi tambahan: MDN DOM manipulation.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: `index.html` dengan elemen teks dan tombol.

## B) Langkah Praktik
1. Buat object `state` dengan nilai awal.
2. Buat fungsi `render()` yang membaca state dan update DOM.
3. Pasang event handler tombol untuk update state, lalu panggil `render()`.

## C) Expected Output
- UI yang diharapkan: nilai counter berubah setiap interaksi.
- Output console/network yang diharapkan: tidak ada error JavaScript di console.

## D) Troubleshooting
- Masalah umum 1 + solusi: selector `null`, cek `id` di HTML dan urutan script.
- Masalah umum 2 + solusi: UI tidak update, pastikan `render()` dipanggil setelah state berubah.
