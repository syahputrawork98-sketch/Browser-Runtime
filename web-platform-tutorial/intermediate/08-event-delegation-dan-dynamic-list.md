# Event Delegation dan Dynamic List

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Status drill: `wajib`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham render list dari array state.
  - [ ] Paham event `click` dasar.
- Kamus mini:
  - `[baru]` event delegation: menangani event anak lewat parent.
  - `[baru]` dynamic list: daftar item yang bisa bertambah/berkurang saat runtime.
  - `[baru]` event target: elemen yang memicu event.

## 1) Pengantar Singkat Topik
Topik ini membahas cara menangani event untuk list dinamis tanpa memasang listener satu per satu ke setiap item.

## 2) Big Picture
Saat item list bertambah terus, memasang listener per item membuat kode boros dan rawan bug. Event delegation memungkinkan satu listener di parent menangani banyak item, termasuk item baru hasil render. Ini penting untuk performa dasar dan maintainability UI dinamis.

## 3) Small Picture
- Pasang satu listener pada elemen parent list.
- Gunakan `event.target` untuk cek elemen yang diklik.
- Lakukan aksi berdasarkan atribut item (`data-id`, `data-action`).

## 4) Wireframe Alur Konsep
```text
[click item child] -> [listener parent] -> [cek target] -> [update state] -> [render ulang]
```
- Alur utama: klik tombol delete item -> parent handler tangkap -> item terhapus.
- Alur jalan: item baru tetap otomatis punya perilaku click tanpa listener baru.
- Alur error: target tidak difilter sehingga klik area lain ikut memicu aksi.

## 5) Analogi Dunia Nyata
Seperti satu petugas gerbang memeriksa semua orang yang masuk, bukan menaruh satu petugas di tiap jalur kecil.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: todo list, table interaktif, menu dinamis.
- Alasan pakai: lebih efisien dari sisi listener dan lebih mudah dirawat.
- Kapan tidak dipakai: elemen statis kecil dengan interaksi minim.

## 7) Contoh Sederhana + Bedah Output
```html
<form id="todoForm">
  <input id="todoInput" placeholder="Tambah task" />
  <button type="submit">Add</button>
</form>
<ul id="todoList"></ul>

<script>
  const state = {
    items: [
      { id: 1, text: 'Belajar delegation' },
      { id: 2, text: 'Render list dinamis' }
    ]
  };

  const todoForm = document.querySelector('#todoForm');
  const todoInput = document.querySelector('#todoInput');
  const todoList = document.querySelector('#todoList');

  function render() {
    todoList.innerHTML = state.items
      .map(
        (item) => `
          <li>
            <span>${item.text}</span>
            <button data-action="delete" data-id="${item.id}">Delete</button>
          </li>
        `
      )
      .join('');
  }

  todoForm.addEventListener('submit', (event) => {
    event.preventDefault();
    const text = todoInput.value.trim();
    if (!text) return;
    state.items.push({ id: Date.now(), text });
    todoInput.value = '';
    render();
  });

  todoList.addEventListener('click', (event) => {
    const btn = event.target.closest('button[data-action="delete"]');
    if (!btn) return;

    const id = Number(btn.dataset.id);
    state.items = state.items.filter((item) => item.id !== id);
    render();
  });

  render();
</script>
```
Bedah output:
1. Item awal tampil dengan tombol delete.
2. Item baru hasil submit otomatis bisa di-delete tanpa listener tambahan.
3. Klik delete memicu update state lalu render ulang list.

## 8) Jebakan Umum (Pitfalls)
- Pasang listener di setiap item saat render (duplikasi listener).
- Lupa memfilter target klik (`if (!btn) return`).
- Mengandalkan index list untuk ID sehingga salah hapus saat urutan berubah.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const el = document.createElement('button');
el.dataset.id = '10';
console.log(Number(el.dataset.id) + 1);
```
Kunci:
- Output: `11`
- Alasan: `dataset` bertipe string, dikonversi ke number sebelum operasi.

## 10) Debug Story
- Gejala: item baru tidak merespons tombol delete.
- Akar masalah: listener dipasang di item lama saja, bukan di parent list.
- Langkah debug: cek jumlah listener aktif dan log `event.target`.
- Solusi: pakai satu listener parent + seleksi target dengan `closest`.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menerapkan event delegation pada list dinamis.
- [ ] Bisa menambah/hapus item tanpa menambah listener per item.
- [ ] Bisa menjelaskan kenapa delegation lebih scalable.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: komponen tanpa framework.
- Topik terkait: `09-url-state-dan-query-params.md`.
- Referensi tambahan: MDN Event bubbling dan `Element.closest()`.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: `index.html` berisi form input dan `<ul>` list.

## B) Langkah Praktik
1. Buat state list dan fungsi render item.
2. Tambahkan submit form untuk add item.
3. Pasang satu listener `click` di parent list.
4. Gunakan `data-id` + `data-action` untuk routing aksi.

## C) Expected Output
- UI yang diharapkan: item baru langsung muncul dan bisa dihapus.
- Output console/network yang diharapkan: tidak ada error saat klik area non-button.

## D) Troubleshooting
- Masalah umum 1 + solusi: delete tidak jalan, cek `closest('button[data-action=\"delete\"]')`.
- Masalah umum 2 + solusi: item salah terhapus, pastikan `data-id` unik dan compare sebagai number.
