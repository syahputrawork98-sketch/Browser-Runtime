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

### Added
- Tidak ada.

### Changed
- Tidak ada.

## [0.3.0] - 2026-03-04

### Added
- Menambahkan materi lengkap `advanced/13-scheduling-prioritas-dan-trade-off.md`.
- Menambahkan materi lengkap `advanced/14-batching-dom-read-write-untuk-runtime-stability.md`.
- Menambahkan materi lengkap `advanced/15-backpressure-dan-request-burst-control.md`.
- Menambahkan materi lengkap `advanced/16-observability-runtime-log-timing-trace.md`.
- Menambahkan materi lengkap `advanced/17-testing-runtime-ordering-dan-timing.md`.
- Menambahkan materi lengkap `advanced/18-capstone-runtime-debugging-lab.md`.

### Changed
- Memperbarui status `advanced/13-scheduling-prioritas-dan-trade-off.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/14-batching-dom-read-write-untuk-runtime-stability.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/15-backpressure-dan-request-burst-control.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/16-observability-runtime-log-timing-trace.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/17-testing-runtime-ordering-dan-timing.md` di index `README.md` dari `planned` menjadi `available`.
- Memperbarui status `advanced/18-capstone-runtime-debugging-lab.md` di index `README.md` dari `planned` menjadi `available`.
- Menyelesaikan milestone level Advanced (`13-18`) pada track `browser-runtime-first-principles`.

## [0.2.0] - 2026-03-04

### Added
- Menambahkan materi lengkap level Intermediate:
  - `intermediate/07-microtask-starvation-dan-fairness.md`
  - `intermediate/08-cancellation-pattern-abortcontroller.md`
  - `intermediate/09-race-condition-dan-last-write-wins.md`
  - `intermediate/10-runtime-side-effects-dan-state-consistency.md`
  - `intermediate/11-memory-leak-dasar-listener-timer-closure.md`
  - `intermediate/12-mini-project-async-workflow-inspector.md`

### Changed
- Memperbarui status modul Intermediate `07-12` di index `README.md` dari `planned` menjadi `available`.

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
