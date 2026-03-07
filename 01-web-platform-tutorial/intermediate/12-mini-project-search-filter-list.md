# Mini Project: Search, Filter, dan List Manager

## Meta
- Level: `Intermediate`
- Estimasi durasi: `70-100 menit`
- Status drill: `wajib`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Selesai modul `07` sampai `11`.
  - [ ] Paham component pattern, delegation, URL state, filter-sort-page, dan multi-step validation.
- Kamus mini:
  - `[ulang]` query sync: sinkron state ke URL.
  - `[baru]` list manager: halaman pengelolaan data item dengan kontrol penuh.
  - `[baru]` view model: data turunan untuk kebutuhan render.

## 1) Pengantar Singkat Topik
Topik ini adalah capstone level Intermediate untuk menggabungkan pola komponen, event delegation, URL state, filter/sort/pagination, dan validasi form dalam satu proyek.

## 2) Big Picture
Di level Intermediate, tantangannya bukan lagi fitur tunggal, tetapi integrasi banyak fitur agar tetap rapi. Mini project ini memaksa kita menata arsitektur UI dengan state yang jelas, transform data konsisten, dan kontrol interaksi yang maintainable. Setelah selesai, kamu siap masuk level Advanced dengan fondasi engineering yang lebih kuat.

## 3) Small Picture
- Definisikan state global untuk data, filter, sort, page, dan form.
- Gunakan render root berbasis komponen fungsi.
- Gunakan delegation untuk aksi list item.
- Sinkronkan filter utama ke URL query.
- Validasi form add/edit sebelum update state.

## 4) Wireframe Alur Konsep
```text
[init app] -> [load state + parse URL] -> [render root]
  -> [user search/filter/sort/page] -> [update state + sync URL + render]
  -> [user add/edit/delete item] -> [validate + update state + render]
```
- Alur utama: semua aksi user masuk lewat update state terpusat lalu render ulang.
- Alur jalan: URL langsung membuka kondisi list yang sama.
- Alur error: transform/filter dilakukan di banyak tempat dan hasil jadi tidak sinkron.

## 5) Analogi Dunia Nyata
Seperti panel admin kecil: ada tabel item, ada filter/pencarian, ada form tambah/edit, dan semua aksi harus tetap konsisten.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: inventory mini, daftar kontak, katalog internal.
- Alasan pakai: latihan integrasi arsitektur frontend tanpa framework.
- Kapan tidak dipakai: hanya butuh demo satu fitur sederhana.

