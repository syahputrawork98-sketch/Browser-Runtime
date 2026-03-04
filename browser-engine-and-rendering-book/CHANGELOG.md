# Changelog - Browser Engine and Rendering Book

Semua perubahan penting khusus track `browser-engine-and-rendering-book` dicatat di file ini.

Format mengikuti prinsip [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) dan Semantic Versioning.

## Aturan Versi
- Gunakan format versi: `MAJOR.MINOR.PATCH`.
- Rilis wajib memakai tanggal format `YYYY-MM-DD`.
- Perubahan baru dicatat dulu di `Unreleased`.

## Aturan Batas Baris
- `browser-engine-and-rendering-book/CHANGELOG.md` maksimal **200 baris**.
- Jika lebih dari 200 baris, pindahkan entri lama ke `CHANGELOG-ARCHIVE.md`.
- Pertahankan entri terbaru (termasuk `Unreleased`) di file utama.

## [Unreleased]

### Added
- Menambahkan roadmap level-based di `README.md`:
  - `Gambaran Besar`, `Roadmap Wireframe`, `Tujuan Akhir`
  - `Aturan Level`, `Aturan Track Khusus`
  - `Index Modul` Beginner/Intermediate/Advanced/Expert Bridge
  - `Referensi Track` dan `Track Change Log`
- Menyusun ulang struktur konten ke folder level:
  - `beginner/`, `intermediate/`, `advanced/`, `expert-bridge/`
  - Memindahkan `01-render-pipeline-dasar.md` ke `beginner/01-render-pipeline-dasar.md`
  - Menambahkan file stub draft untuk modul `02` s.d. `18`
  - Menambahkan `expert-bridge/README.md`
- Menambahkan dan melengkapi materi beginner `01-06` dengan format `core + A/B/C`:
  - `beginner/01-render-pipeline-dasar.md`
  - `beginner/02-style-recalc-dan-selector-cost.md`
  - `beginner/03-layout-flow-dan-reflow-trigger.md`
  - `beginner/04-paint-cost-dan-layer-basics.md`
  - `beginner/05-compositing-transform-opacity.md`
  - `beginner/06-mini-project-render-audit-dasar.md`
- Menambahkan dan melengkapi materi intermediate:
  - `intermediate/07-layout-thrashing-dan-batching-read-write.md` (format `core + A/B/C`, termasuk 2 drill prediksi wajib untuk level intermediate)
  - `intermediate/08-scroll-jank-dan-main-thread-contention.md` (format `core + A/B/C`, termasuk 2 drill prediksi wajib untuk level intermediate)
  - `intermediate/09-animation-pipeline-css-vs-js.md` (format `core + A/B/C`, termasuk 2 drill prediksi wajib untuk level intermediate)
  - `intermediate/10-list-rendering-large-dom-windowing-basics.md` (format `core + A/B/C`, termasuk 2 drill prediksi wajib untuk level intermediate)
  - `intermediate/11-image-font-loading-impact-ke-render.md` (format `core + A/B/C`, termasuk 2 drill prediksi wajib untuk level intermediate)
  - `intermediate/12-mini-project-jank-diagnosis-lab.md` (format `core + A/B/C`, termasuk 2 drill prediksi wajib untuk level intermediate)

### Changed
- Memperbarui status modul beginner `01-06` di index `README.md` menjadi `available`.
- Memperbarui status modul `intermediate/07-layout-thrashing-dan-batching-read-write.md` di index `README.md` menjadi `available`.
- Memperbarui status modul `intermediate/08-scroll-jank-dan-main-thread-contention.md` di index `README.md` menjadi `available`.
- Memperbarui status modul `intermediate/09-animation-pipeline-css-vs-js.md` di index `README.md` menjadi `available`.
- Memperbarui status modul `intermediate/10-list-rendering-large-dom-windowing-basics.md` di index `README.md` menjadi `available`.
- Memperbarui status modul `intermediate/11-image-font-loading-impact-ke-render.md` di index `README.md` menjadi `available`.
- Memperbarui status modul `intermediate/12-mini-project-jank-diagnosis-lab.md` di index `README.md` menjadi `available`.
- Menindaklanjuti hasil review beginner modules:
  - menambahkan section `A/B/C` pada modul `01`,
  - menyelaraskan contoh HTML/CSS pada modul `02`,
  - menambahkan label eksplisit `Regression risk` pada modul `01-06`,
  - memperbaiki typo `meterik` menjadi `metrik` pada modul `06`.
