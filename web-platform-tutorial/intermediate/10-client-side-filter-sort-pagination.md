# Client-Side Filter, Sort, dan Pagination

## Meta
- Level: `Intermediate`
- Estimasi durasi: `50-70 menit`
- Status drill: `wajib`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham state, render list, dan query params dasar.
  - [ ] Paham array method `filter`, `sort`, `slice`.
- Kamus mini:
  - `[baru]` filter: menyaring data sesuai kondisi.
  - `[baru]` sort: mengurutkan data.
  - `[baru]` pagination: membagi data per halaman.

## 1) Pengantar Singkat Topik
Topik ini membahas bagaimana menampilkan data list besar secara rapi dengan kombinasi filter, sort, dan pagination di sisi client.

## 2) Big Picture
List data yang banyak cepat jadi sulit dipakai jika semua item ditampilkan sekaligus. Dengan filter, sort, dan pagination, user bisa menemukan data lebih cepat dan UI tetap ringan. Pola ini adalah fondasi penting sebelum masuk ke table/grid yang lebih kompleks.

## 3) Small Picture
- Simpan state UI: keyword filter, sort key/order, current page.
- Turunkan data langkah demi langkah: `filter -> sort -> paginate`.
- Render hanya item pada halaman aktif.

## 4) Wireframe Alur Konsep
```text
[raw data] -> [filter] -> [sort] -> [paginate] -> [render page items]
```
- Alur utama: user ubah kontrol -> state update -> render hasil baru.
- Alur jalan: perubahan filter reset page ke 1 untuk mencegah halaman kosong.
- Alur error: urutan transform salah menyebabkan hasil tidak konsisten.

## 5) Analogi Dunia Nyata
Seperti cari buku di perpustakaan: pilih kategori (filter), urutkan alfabet (sort), lalu lihat rak per blok (pagination).

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: katalog produk, daftar user, log sederhana.
- Alasan pakai: UX lebih terkontrol saat data bertambah.
- Kapan tidak dipakai: data sangat sedikit dan tidak butuh navigasi.

## 7) Contoh Sederhana + Bedah Output
```html
<input id="searchInput" placeholder="Cari nama..." />
<select id="sortSelect">
  <option value="name-asc">Nama A-Z</option>
  <option value="name-desc">Nama Z-A</option>
</select>
<button id="prevBtn">Prev</button>
<span id="pageText"></span>
<button id="nextBtn">Next</button>
<ul id="list"></ul>

<script>
  const data = [
    { id: 1, name: 'Ari' }, { id: 2, name: 'Bima' }, { id: 3, name: 'Citra' },
    { id: 4, name: 'Dina' }, { id: 5, name: 'Eka' }, { id: 6, name: 'Fajar' }
  ];

  const state = { q: '', sort: 'name-asc', page: 1, pageSize: 2 };

  const searchInput = document.querySelector('#searchInput');
  const sortSelect = document.querySelector('#sortSelect');
  const prevBtn = document.querySelector('#prevBtn');
  const nextBtn = document.querySelector('#nextBtn');
  const pageText = document.querySelector('#pageText');
  const list = document.querySelector('#list');

  function getViewData() {
    const filtered = data.filter((item) =>
      item.name.toLowerCase().includes(state.q.toLowerCase())
    );

    const sorted = [...filtered].sort((a, b) => {
      if (state.sort === 'name-asc') return a.name.localeCompare(b.name);
      return b.name.localeCompare(a.name);
    });

    const totalPages = Math.max(1, Math.ceil(sorted.length / state.pageSize));
    if (state.page > totalPages) state.page = totalPages;

    const start = (state.page - 1) * state.pageSize;
    const items = sorted.slice(start, start + state.pageSize);

    return { items, totalPages, totalItems: sorted.length };
  }

  function render() {
    const view = getViewData();
    list.innerHTML = view.items.map((item) => `<li>${item.name}</li>`).join('');
    pageText.textContent = `Page ${state.page}/${view.totalPages}`;
    prevBtn.disabled = state.page <= 1;
    nextBtn.disabled = state.page >= view.totalPages;
  }

  searchInput.addEventListener('input', (e) => {
    state.q = e.target.value.trim();
    state.page = 1;
    render();
  });

  sortSelect.addEventListener('change', (e) => {
    state.sort = e.target.value;
    state.page = 1;
    render();
  });

  prevBtn.addEventListener('click', () => {
    if (state.page > 1) state.page -= 1;
    render();
  });

  nextBtn.addEventListener('click', () => {
    const view = getViewData();
    if (state.page < view.totalPages) state.page += 1;
    render();
  });

  render();
</script>
```
Bedah output:
1. Input search menyaring list.
2. Sort mengubah urutan hasil.
3. Pagination menampilkan item per halaman dan tombol prev/next mengikuti batas.

## 8) Jebakan Umum (Pitfalls)
- Mutasi array asli saat sort (`data.sort(...)`) padahal dibutuhkan data mentah.
- Tidak reset halaman saat filter berubah.
- Menghitung total page sebelum filter/sort selesai.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const arr = [5, 1, 3];
const sorted = [...arr].sort((a, b) => a - b);
console.log(arr.join(','));
console.log(sorted.join(','));
```
Kunci:
- Output:
  - `5,1,3`
  - `1,3,5`
- Alasan: spread membuat copy, jadi array asli tidak berubah.

## 10) Debug Story
- Gejala: setelah search, list kosong walau data ada.
- Akar masalah: current page tetap di page besar dari kondisi sebelum filter.
- Langkah debug: log `state.page`, `totalPages`, dan `filtered.length`.
- Solusi: reset `state.page = 1` setiap filter/sort berubah.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menerapkan alur `filter -> sort -> paginate` yang konsisten.
- [ ] Bisa menjaga state page valid saat data hasil filter menyusut.
- [ ] Bisa mencegah mutasi data mentah saat sort.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: URL state dan query params.
- Topik terkait: `11-form-multi-step-dan-validasi-lanjutan.md`.
- Referensi tambahan: MDN Array `filter`, `sort`, `slice`.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: input search, select sort, nav pagination, dan list element.

## B) Langkah Praktik
1. Definisikan state filter/sort/page.
2. Buat fungsi transform data berurutan.
3. Render hasil page aktif + kontrol prev/next.
4. Hubungkan event input/select/button ke update state dan render.

## C) Expected Output
- UI yang diharapkan: hasil list tetap konsisten saat kombinasi filter+sort+page berubah.
- Output console/network yang diharapkan: tidak ada error dan page tidak melebihi total page.

## D) Troubleshooting
- Masalah umum 1 + solusi: page jadi 0 atau > total, clamp nilai page setelah hitung total page.
- Masalah umum 2 + solusi: urutan aneh saat sort, cek comparator dan pastikan sort terhadap salinan array.
