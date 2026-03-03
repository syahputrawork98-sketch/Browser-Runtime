# Changelog

Semua perubahan penting pada repo ini akan dicatat di file ini.

Format mengikuti prinsip [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) dan Semantic Versioning.

## Aturan Versi
- Gunakan format versi: `MAJOR.MINOR.PATCH` (contoh: `1.2.0`).
- Setiap rilis wajib memakai tanggal format `YYYY-MM-DD`.
- Perubahan baru dicatat dulu di `Unreleased`, lalu dipindah saat rilis.

## Aturan Batas Baris
- `CHANGELOG.md` maksimal **200 baris**.
- Jika lebih dari 200 baris, pindahkan entri paling lama ke `CHANGELOG-ARCHIVE.md`.
- Pertahankan entri terbaru di `CHANGELOG.md` (termasuk `Unreleased`).
- Saat memindahkan, urutan kronologis wajib tetap konsisten.

## Prosedur Rotasi
1. Cek jumlah baris `CHANGELOG.md`.
2. Jika jumlah baris `<= 200`, tidak ada rotasi.
3. Jika jumlah baris `> 200`, pindahkan entri versi paling lama (bukan `Unreleased`) ke `CHANGELOG-ARCHIVE.md`.
4. Pastikan format section versi tetap sama di dua file.
5. Simpan `Unreleased` dan entri terbaru tetap di `CHANGELOG.md`.
6. Catat aksi rotasi di section `Unreleased` bagian `Changed`.

## [Unreleased]

### Added
- Tidak ada.

### Changed
- Tidak ada.

## [0.3.0] - 2026-03-04

### Added
- Tidak ada.

### Changed
- Menambahkan materi `web-platform-tutorial/advanced/14-cache-strategy-memory-localstorage.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/advanced/15-performance-basics-debounce-throttle-reflow.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/advanced/16-accessibility-basics-keyboard-aria.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/advanced/17-testing-basics-dom-and-api-flow.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/advanced/18-capstone-web-platform-app.md` dan mengubah statusnya menjadi `available` di index track README.

## [0.2.0] - 2026-03-04

### Added
- Menambahkan roadmap ekosistem Browser Runtime di `README.md`.
- Menambahkan struktur folder fase roadmap:
  - `web-platform-tutorial/`
  - `browser-runtime-first-principles/`
  - `browser-engine-and-rendering-book/`
  - `runtime-exercises-and-katas/`
  - `browser-case-studies/`
  - `web-standards-spec-companion/`
- Menambahkan `CHANGELOG.md` sebagai pusat log perubahan.
- Menambahkan `CHANGELOG-ARCHIVE.md` untuk rotasi entri lama.
- Menambahkan folder `templates/` untuk standar penulisan konten.
- Menambahkan template inti `templates/core-topic-template.md`.
- Menambahkan template track `templates/browser-runtime-first-principles-template.md`.
- Menambahkan template track `templates/browser-engine-and-rendering-book-template.md`.
- Menambahkan template track `templates/runtime-exercises-and-katas-template.md`.
- Menambahkan template track `templates/browser-case-studies-template.md`.
- Menambahkan template track `templates/web-standards-spec-companion-template.md`.
- Menambahkan `README.md` per track untuk panduan template lokal.
- Menambahkan topik awal `web-platform-tutorial/01-dom-events-fetch-mini-app.md`.
- Menambahkan topik awal `browser-runtime-first-principles/01-call-stack-task-microtask.md`.
- Menambahkan topik awal `browser-engine-and-rendering-book/01-render-pipeline-dasar.md`.
- Menambahkan topik awal `runtime-exercises-and-katas/01-event-loop-prediction-drills.md`.
- Menambahkan topik awal `browser-case-studies/01-ui-jank-layout-thrashing.md`.
- Menambahkan topik awal `web-standards-spec-companion/01-event-loop-spec-mapping.md`.
- Menambahkan log track `web-platform-tutorial/CHANGELOG.md`.
- Menambahkan arsip log track `web-platform-tutorial/CHANGELOG-ARCHIVE.md`.
- Menambahkan template lokal track:
  - `web-platform-tutorial/templates/core-topic-template.md`
  - `web-platform-tutorial/templates/web-platform-tutorial-template.md`
  - `web-platform-tutorial/templates/README.md`

### Changed
- Menambahkan kebijakan changelog di `README.md` agar aturan log konsisten.
- Menambahkan section `Format Konten` di `README.md` dan menautkan semua template.
- Memperbarui `README.md` tiap track dengan urutan belajar dan target output.
- Memperbarui `web-platform-tutorial/README.md` agar memakai template lokal dan sistem log track sendiri.
- Memperbarui referensi template Web Platform Tutorial di `README.md` root ke path lokal track.
- Menghapus `templates/web-platform-tutorial-template.md` dari root untuk menghindari duplikasi template track.
- Menambahkan pembagian level `Beginner -> Intermediate -> Advanced -> Expert` dan roadmap 18 modul di `web-platform-tutorial/README.md`.
- Memperbarui template lokal `web-platform-tutorial` (meta durasi/level, aturan drill beginner, dan extension wajib).
- Memperbarui `web-platform-tutorial/01-dom-events-fetch-mini-app.md` agar sinkron dengan format terbaru.
- Menambahkan folder aturan level khusus `web-platform-tutorial/levels/` dan memindahkan aturan level detail ke sana.
- Menambahkan struktur folder konten level pada `web-platform-tutorial/` (`beginner/`, `intermediate/`, `advanced/`, `expert-bridge/`) beserta file planned awal.
- Memindahkan `web-platform-tutorial/01-dom-events-fetch-mini-app.md` ke `web-platform-tutorial/beginner/01-dom-events-fetch-mini-app.md`.
- Menambahkan `web-platform-tutorial/expert-bridge/README.md` sebagai jalur lanjut ke track expert.
- Menyederhanakan `web-platform-tutorial/README.md` agar menekankan gambaran besar dan tujuan akhir track.
- Memusatkan aturan level `web-platform-tutorial` ke `README.md` dan menghapus file `web-platform-tutorial/levels/*.md`.
- Menghapus detail `Drill` dan `Durasi modul` dari `web-platform-tutorial/README.md` agar aturan level lebih ringkas.
- Menambahkan kembali aturan drill yang eksplisit per level dan index modul per level di `web-platform-tutorial/README.md`.
- Merapikan `web-platform-tutorial/CHANGELOG.md` section `Unreleased` agar tidak kontradiktif.
- Menambahkan materi `web-platform-tutorial/beginner/02-dom-state-render-dasar.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/beginner/03-form-validation-dan-error-ui.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/beginner/04-fetch-advanced-loading-retry-abort.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/beginner/05-local-storage-dan-state-persistence.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/beginner/06-mini-project-dashboard-browser.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/intermediate/07-dom-component-pattern-tanpa-framework.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/intermediate/08-event-delegation-dan-dynamic-list.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/intermediate/09-url-state-dan-query-params.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/intermediate/10-client-side-filter-sort-pagination.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/intermediate/11-form-multi-step-dan-validasi-lanjutan.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/intermediate/12-mini-project-search-filter-list.md` dan mengubah statusnya menjadi `available` di index track README.
- Menambahkan materi `web-platform-tutorial/advanced/13-network-resilience-timeout-retry-backoff.md` dan mengubah statusnya menjadi `available` di index track README.
