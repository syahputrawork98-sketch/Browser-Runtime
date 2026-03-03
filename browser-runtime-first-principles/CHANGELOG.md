# Changelog - Browser Runtime First Principles

Semua perubahan penting khusus track `browser-runtime-first-principles` dicatat di file ini.

Format mengikuti prinsip [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) dan Semantic Versioning.

## Aturan Versi
- Gunakan format versi: `MAJOR.MINOR.PATCH`.
- Rilis wajib memakai tanggal format `YYYY-MM-DD`.
- Perubahan baru dicatat dulu di `Unreleased`.

## Aturan Batas Baris
- `browser-runtime-first-principles/CHANGELOG.md` maksimal **200 baris**.
- Jika lebih dari 200 baris, pindahkan entri lama ke `CHANGELOG-ARCHIVE.md`.
- Pertahankan entri terbaru (termasuk `Unreleased`) di file utama.

## [Unreleased]
Belum ada perubahan.

## [0.1.0] - 2026-03-04

### Added
- Menambahkan struktur level track:
  - `beginner/`
  - `intermediate/`
  - `advanced/`
  - `expert-bridge/`
- Menambahkan template lokal track:
  - `templates/core-topic-template.md`
  - `templates/browser-runtime-first-principles-template.md`
  - `templates/README.md`
- Menambahkan sistem log track:
  - `CHANGELOG.md`
  - `CHANGELOG-ARCHIVE.md`
- Menambahkan index roadmap 18 modul (`01-18`) dengan status awal planned.
- Menyelesaikan materi Beginner `01-06` dan mengubah statusnya menjadi `available`.

### Changed
- Memindahkan `01-call-stack-task-microtask.md` ke `beginner/01-call-stack-task-microtask.md`.
- Memperbarui `README.md` track dengan:
  - aturan level
  - aturan drill wajib
  - aturan runtime khusus (`Model Mental`, `Failure Case`, `Spec Hint`)
  - index modul per level
