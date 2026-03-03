# Local Storage dan State Persistence Dasar

## Meta
- Level: `Beginner`
- Estimasi durasi: `30-45 menit`
- Status drill: `opsional ringan`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham state sederhana dan `render()`.
  - [ ] Paham event input/click.
- Kamus mini:
  - `[baru]` persistence: data tetap ada setelah page reload.
  - `[baru]` localStorage: penyimpanan key-value di browser.
  - `[baru]` serialize: ubah object jadi string (JSON).

## 1) Pengantar Singkat Topik
Topik ini membahas cara menyimpan state ke `localStorage` agar data tetap ada setelah browser direfresh.

## 2) Big Picture
Tanpa persistence, user harus mengulang input setiap kali reload halaman. Dengan `localStorage`, state penting bisa dipulihkan otomatis saat aplikasi dibuka lagi. Ini meningkatkan pengalaman user untuk aplikasi kecil seperti todo, preference, dan draft form.

## 3) Small Picture
- Simpan state ke `localStorage` saat state berubah.
- Ambil state dari `localStorage` saat aplikasi mulai.
- Render UI berdasarkan state yang dipulihkan.

## 4) Wireframe Alur Konsep
```text
[update state] -> [save ke localStorage] -> [reload page] -> [load state] -> [render]
```
- Alur utama: data tersimpan dan kembali muncul setelah refresh.
- Alur jalan: tidak ada data tersimpan -> pakai default state.
- Alur error: data rusak/invalid JSON -> fallback ke state default.

## 5) Analogi Dunia Nyata
Seperti menaruh catatan belanja di laci rumah. Saat pulang lagi, catatan masih ada dan bisa dipakai lanjut.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: todo sederhana, theme preference, draft input.
- Alasan pakai: implementasi cepat tanpa backend.
- Kapan tidak dipakai: data sensitif atau data besar kompleks.

## 7) Contoh Sederhana + Bedah Output
```html
<input id="todoInput" placeholder="Tambah todo" />
<button id="addBtn">Tambah</button>
<button id="clearBtn">Clear</button>
<ul id="todoList"></ul>

<script>
  const STORAGE_KEY = 'todo-items';
  const todoInput = document.querySelector('#todoInput');
  const addBtn = document.querySelector('#addBtn');
  const clearBtn = document.querySelector('#clearBtn');
  const todoList = document.querySelector('#todoList');

  const state = { items: [] };

  function saveState() {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state.items));
  }

  function loadState() {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return;
    try {
      const parsed = JSON.parse(raw);
      if (Array.isArray(parsed)) state.items = parsed;
    } catch (err) {
      state.items = [];
    }
  }

  function render() {
    todoList.innerHTML = '';
    for (const item of state.items) {
      const li = document.createElement('li');
      li.textContent = item;
      todoList.appendChild(li);
    }
  }

  addBtn.addEventListener('click', () => {
    const value = todoInput.value.trim();
    if (!value) return;
    state.items.push(value);
    todoInput.value = '';
    saveState();
    render();
  });

  clearBtn.addEventListener('click', () => {
    state.items = [];
    saveState();
    render();
  });

  loadState();
  render();
</script>
```
Bedah output:
1. Tambah item, daftar langsung tampil di UI.
2. Refresh halaman, item tetap ada.
3. Klik `Clear`, daftar kosong dan tetap kosong setelah refresh.

## 8) Jebakan Umum (Pitfalls)
- Lupa `JSON.stringify` saat menyimpan object/array.
- Lupa `JSON.parse` saat membaca data.
- Tidak menangani data rusak di storage.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill (opsional):
```js
const items = ['A', 'B'];
localStorage.setItem('x', JSON.stringify(items));
console.log(JSON.parse(localStorage.getItem('x')).length);
```
Kunci:
- Output: `2`
- Alasan: array disimpan sebagai JSON string lalu diparse kembali.

## 10) Debug Story
- Gejala: data hilang setiap reload.
- Akar masalah: `saveState()` tidak dipanggil setelah state update.
- Langkah debug: cek tab Application -> Local Storage dan log saat save.
- Solusi: jadikan aturan "setiap mutasi state penting -> saveState()".

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menyimpan state sederhana ke `localStorage`.
- [ ] Bisa memulihkan state saat app dimulai.
- [ ] Bisa menangani fallback jika data storage tidak valid.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: state dan render dasar.
- Topik terkait: mini project dashboard browser.
- Referensi tambahan: MDN Web Storage API.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools (tab Application/Storage).
- File awal: `index.html` dengan input, tombol, dan list.

## B) Langkah Praktik
1. Definisikan state list sederhana.
2. Buat `saveState()` dan `loadState()` memakai `localStorage`.
3. Hubungkan event tambah/clear ke update state + save + render.

## C) Expected Output
- UI yang diharapkan: daftar todo persist setelah reload.
- Output console/network yang diharapkan: tidak ada error parse/runtime.

## D) Troubleshooting
- Masalah umum 1 + solusi: `Unexpected token` saat parse, data storage rusak; clear key lalu reload.
- Masalah umum 2 + solusi: data tidak tersimpan, pastikan key dan `saveState()` dipanggil setelah update.
