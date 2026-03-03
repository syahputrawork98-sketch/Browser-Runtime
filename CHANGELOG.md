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

## [0.4.0] - 2026-03-04

### Changed
- Menerapkan pola changelog bertingkat:
  - `CHANGELOG.md` root untuk ringkasan strategis lintas track.
  - `CHANGELOG.md` per track untuk detail progres modul.
- Menyelesaikan milestone `browser-runtime-first-principles` level Beginner (`01-06`) dan merilisnya di changelog track sebagai `0.1.0`.
- Menambahkan progres awal `browser-runtime-first-principles` level Intermediate dengan menyelesaikan modul `07`.
- Melanjutkan progres `browser-runtime-first-principles` level Intermediate dengan menyelesaikan modul `08`.
- Melanjutkan progres `browser-runtime-first-principles` level Intermediate dengan menyelesaikan modul `09`.
- Melanjutkan progres `browser-runtime-first-principles` level Intermediate dengan menyelesaikan modul `10`.
- Melanjutkan progres `browser-runtime-first-principles` level Intermediate dengan menyelesaikan modul `11`.
- Menyelesaikan milestone `browser-runtime-first-principles` level Intermediate (`07-12`).
- Merilis `browser-runtime-first-principles` versi `0.2.0` untuk milestone Intermediate (`07-12`) dan memulai progres Advanced dengan menyelesaikan modul `13`.
- Melanjutkan progres `browser-runtime-first-principles` level Advanced dengan menyelesaikan modul `14`.
- Melanjutkan progres `browser-runtime-first-principles` level Advanced dengan menyelesaikan modul `15`.
- Melanjutkan progres `browser-runtime-first-principles` level Advanced dengan menyelesaikan modul `16`.
- Melanjutkan progres `browser-runtime-first-principles` level Advanced dengan menyelesaikan modul `17`.
- Menyelesaikan milestone `browser-runtime-first-principles` level Advanced (`13-18`) dengan menuntaskan modul `18`.
- Merilis `browser-runtime-first-principles` versi `0.3.0` untuk milestone Advanced (`13-18`).

## [0.3.0] - 2026-03-04

### Changed
- Menyelesaikan batch Advanced pada `web-platform-tutorial` (modul `14-18`) dan memperbarui status index track menjadi `available`.

## [0.2.0] - 2026-03-04

### Added
- Menambahkan roadmap ekosistem Browser Runtime di `README.md`.
- Menambahkan struktur folder fase roadmap utama:
  - `web-platform-tutorial/`
  - `browser-runtime-first-principles/`
  - `browser-engine-and-rendering-book/`
  - `runtime-exercises-and-katas/`
  - `browser-case-studies/`
  - `web-standards-spec-companion/`
- Menambahkan sistem changelog (`CHANGELOG.md`, `CHANGELOG-ARCHIVE.md`) dan template konten lintas track.
- Menambahkan baseline materi awal untuk semua track.

### Changed
- Menetapkan kebijakan changelog dan format konten di `README.md`.
- Menyelaraskan struktur serta dokumentasi `web-platform-tutorial` sebagai track bertingkat (Beginner -> Intermediate -> Advanced -> Expert Bridge).
- Menyelesaikan materi `web-platform-tutorial` sampai Intermediate (`07-12`) dan Advanced awal (`13`), termasuk sinkronisasi status modul di index track.
