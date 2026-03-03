# DOM Event Dispatch dan Listener Flow

## Meta
- Level: `Beginner`
- Estimasi durasi: `35-50 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham event listener dasar (`addEventListener`).
  - [ ] Paham callback sync dan urutan eksekusi sederhana.
- Kamus mini:
  - `[baru]` event dispatch: proses browser menyalurkan event ke target dan ancestor.
  - `[baru]` bubbling: event bergerak dari target ke parent hingga root.
  - `[baru]` capturing: event bergerak dari root ke target sebelum bubbling.
  - `[baru]` `stopPropagation`: menghentikan propagasi event ke level berikutnya.

## 1) Pengantar Singkat Topik
Topik ini membahas bagaimana event DOM berjalan dari elemen target ke elemen lain di sekitarnya. Tujuannya agar kamu bisa memprediksi urutan listener saat ada elemen bertingkat.

## 2) Big Picture
Bug interaksi UI sering terjadi bukan karena fungsi utama salah, tapi karena listener parent/child berjalan di urutan yang tidak dipahami. Tanpa mental model dispatch, klik sederhana bisa men-trigger aksi dobel atau aksi yang tidak diinginkan. Dengan memahami capturing, target, dan bubbling, kamu bisa mendesain interaksi DOM yang stabil dan terkontrol.

## 3) Small Picture
- Event memiliki fase: capturing -> target -> bubbling.
- Default `addEventListener` berada di bubbling phase.
- Listener parent tetap bisa terpanggil saat child diklik jika propagasi tidak dihentikan.
- `stopPropagation` menghentikan perjalanan event ke ancestor selanjutnya.

## 4) Wireframe Alur Konsep
```text
[root]
  -> [parent]
    -> [target]
    -> [parent]
  -> [root]
```
- Alur utama: event mencapai target lalu bubble naik.
- Alur jalan: listener capturing bisa jalan lebih dulu jika diaktifkan.
- Alur error: listener parent ikut terpanggil padahal diharapkan hanya target.

## 5) Analogi Dunia Nyata
Seperti informasi rapat: dari pimpinan ke anggota (capturing), dibahas di meja peserta utama (target), lalu laporan naik lagi ke atasan (bubbling).

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: tombol dalam kartu/list, menu dropdown, event delegation.
- Alasan pakai: mengatur interaksi nested element tanpa konflik.
- Kapan tidak dipakai: elemen tunggal tanpa hierarki interaksi.

## 7) Contoh Sederhana + Bedah Output
```html
<div id="parent">
  Parent
  <button id="child">Click Child</button>
</div>

<script>
  const parent = document.querySelector('#parent');
  const child = document.querySelector('#child');

  parent.addEventListener('click', () => {
    console.log('parent-bubble');
  });

  child.addEventListener('click', () => {
    console.log('child-target');
  });
</script>
```
Bedah output saat klik tombol `child`:
1. Listener target (`child-target`) berjalan.
2. Event bubble ke parent.
3. Listener parent (`parent-bubble`) berjalan.

## 8) Jebakan Umum (Pitfalls)
- Mengira klik child tidak akan pernah memicu listener parent.
- Menaruh `stopPropagation` sembarang sehingga fitur lain tidak jalan.
- Tidak membedakan capturing dan bubbling saat debugging urutan listener.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
// parent: bubble listener
// child: bubble listener
// klik child
```
Kunci:
- Output urutan: `child` lalu `parent`
- Alasan: target phase dulu, lalu bubbling ke ancestor.

Drill 2:
```js
parent.addEventListener('click', () => console.log('parent-capture'), true);
parent.addEventListener('click', () => console.log('parent-bubble'));
child.addEventListener('click', () => console.log('child'));
// klik child
```
Kunci:
- Output: `parent-capture`, `child`, `parent-bubble`
- Alasan: capturing listener parent berjalan sebelum target, lalu bubbling parent.

Drill 3:
```js
parent.addEventListener('click', () => console.log('parent'));
child.addEventListener('click', (e) => {
  e.stopPropagation();
  console.log('child');
});
// klik child
```
Kunci:
- Output: `child`
- Alasan: propagasi dihentikan di target, jadi parent tidak menerima bubble.

## 10) Debug Story
- Gejala: klik tombol hapus di item list juga membuka detail item.
- Akar masalah: listener parent card ikut terpanggil karena bubbling.
- Langkah debug: pasang log di target dan parent, cek urutan listener.
- Solusi: di tombol hapus gunakan `stopPropagation` secara terukur atau pisahkan area klik parent.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan fase capturing, target, dan bubbling.
- [ ] Bisa memprediksi urutan listener di parent-child.
- [ ] Bisa menyelesaikan konflik interaksi nested element.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: callback dan event listener dasar.
- Topik terkait: event delegation dan dynamic list.
- Referensi tambahan: konsep event propagation di DOM.

## 13) Model Mental
- Entitas utama: event target, ancestor chain, listener per fase.
- Urutan proses: capturing ke target -> target listener -> bubbling ke atas.
- Invariant (aturan yang selalu benar): listener default tanpa opsi capture berjalan pada bubbling phase.

## 14) Failure Case
- Skenario gagal: tombol di dalam card punya aksi spesifik, tapi aksi card parent juga ikut jalan.
- Akar penyebab runtime: event bubble melewati target ke parent tanpa diblok.
- Perbaikan: gunakan `stopPropagation` pada aksi child yang memang tidak boleh memicu parent.

## 15) Spec Hint
- Section spec terkait (ringkas): DOM event dispatch dan propagation path.
- Kata kunci pencarian spec: `event dispatch`, `bubbling`, `capturing`, `stopPropagation`.
