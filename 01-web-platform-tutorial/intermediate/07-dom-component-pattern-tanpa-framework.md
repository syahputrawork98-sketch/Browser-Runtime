# DOM Component Pattern Tanpa Framework

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Status drill: `wajib`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Selesai level Beginner (`01`-`06`).
  - [ ] Paham state, render, dan event handler.
- Kamus mini:
  - `[baru]` component: unit UI kecil dengan tanggung jawab jelas.
  - `[baru]` props: data input untuk komponen.
  - `[baru]` render function: fungsi pembentuk HTML dari data.

## 1) Pengantar Singkat Topik
Topik ini membahas cara memecah UI jadi komponen kecil tanpa framework agar kode lebih mudah dirawat.

## 2) Big Picture
Saat halaman makin besar, kode UI tunggal cepat berantakan karena semua logika bercampur. Dengan component pattern, setiap bagian UI punya boundary, input, dan output yang jelas. Ini menurunkan kompleksitas serta mempermudah refactor dan debugging.

## 3) Small Picture
- Definisikan fungsi komponen yang menerima data.
- Komponen mengembalikan string HTML/elemen DOM.
- Render root menyusun komponen menjadi halaman utuh.

## 4) Wireframe Alur Konsep
```text
[state global] -> [component render functions] -> [gabung HTML] -> [render root]
```
- Alur utama: state berubah -> render root -> setiap komponen update.
- Alur jalan: komponen dipakai ulang di lokasi berbeda.
- Alur error: komponen saling mengubah DOM langsung tanpa boundary.

## 5) Analogi Dunia Nyata
Seperti merakit furnitur modular: tiap bagian (kaki/meja/laci) dibuat terpisah lalu dirangkai jadi satu produk.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: dashboard sederhana, list item, card panel.
- Alasan pakai: kode lebih terstruktur sebelum pindah ke framework.
- Kapan tidak dipakai: script kecil sekali yang tidak berkembang.

## 7) Contoh Sederhana + Bedah Output
```html
<div id="app"></div>

<script>
  const state = {
    title: 'Task Board',
    tasks: ['Belajar DOM', 'Latihan fetch']
  };

  function Header(props) {
    return `<header><h2>${props.title}</h2></header>`;
  }

  function TaskList(props) {
    const items = props.tasks.map((task) => `<li>${task}</li>`).join('');
    return `<ul>${items}</ul>`;
  }

  function App(props) {
    return `
      <section>
        ${Header({ title: props.title })}
        ${TaskList({ tasks: props.tasks })}
      </section>
    `;
  }

  function render() {
    const app = document.querySelector('#app');
    app.innerHTML = App(state);
  }

  render();
</script>
```
Bedah output:
1. `Header` dan `TaskList` dirender terpisah.
2. `App` menyusun dua komponen jadi satu tampilan.
3. Perubahan state dapat dihubungkan ke `render()` ulang.

## 8) Jebakan Umum (Pitfalls)
- Komponen terlalu besar dan kembali jadi "god function".
- Komponen mengakses state global langsung tanpa props.
- Logika event bercampur dengan generator HTML secara acak.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
function Badge(props) {
  return `[${props.label}]`;
}
console.log(Badge({ label: 'NEW' }));
```
Kunci:
- Output: `[NEW]`
- Alasan: fungsi komponen menerima props lalu membentuk output deterministik.

## 10) Debug Story
- Gejala: sebagian panel tidak update setelah state berubah.
- Akar masalah: `render()` root dipanggil, tapi satu panel masih edit DOM manual di luar pola komponen.
- Langkah debug: telusuri semua lokasi `innerHTML` dan pastikan lewat jalur render root.
- Solusi: satukan update UI melalui fungsi komponen + render root.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa memecah UI menjadi minimal 2-3 komponen kecil.
- [ ] Bisa mengirim data lewat props tanpa akses state liar.
- [ ] Bisa menjaga satu jalur render root untuk update UI.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: mini project Beginner.
- Topik terkait: `08-event-delegation-dan-dynamic-list.md`.
- Referensi tambahan: pola render function UI vanilla JS.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: `index.html` dengan root element `#app`.

## B) Langkah Praktik
1. Buat 2 komponen fungsi sederhana (`Header`, `List`).
2. Susun keduanya dalam komponen root `App`.
3. Buat `render()` tunggal untuk menempelkan output ke root.
4. Uji dengan mengubah state dan render ulang.

## C) Expected Output
- UI yang diharapkan: tampilan terbagi komponen, bukan satu blok monolitik.
- Output console/network yang diharapkan: tidak ada error JavaScript saat render.

## D) Troubleshooting
- Masalah umum 1 + solusi: HTML tidak muncul, cek elemen root `#app` dan isi return komponen.
- Masalah umum 2 + solusi: komponen tampil `undefined`, pastikan props yang dibutuhkan dikirim saat dipanggil.

