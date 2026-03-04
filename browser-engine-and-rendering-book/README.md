# Browser Engine and Rendering Book

Track ini fokus membangun pemahaman mekanisme rendering browser dari level fondasi sampai keputusan optimasi berbasis data.

## Gambaran Besar
`Browser-Engine-and-Rendering-Book` adalah jalur belajar bertahap untuk memahami pipeline render end-to-end:
`parse -> style -> layout -> paint -> composite`.

Alur belajarnya:
- Mulai dari model dasar pipeline dan biaya render per jenis perubahan UI.
- Lanjut ke diagnosis bottleneck dan anti-pattern yang memicu jank.
- Naik ke pengambilan keputusan arsitektur UI dan verifikasi hasil optimasi via DevTools.
- Setelah selesai, lanjut ke `browser-case-studies` dan `web-standards-spec-companion`.

## Roadmap Wireframe
```text
[Beginner]
  Pahami "apa yang terjadi" di pipeline render
  Output: bisa petakan perubahan UI -> stage yang terdampak
       |
       v
[Intermediate]
  Diagnosis "kenapa lambat/jank" pada kasus nyata
  Output: bisa temukan bottleneck dan regression risk
       |
       v
[Advanced]
  Putuskan "optimasi mana paling tepat + trade-off"
  Output: bisa validasi keputusan dengan metric + DevTools evidence
       |
       v
[Expert Bridge]
  Transisi ke case-study/spec mindset
  Output: siap analisis kasus produksi dan validasi asumsi ke spec
```

## Tujuan Akhir
Setelah menyelesaikan track ini, target akhirnya adalah:
- Mampu menjelaskan pipeline render browser secara runtut dan akurat.
- Mampu memprediksi biaya perubahan UI sebelum implementasi.
- Mampu mendiagnosis jank dan membuktikan akar masalah lewat DevTools.
- Mampu memilih strategi optimasi yang tepat berdasarkan metrik, bukan tebakan.

Template utama:
- `../templates/core-topic-template.md`
- `../templates/browser-engine-and-rendering-book-template.md`

## Aturan Level (Ringkas)
- `Beginner`
  - Fokus: model mental pipeline render dan pemetaan trigger dasar.
  - Aturan drill: opsional/ringan.
- `Intermediate`
  - Fokus: diagnosis bottleneck pada interaksi UI nyata.
  - Aturan drill: wajib.
- `Advanced`
  - Fokus: trade-off optimasi, arsitektur UI, dan validasi hasil berbasis data.
  - Aturan drill: wajib.
- `Expert Bridge`
  - Tidak ada modul baru di track ini.
  - Lanjutkan ke `browser-case-studies/` dan `web-standards-spec-companion/`.

## Aturan Track Khusus
- Setiap modul wajib pakai `core-topic-template.md` + section A/B/C dari template track rendering.
- Setiap modul minimal punya:
  - 2 `Perf Metric` dengan target angka dan dampak jika gagal.
  - 1 `Debug Story`.
  - 1 skenario `regression risk` atau failure case render.
  - 1 prosedur verifikasi via DevTools (panel, langkah, indikator sukses).
- Untuk level `Intermediate` dan `Advanced`:
  - Minimal 2 drill prediksi dampak render (`style/layout/paint/composite`) + reasoning.
  - Wajib jelaskan `kapan tidak dipakai` untuk teknik optimasi yang dibahas.

## Index Modul
### Beginner
1. `beginner/01-render-pipeline-dasar.md` (available)
2. `beginner/02-style-recalc-dan-selector-cost.md` (available)
3. `beginner/03-layout-flow-dan-reflow-trigger.md` (available)
4. `beginner/04-paint-cost-dan-layer-basics.md` (available)
5. `beginner/05-compositing-transform-opacity.md` (available)
6. `beginner/06-mini-project-render-audit-dasar.md` (available)

### Intermediate
1. `intermediate/07-layout-thrashing-dan-batching-read-write.md` (available)
2. `intermediate/08-scroll-jank-dan-main-thread-contention.md` (available)
3. `intermediate/09-animation-pipeline-css-vs-js.md` (available)
4. `intermediate/10-list-rendering-large-dom-windowing-basics.md` (available)
5. `intermediate/11-image-font-loading-impact-ke-render.md` (available)
6. `intermediate/12-mini-project-jank-diagnosis-lab.md` (available)

### Advanced
1. `advanced/13-render-budget-dan-frame-time-trade-off.md` (draft)
2. `advanced/14-layer-strategy-memory-vs-smoothness.md` (draft)
3. `advanced/15-containment-content-visibility-dan-isolation.md` (draft)
4. `advanced/16-measurement-playbook-performance-panel-deep-dive.md` (draft)
5. `advanced/17-guardrail-testing-untuk-render-regression.md` (draft)
6. `advanced/18-capstone-rendering-architecture-review.md` (draft)

### Expert Bridge
1. `expert-bridge/README.md` (available)

## Referensi Track
- Template penulisan: `../templates/`
- Materi per level: `beginner/`, `intermediate/`, `advanced/`, `expert-bridge/`

## Track Change Log
- Semua perubahan khusus track ini dicatat di `CHANGELOG.md`.
- Jika log track melebihi 200 baris, pindahkan entri lama ke `CHANGELOG-ARCHIVE.md`.