## 7) Contoh Sederhana + Bedah Output
```html
<section>
  <input id="searchInput" placeholder="Cari nama..." />
  <select id="sortSelect">
    <option value="name-asc">Nama A-Z</option>
    <option value="name-desc">Nama Z-A</option>
  </select>
</section>

<section>
  <form id="itemForm">
    <input id="nameInput" placeholder="Nama item" />
    <button type="submit">Tambah</button>
  </form>
  <p id="formError"></p>
</section>

<ul id="itemList"></ul>
<button id="prevBtn">Prev</button>
<span id="pageText"></span>
<button id="nextBtn">Next</button>

<script>
  const state = {
    items: [
      { id: 1, name: 'Alpha' },
      { id: 2, name: 'Beta' },
      { id: 3, name: 'Gamma' },
      { id: 4, name: 'Delta' }
    ],
    ui: { q: '', sort: 'name-asc', page: 1, pageSize: 2 }
  };

  const searchInput = document.querySelector('#searchInput');
  const sortSelect = document.querySelector('#sortSelect');
  const itemForm = document.querySelector('#itemForm');
  const nameInput = document.querySelector('#nameInput');
  const formError = document.querySelector('#formError');
  const itemList = document.querySelector('#itemList');
  const prevBtn = document.querySelector('#prevBtn');
  const nextBtn = document.querySelector('#nextBtn');
  const pageText = document.querySelector('#pageText');

  function syncUrl() {
    const params = new URLSearchParams(window.location.search);
    if (state.ui.q) params.set('q', state.ui.q);
    else params.delete('q');
    const next = `${window.location.pathname}?${params.toString()}`.replace(/\?$/, '');
    window.history.replaceState({}, '', next);
  }

  function loadFromUrl() {
    const params = new URLSearchParams(window.location.search);
    state.ui.q = (params.get('q') || '').trim();
  }

  function getViewData() {
    const filtered = state.items.filter((item) =>
      item.name.toLowerCase().includes(state.ui.q.toLowerCase())
    );

    const sorted = [...filtered].sort((a, b) => {
      if (state.ui.sort === 'name-asc') return a.name.localeCompare(b.name);
      return b.name.localeCompare(a.name);
    });

    const totalPages = Math.max(1, Math.ceil(sorted.length / state.ui.pageSize));
    if (state.ui.page > totalPages) state.ui.page = totalPages;

    const start = (state.ui.page - 1) * state.ui.pageSize;
    const pageItems = sorted.slice(start, start + state.ui.pageSize);

    return { pageItems, totalPages };
  }

  function render() {
    const view = getViewData();
    itemList.innerHTML = view.pageItems
      .map(
        (item) => `
          <li>
            <span>${item.name}</span>
            <button data-action="delete" data-id="${item.id}">Delete</button>
          </li>
        `
      )
      .join('');
    searchInput.value = state.ui.q;
    sortSelect.value = state.ui.sort;
    pageText.textContent = `Page ${state.ui.page}/${view.totalPages}`;
    prevBtn.disabled = state.ui.page <= 1;
    nextBtn.disabled = state.ui.page >= view.totalPages;
  }

  searchInput.addEventListener('input', (e) => {
    state.ui.q = e.target.value.trim();
    state.ui.page = 1;
    syncUrl();
    render();
  });

  sortSelect.addEventListener('change', (e) => {
    state.ui.sort = e.target.value;
    state.ui.page = 1;
    render();
  });

  itemForm.addEventListener('submit', (e) => {
    e.preventDefault();
    formError.textContent = '';
    const value = nameInput.value.trim();
    if (value.length < 2) {
      formError.textContent = 'Nama item minimal 2 karakter.';
      return;
    }
    state.items.push({ id: Date.now(), name: value });
    nameInput.value = '';
    render();
  });

  itemList.addEventListener('click', (e) => {
    const btn = e.target.closest('button[data-action="delete"]');
    if (!btn) return;
    const id = Number(btn.dataset.id);
    state.items = state.items.filter((item) => item.id !== id);
    render();
  });

  prevBtn.addEventListener('click', () => {
    if (state.ui.page > 1) state.ui.page -= 1;
    render();
  });

  nextBtn.addEventListener('click', () => {
    const view = getViewData();
    if (state.ui.page < view.totalPages) state.ui.page += 1;
    render();
  });

  loadFromUrl();
  render();
</script>
```
Bedah output:
1. Search tersinkron ke URL dan tetap aktif saat refresh.
2. Sort + pagination tetap konsisten terhadap data hasil filter.
3. Add item divalidasi, delete memakai event delegation.

## 8) Jebakan Umum (Pitfalls)
- Menaruh logic filter/sort/paging di banyak tempat berbeda.
- Lupa reset page saat query/sort berubah.
- Aksi delete langsung edit DOM tanpa update state.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const state = { q: 'al', items: ['Alpha', 'Beta'] };
const filtered = state.items.filter((x) => x.toLowerCase().includes(state.q));
console.log(filtered.length);
```
Kunci:
- Output: `1`
- Alasan: hanya `Alpha` yang mengandung `al`.

## 10) Debug Story
- Gejala: setelah delete item di page akhir, list kosong padahal data masih ada.
- Akar masalah: current page tidak di-clamp setelah jumlah data berkurang.
- Langkah debug: log `page`, `totalPages`, dan `filtered.length` setiap render.
- Solusi: lakukan clamp page di fungsi view data sebelum render.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menggabungkan minimal 5 pola intermediate dalam satu aplikasi.
- [ ] Bisa mempertahankan konsistensi state <-> URL <-> UI.
- [ ] Bisa menjelaskan arsitektur state dan alur render proyek.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: modul `07` sampai `11`.
- Topik terkait: mulai ke level Advanced (`13`).
- Referensi tambahan: MDN URLSearchParams, delegation, dan array transforms.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools (Console, Application, Performance basic).
- File awal: search controls, form input, list area, dan pagination controls.

## B) Langkah Praktik
1. Definisikan state global + view data pipeline.
2. Implementasi URL sync untuk query utama.
3. Implementasi add/delete dengan validasi + delegation.
4. Implementasi filter/sort/pagination dan pastikan konsisten.

## C) Expected Output
- UI yang diharapkan: list manager berjalan stabil di seluruh skenario utama.
- Output console/network yang diharapkan: tidak ada error JS pada interaksi normal.

## D) Troubleshooting
- Masalah umum 1 + solusi: query URL tidak sinkron, cek urutan `state -> syncUrl -> render`.
- Masalah umum 2 + solusi: data list tidak update benar, pastikan perubahan selalu lewat state lalu render ulang.
