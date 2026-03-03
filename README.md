# Browser-Runtime

## ECOSYSTEM ROADMAP (HIGH LEVEL)

```text
[Web-Platform-Tutorial]
  Tujuan: mulai cepat memahami lingkungan browser (DOM, BOM, fetch, console)
  Output: mini app browser sederhana berjalan
        |
        v
[Browser-Runtime-First-Principles]
  Tujuan: bangun mental model inti runtime (call stack, task/microtask, GC, scope, closure)
  Output: bisa prediksi urutan eksekusi & debug issue dasar async/state
        |
        v
[Browser-Engine-and-Rendering-Book]
  Tujuan: pahami pipeline browser end-to-end (parse -> style -> layout -> paint -> composite)
  Output: bisa ambil keputusan arsitektur UI/performa berbasis mekanisme internal
        |
        v
[Runtime-Exercises-and-Katas]
  Tujuan: latihan terstruktur untuk event loop, async patterns, rendering cost, memory leak
  Output: eksekusi teknis lebih konsisten dan minim bug regresi
        |
        v
[Browser-Case-Studies]
  Tujuan: belajar dari kasus nyata (jank, race condition, hydration mismatch, slow TTI)
  Output: problem-solving mindset level production
        |
        v
[Web-Standards-Spec-Companion]
  Tujuan: jembatan ke spesifikasi (HTML, DOM, Fetch, ECMAScript, WHATWG event loop)
  Output: reasoning teknis tingkat lanjut, bisa validasi asumsi langsung ke spec
```

## Change Log

- Semua perubahan penting wajib dicatat di [`CHANGELOG.md`](./CHANGELOG.md).
- Gunakan section `Unreleased` untuk perubahan terbaru sebelum rilis versi.
- Batas `CHANGELOG.md` adalah 200 baris; jika lebih, pindahkan entri lama ke [`CHANGELOG-ARCHIVE.md`](./CHANGELOG-ARCHIVE.md).

## Format Konten

Gunakan pola `core format + track-specific extension`:
- Core format dipakai di semua track.
- Setiap track menambahkan section khusus sesuai tipe kontennya.

Template yang dipakai:
- Core: [`templates/core-topic-template.md`](./templates/core-topic-template.md)
- Web Platform Tutorial: [`web-platform-tutorial/templates/web-platform-tutorial-template.md`](./web-platform-tutorial/templates/web-platform-tutorial-template.md)
- Browser Runtime First Principles: [`templates/browser-runtime-first-principles-template.md`](./templates/browser-runtime-first-principles-template.md)
- Browser Engine and Rendering Book: [`templates/browser-engine-and-rendering-book-template.md`](./templates/browser-engine-and-rendering-book-template.md)
- Runtime Exercises and Katas: [`templates/runtime-exercises-and-katas-template.md`](./templates/runtime-exercises-and-katas-template.md)
- Browser Case Studies: [`templates/browser-case-studies-template.md`](./templates/browser-case-studies-template.md)
- Web Standards Spec Companion: [`templates/web-standards-spec-companion-template.md`](./templates/web-standards-spec-companion-template.md)
