# Changelog - Web Platform Tutorial

Semua perubahan penting khusus track `web-platform-tutorial` dicatat di file ini.

Format mengikuti prinsip [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) dan Semantic Versioning.

## Aturan Versi
- Gunakan format versi: `MAJOR.MINOR.PATCH`.
- Rilis wajib memakai tanggal format `YYYY-MM-DD`.
- Perubahan baru dicatat dulu di `Unreleased`.

## Aturan Batas Baris
- `web-platform-tutorial/CHANGELOG.md` maksimal **200 baris**.
- Jika lebih dari 200 baris, pindahkan entri lama ke `CHANGELOG-ARCHIVE.md`.
- Pertahankan entri terbaru (termasuk `Unreleased`) di file utama.

## [Unreleased]

### Added
- Tidak ada.

### Changed
- Tidak ada.

## [0.3.0] - 2026-03-04

### Added
- Menambahkan materi lengkap `advanced/14-cache-strategy-memory-localstorage.md` (format core + extension A-D).
- Menambahkan materi lengkap `advanced/15-performance-basics-debounce-throttle-reflow.md` (format core + extension A-D).
- Menambahkan materi lengkap `advanced/16-accessibility-basics-keyboard-aria.md` (format core + extension A-D).
- Menambahkan materi lengkap `advanced/17-testing-basics-dom-and-api-flow.md` (format core + extension A-D).
- Menambahkan materi lengkap `advanced/18-capstone-web-platform-app.md` (format core + extension A-D).

### Changed
- Memperbarui status `advanced/14-cache-strategy-memory-localstorage.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/15-performance-basics-debounce-throttle-reflow.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/16-accessibility-basics-keyboard-aria.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/17-testing-basics-dom-and-api-flow.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/18-capstone-web-platform-app.md` di index `README.md` dari `planned` menjadi `available`.

## [0.2.0] - 2026-03-04

### Added
- Menambahkan materi lengkap `intermediate/07-dom-component-pattern-tanpa-framework.md` (format core + extension A-D).
- Menambahkan materi lengkap `intermediate/08-event-delegation-dan-dynamic-list.md` (format core + extension A-D).
- Menambahkan materi lengkap `intermediate/09-url-state-dan-query-params.md` (format core + extension A-D).
- Menambahkan materi lengkap `intermediate/10-client-side-filter-sort-pagination.md` (format core + extension A-D).
- Menambahkan materi lengkap `intermediate/11-form-multi-step-dan-validasi-lanjutan.md` (format core + extension A-D).
- Menambahkan materi lengkap `intermediate/12-mini-project-search-filter-list.md` (format core + extension A-D).
- Menambahkan materi lengkap `advanced/13-network-resilience-timeout-retry-backoff.md` (format core + extension A-D).

### Changed
- Memperbarui status `intermediate/07-dom-component-pattern-tanpa-framework.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `intermediate/08-event-delegation-dan-dynamic-list.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `intermediate/09-url-state-dan-query-params.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `intermediate/10-client-side-filter-sort-pagination.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `intermediate/11-form-multi-step-dan-validasi-lanjutan.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `intermediate/12-mini-project-search-filter-list.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/13-network-resilience-timeout-retry-backoff.md` di index `README.md` dari `planned` menjadi `available`.

## [0.1.0] - 2026-03-03

### Added
- Menambahkan struktur konten level:
  - `beginner/`
  - `intermediate/`
  - `advanced/`
  - `expert-bridge/`
- Menambahkan template lokal track:
  - `templates/core-topic-template.md`
  - `templates/web-platform-tutorial-template.md`
  - `templates/README.md`
- Menambahkan materi Beginner:
  - `beginner/01-dom-events-fetch-mini-app.md`
  - `beginner/02-dom-state-render-dasar.md`
  - `beginner/03-form-validation-dan-error-ui.md`
  - `beginner/04-fetch-advanced-loading-retry-abort.md`
  - `beginner/05-local-storage-dan-state-persistence.md`
  - `beginner/06-mini-project-dashboard-browser.md`
- Menambahkan placeholder planned untuk modul `intermediate/07` sampai `advanced/18`.
- Menambahkan `expert-bridge/README.md` sebagai panduan transisi.
- Menambahkan log track: `CHANGELOG.md` dan `CHANGELOG-ARCHIVE.md`.

### Changed
- Memperbarui `README.md` track agar menjadi sumber aturan tunggal:
  - gambaran besar
  - tujuan akhir
  - aturan level ringkas
  - index modul per level
- Memperbarui `templates/core-topic-template.md` agar merujuk aturan level ke `README.md`.
- Memperbarui `templates/web-platform-tutorial-template.md` dan `templates/README.md` untuk menegaskan pola `core + extension` (A-D wajib).
- Menghapus folder aturan level terpisah `levels/` agar aturan tidak tersebar.
- Memperbarui status modul Beginner di index `README.md` menjadi `available`.
