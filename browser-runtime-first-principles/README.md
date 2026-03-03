# Browser Runtime First Principles

Track ini fokus membangun mental model runtime browser agar prediksi eksekusi dan debugging async menjadi akurat.

## Gambaran Besar
`Browser Runtime First Principles` adalah jalur belajar bertahap untuk memahami mekanisme eksekusi JavaScript di browser dari level konsep inti sampai reasoning tingkat lanjut.

Alur belajarnya:
- Mulai dari call stack, task/microtask, promise jobs, timer, dan event dispatch.
- Lanjut ke kondisi nyata: race condition, cancellation, memory leak, dan konsistensi state.
- Naik ke pengambilan keputusan: scheduling, observability runtime, dan testing ordering/timing.
- Setelah selesai, lanjut ke latihan intensif dan jembatan spec.

## Tujuan Akhir
Setelah menyelesaikan track ini, target akhirnya adalah:
- Mampu memprediksi urutan eksekusi sync/microtask/macrotask tanpa menebak.
- Mampu menjelaskan penyebab bug timing dengan model mental yang benar.
- Mampu melakukan debugging runtime berbasis bukti (log, trace, timing).
- Siap lanjut ke `runtime-exercises-and-katas` dan `web-standards-spec-companion`.

## Aturan Level (Ringkas)
- `Beginner`
  - Fokus: membangun model mental dasar runtime browser.
  - Aturan drill: wajib.
- `Intermediate`
  - Fokus: stabilitas alur async pada kondisi non-ideal.
  - Aturan drill: wajib.
- `Advanced`
  - Fokus: trade-off scheduling, observability, dan validasi teknis.
  - Aturan drill: wajib.
- `Expert Bridge`
  - Tidak ada modul baru di track ini.
  - Lanjutkan ke `runtime-exercises-and-katas/` dan `web-standards-spec-companion/`.

## Aturan Track Khusus
- Drill prediksi output wajib di semua level.
- Setiap modul minimal memiliki:
  - 3 prediksi output drill + kunci jawaban reasoning.
  - 1 debug story.
  - 1 failure case.
- Gunakan tambahan section runtime:
  - `13) Model Mental`
  - `14) Failure Case`
  - `15) Spec Hint`

## Index Modul
### Beginner
1. `beginner/01-call-stack-task-microtask.md` (available)
2. `beginner/02-promise-jobs-dan-urutan-eksekusi.md` (available)
3. `beginner/03-timer-delay-minimum-dan-clamping.md` (available)
4. `beginner/04-dom-event-dispatch-dan-listener-flow.md` (available)
5. `beginner/05-render-tick-dan-ui-update-timing.md` (available)
6. `beginner/06-mini-project-runtime-trace-playground.md` (available)

### Intermediate
1. `intermediate/07-microtask-starvation-dan-fairness.md` (available)
2. `intermediate/08-cancellation-pattern-abortcontroller.md` (available)
3. `intermediate/09-race-condition-dan-last-write-wins.md` (available)
4. `intermediate/10-runtime-side-effects-dan-state-consistency.md` (available)
5. `intermediate/11-memory-leak-dasar-listener-timer-closure.md` (available)
6. `intermediate/12-mini-project-async-workflow-inspector.md` (available)

### Advanced
1. `advanced/13-scheduling-prioritas-dan-trade-off.md` (planned)
2. `advanced/14-batching-dom-read-write-untuk-runtime-stability.md` (planned)
3. `advanced/15-backpressure-dan-request-burst-control.md` (planned)
4. `advanced/16-observability-runtime-log-timing-trace.md` (planned)
5. `advanced/17-testing-runtime-ordering-dan-timing.md` (planned)
6. `advanced/18-capstone-runtime-debugging-lab.md` (planned)

### Expert Bridge
1. `expert-bridge/README.md` (available)

## Referensi Track
- Template penulisan: [`templates/`](./templates/)
- Materi per level: `beginner/`, `intermediate/`, `advanced/`, `expert-bridge/`

## Track Change Log
- Semua perubahan khusus track ini dicatat di [`CHANGELOG.md`](./CHANGELOG.md).
- Jika log track melebihi 200 baris, pindahkan entri lama ke [`CHANGELOG-ARCHIVE.md`](./CHANGELOG-ARCHIVE.md).
