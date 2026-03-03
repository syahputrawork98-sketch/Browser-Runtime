# Performance Basics: Debounce, Throttle, dan Reflow

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan level: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event input/scroll dan render DOM.
  - [ ] Paham konsep state + update UI.
- Kamus mini:
  - `[baru]` debounce: menunda eksekusi sampai jeda tertentu tanpa event baru.
  - `[baru]` throttle: membatasi eksekusi maksimal sekali per interval waktu.
  - `[baru]` reflow: proses browser menghitung ulang layout.
  - `[baru]` layout thrashing: baca-tulis layout berulang yang memicu reflow berlebihan.

## 1) Pengantar Singkat Topik
Topik ini membahas optimasi performa dasar untuk event yang sering terjadi, seperti `input`, `scroll`, dan `resize`. Fokusnya: kapan pakai debounce, kapan pakai throttle, dan bagaimana menghindari reflow yang tidak perlu.

## 2) Big Picture
UI yang tersendat sering bukan karena algoritma berat, tetapi karena handler event menembak terlalu sering atau manipulasi DOM yang buruk. Debounce/throttle menurunkan frekuensi kerja JS, sementara pengelolaan reflow menekan biaya layout dari browser. Kombinasi keduanya memberi interaksi yang lebih halus. Setelah materi ini, kamu bisa membuat fitur yang tetap responsif saat user mengetik cepat atau scroll panjang.

## 3) Small Picture
- Gunakan debounce untuk aksi yang menunggu user selesai input.
- Gunakan throttle untuk aksi berkala saat event terus terjadi (mis. scroll).
- Batch operasi DOM: kumpulkan perubahan, minimalkan baca-tulis layout campur aduk.
- Hindari membaca properti layout berkali-kali setelah menulis style.

## 4) Wireframe Alur Konsep
```text
[event berulang cepat]
  -> [debounce/throttle]
  -> [handler lebih jarang]
  -> [update DOM terkontrol]
  -> [frame lebih stabil]
```
- Alur utama: event padat -> eksekusi terkontrol -> UI tetap halus.
- Alur jalan: handler ringan tapi tetap dibatasi agar hemat resource.
- Alur error: handler sering + layout thrashing -> jank/fps turun.

## 5) Analogi Dunia Nyata
Debounce seperti menunggu orang selesai bicara sebelum mencatat, throttle seperti satpam yang hanya memperbolehkan satu orang masuk tiap beberapa detik.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: live search, infinite scroll, resize listener, hitung posisi elemen.
- Alasan pakai: menjaga responsivitas UI dan mengurangi beban CPU.
- Kapan tidak dipakai: event jarang terjadi dan handler sangat ringan.

## 7) Contoh Sederhana + Bedah Output
```html
<input id="searchInput" placeholder="Cari..." />
<p id="searchStatus">Search idle</p>

<p id="scrollStatus">Scroll idle</p>
<div style="height: 1200px; border: 1px solid #ccc; margin-top: 8px;">
  Area scroll demo
</div>

<script>
  const searchInput = document.querySelector('#searchInput');
  const searchStatus = document.querySelector('#searchStatus');
  const scrollStatus = document.querySelector('#scrollStatus');

  function debounce(fn, delayMs) {
    let timer = null;
    return (...args) => {
      clearTimeout(timer);
      timer = setTimeout(() => fn(...args), delayMs);
    };
  }

  function throttle(fn, intervalMs) {
    let last = 0;
    return (...args) => {
      const now = Date.now();
      if (now - last >= intervalMs) {
        last = now;
        fn(...args);
      }
    };
  }

  const handleSearch = debounce((value) => {
    searchStatus.textContent = `Search query: ${value}`;
  }, 300);

  const handleScroll = throttle(() => {
    scrollStatus.textContent = `ScrollY: ${Math.round(window.scrollY)}`;
  }, 200);

  searchInput.addEventListener('input', (event) => {
    handleSearch(event.target.value.trim());
  });

  window.addEventListener('scroll', handleScroll);

  // Contoh buruk (layout thrashing): read->write berulang dalam loop
  // Hindari pola seperti ini pada elemen banyak:
  // const h = box.offsetHeight; box.style.height = (h + 1) + 'px';
</script>
```
Bedah output:
1. Saat mengetik cepat, status search tidak update di tiap karakter, tetapi setelah jeda 300ms.
2. Saat scroll, status update berkala (maks sekali per 200ms), bukan setiap event scroll.
3. UI tetap lebih stabil dibanding handler mentah tanpa pembatasan.

## 8) Jebakan Umum (Pitfalls)
- Salah pilih debounce/throttle sehingga UX terasa lag atau terlalu lambat update.
- Interval terlalu agresif (mis. throttle 1000ms untuk interaksi yang butuh respons cepat).
- Debounce pada aksi yang wajib real-time.
- Baca properti layout (`offsetHeight`, `getBoundingClientRect`) setelah write style berulang.

## 9) Prediksi Output Drill + Kunci Jawaban
Catatan:
- Ikuti aturan drill di `../README.md` section `Aturan Level (Ringkas)`.

Drill:
```js
let count = 0;
function throttle(fn, ms) {
  let last = 0;
  return () => {
    const now = Date.now();
    if (now - last >= ms) {
      last = now;
      fn();
    }
  };
}

const run = throttle(() => {
  count += 1;
  console.log(count);
}, 200);

run();
run();
setTimeout(run, 250);
```
Kunci:
- Output:
  - `1`
  - `2`
- Alasan: panggilan kedua langsung diblok karena interval belum lewat, panggilan setelah 250ms lolos.

## 10) Debug Story
- Gejala: input search terasa patah-patah saat user mengetik cepat.
- Akar masalah: tiap event `input` langsung memicu filter berat + render list besar.
- Langkah debug: ukur frekuensi handler dengan log timestamp dan profil di Performance tab.
- Solusi: debounce input, pecah kerja berat, dan kurangi operasi DOM per event.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa memilih debounce vs throttle sesuai skenario.
- [ ] Bisa mengidentifikasi gejala layout thrashing.
- [ ] Bisa menerapkan optimasi sederhana untuk event padat.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `intermediate/10-client-side-filter-sort-pagination.md`.
- Topik terkait: `advanced/16-accessibility-basics-keyboard-aria.md`.
- Referensi tambahan: MDN event loop rendering, `requestAnimationFrame`, dan performance profiling.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: DevTools (`Performance` dan `Console`).
- File awal: input search, area panjang untuk scroll, dan elemen status.

## B) Langkah Praktik
1. Implement helper `debounce` dan `throttle`.
2. Terapkan debounce ke live search.
3. Terapkan throttle ke listener `scroll`.
4. Profiling sebelum/sesudah di tab Performance.

## C) Expected Output
- UI yang diharapkan: interaksi search/scroll lebih halus dengan update terkontrol.
- Output console/network yang diharapkan: frekuensi log handler menurun jelas.

## D) Troubleshooting
- Masalah umum 1 + solusi: tidak ada update sama sekali, cek closure timer/last pada helper.
- Masalah umum 2 + solusi: tetap lag, cek operasi DOM berat di dalam handler.
- Masalah umum 3 + solusi: hasil search terasa terlambat, kecilkan delay debounce secukupnya.
