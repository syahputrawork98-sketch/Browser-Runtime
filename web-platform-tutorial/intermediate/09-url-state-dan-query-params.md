# URL State dan Query Params

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Status drill: `wajib`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham state + render list.
  - [ ] Paham event input/change.
- Kamus mini:
  - `[baru]` URL state: state yang disimpan di URL.
  - `[baru]` query params: pasangan key=value di URL (`?q=js&page=2`).
  - `[baru]` deep link: link yang langsung membuka kondisi UI tertentu.

## 1) Pengantar Singkat Topik
Topik ini membahas sinkronisasi state UI dengan query params agar halaman bisa dibagikan dan dipulihkan dari URL.

## 2) Big Picture
Tanpa URL state, filter/pencarian hilang saat refresh atau saat link dibagikan. Dengan query params, kondisi halaman jadi reproducible dan user bisa kembali ke state yang sama. Ini meningkatkan UX sekaligus memudahkan debugging karena state terlihat jelas di alamat browser.

## 3) Small Picture
- Baca query params saat app start -> isi state awal.
- Saat state filter berubah -> update URL.
- Render UI berdasarkan state yang sudah sinkron.

## 4) Wireframe Alur Konsep
```text
[URL query] -> [parse ke state] -> [render]
[user ubah filter] -> [update state] -> [update URL] -> [render]
```
- Alur utama: buka link dengan query -> halaman langsung tampil sesuai filter.
- Alur jalan: user ubah filter -> URL berubah tanpa reload.
- Alur error: URL tidak divalidasi -> state invalid dan UI rusak.

## 5) Analogi Dunia Nyata
Seperti menyimpan setelan televisi di remote: saat tombol preset ditekan, channel langsung balik ke konfigurasi yang sama.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: search page, filter katalog, tab aktif.
- Alasan pakai: state UI bisa dishare, bookmarkable, dan reproducible.
- Kapan tidak dipakai: state sangat sensitif/privat yang tidak boleh muncul di URL.

## 7) Contoh Sederhana + Bedah Output
```html
<input id="searchInput" placeholder="Cari task..." />
<ul id="taskList"></ul>

<script>
  const allTasks = ['Belajar JS', 'Belajar DOM', 'Belajar Fetch', 'Review Kode'];
  const state = { q: '' };

  const searchInput = document.querySelector('#searchInput');
  const taskList = document.querySelector('#taskList');

  function loadFromUrl() {
    const params = new URLSearchParams(window.location.search);
    state.q = (params.get('q') || '').trim();
  }

  function syncUrl() {
    const params = new URLSearchParams(window.location.search);
    if (state.q) params.set('q', state.q);
    else params.delete('q');
    const next = `${window.location.pathname}?${params.toString()}`.replace(/\?$/, '');
    window.history.replaceState({}, '', next);
  }

  function render() {
    const keyword = state.q.toLowerCase();
    const filtered = allTasks.filter((task) =>
      task.toLowerCase().includes(keyword)
    );
    taskList.innerHTML = filtered.map((task) => `<li>${task}</li>`).join('');
    searchInput.value = state.q;
  }

  searchInput.addEventListener('input', (event) => {
    state.q = event.target.value.trim();
    syncUrl();
    render();
  });

  loadFromUrl();
  render();
</script>
```
Bedah output:
1. Jika URL `?q=dom`, list langsung terfilter saat halaman dibuka.
2. Saat input berubah, query `q` di URL ikut berubah tanpa reload.
3. Refresh halaman tetap mempertahankan filter dari URL.

## 8) Jebakan Umum (Pitfalls)
- Update URL dengan `location.href` sehingga halaman reload total.
- Tidak decode/trim query sehingga filter tidak konsisten.
- Menyimpan terlalu banyak state di URL sampai sulit dibaca.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const params = new URLSearchParams('?q=js&page=2');
console.log(params.get('q'));
console.log(params.get('page'));
```
Kunci:
- Output:
  - `js`
  - `2`
- Alasan: `URLSearchParams.get` mengembalikan nilai string untuk key yang ada.

## 10) Debug Story
- Gejala: filter terlihat aktif, tapi saat refresh kembali default.
- Akar masalah: state berubah di memori, tapi URL tidak di-update.
- Langkah debug: bandingkan nilai state dengan `window.location.search`.
- Solusi: jadikan aturan setelah filter berubah -> `syncUrl()` -> `render()`.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membaca query params sebagai initial state.
- [ ] Bisa sinkronkan perubahan filter ke URL tanpa reload.
- [ ] Bisa menjaga render tetap konsisten setelah refresh.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: event delegation dan dynamic list.
- Topik terkait: `10-client-side-filter-sort-pagination.md`.
- Referensi tambahan: MDN URLSearchParams dan History API.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: input search + list statis sederhana.

## B) Langkah Praktik
1. Parse `window.location.search` ke state awal.
2. Buat fungsi `syncUrl()` dengan `history.replaceState`.
3. Hubungkan input search ke update state + URL + render.
4. Uji refresh dan copy-paste URL.

## C) Expected Output
- UI yang diharapkan: state filter persist lewat URL.
- Output console/network yang diharapkan: tidak ada reload saat filter berubah.

## D) Troubleshooting
- Masalah umum 1 + solusi: URL tidak berubah, cek penggunaan `replaceState` dan string query.
- Masalah umum 2 + solusi: filter salah saat load awal, pastikan `loadFromUrl()` dipanggil sebelum `render()`.
