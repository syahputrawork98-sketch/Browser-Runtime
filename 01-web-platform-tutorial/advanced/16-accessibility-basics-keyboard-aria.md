# Accessibility Basics: Keyboard dan ARIA

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan level: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham struktur HTML semantik dasar.
  - [ ] Paham event keyboard dan update DOM.
- Kamus mini:
  - `[baru]` accessibility (a11y): praktik agar aplikasi bisa dipakai semua orang.
  - `[baru]` keyboard navigation: navigasi UI tanpa mouse.
  - `[baru]` focus: elemen aktif untuk interaksi keyboard.
  - `[baru]` ARIA: atribut tambahan untuk membantu teknologi bantu memahami UI.
  - `[baru]` screen reader: software pembaca konten layar.

## 1) Pengantar Singkat Topik
Topik ini membahas fondasi aksesibilitas praktis untuk UI browser: navigasi keyboard yang benar dan penggunaan ARIA secukupnya. Targetnya adalah komponen bisa dipakai tanpa mouse dan tetap terbaca jelas oleh screen reader.

## 2) Big Picture
Banyak fitur tampak normal untuk pengguna mouse, tetapi rusak total untuk pengguna keyboard atau pembaca layar. Aksesibilitas bukan bonus, melainkan kualitas dasar produk. Dengan fokus order, role, label, dan state ARIA yang benar, aplikasi jadi lebih inklusif dan lebih mudah diuji. Dampak samping positifnya: struktur UI lebih rapi dan robust.

## 3) Small Picture
- Gunakan elemen semantik bawaan (`button`, `input`, `label`) sebelum menambah ARIA.
- Pastikan semua aksi utama bisa dijalankan dengan keyboard.
- Jaga urutan fokus logis dan terlihat jelas.
- Sinkronkan state visual dengan state aksesibilitas (`aria-expanded`, `aria-live`, dll.).

## 4) Wireframe Alur Konsep
```text
[user tekan Tab/Enter/Space]
  -> [fokus pindah ke elemen interaktif]
  -> [aksi dijalankan]
  -> [status diumumkan (aria-live) + UI update]
```
- Alur utama: user keyboard bisa menjalankan flow yang sama dengan mouse.
- Alur jalan: perubahan status tersampaikan ke screen reader.
- Alur error: elemen terlihat klik-able tapi tidak focusable/terbaca.

## 5) Analogi Dunia Nyata
Seperti gedung dengan tangga, lift, dan petunjuk suara: semua orang bisa mencapai ruangan yang sama lewat jalur berbeda.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: form, menu, modal, tab, accordion, daftar interaktif.
- Alasan pakai: memastikan fitur dapat dioperasikan lintas kemampuan pengguna.
- Kapan tidak dipakai: tidak ada pengecualian; minimal a11y selalu diperlukan.

## 7) Contoh Sederhana + Bedah Output
```html
<button id="toggleBtn" aria-expanded="false" aria-controls="detailPanel">
  Tampilkan Detail
</button>

<section id="detailPanel" hidden>
  <h3>Detail Informasi</h3>
  <p>Konten ini muncul saat tombol diaktifkan.</p>
</section>

<p id="statusLive" aria-live="polite"></p>

<script>
  const toggleBtn = document.querySelector('#toggleBtn');
  const detailPanel = document.querySelector('#detailPanel');
  const statusLive = document.querySelector('#statusLive');

  function setExpanded(nextExpanded) {
    toggleBtn.setAttribute('aria-expanded', String(nextExpanded));
    detailPanel.hidden = !nextExpanded;
    toggleBtn.textContent = nextExpanded ? 'Sembunyikan Detail' : 'Tampilkan Detail';
    statusLive.textContent = nextExpanded
      ? 'Panel detail dibuka'
      : 'Panel detail ditutup';
  }

  toggleBtn.addEventListener('click', () => {
    const isExpanded = toggleBtn.getAttribute('aria-expanded') === 'true';
    setExpanded(!isExpanded);
  });

  // Mouse + keyboard akan bekerja otomatis karena elemen adalah <button>.
</script>
```
Bedah output:
1. Tombol bisa diaktifkan dengan mouse, `Enter`, atau `Space`.
2. `aria-expanded` selalu sinkron dengan kondisi panel terbuka/tertutup.
3. Perubahan status diumumkan lewat area `aria-live`.

## 8) Jebakan Umum (Pitfalls)
- Membuat elemen klik dari `div` tanpa role/focus/keyboard handler.
- Mengandalkan warna saja untuk menyampaikan status error/sukses.
- State visual berubah tapi atribut ARIA tidak ikut berubah.
- Menghapus outline focus tanpa pengganti yang jelas.

## 9) Prediksi Output Drill + Kunci Jawaban
Catatan:
- Ikuti aturan drill di `../README.md` section `Aturan Level (Ringkas)`.

Drill:
```js
const btn = document.createElement('button');
btn.setAttribute('aria-expanded', 'false');
btn.setAttribute('aria-expanded', 'true');
console.log(btn.getAttribute('aria-expanded'));
```
Kunci:
- Output: `true`
- Alasan: nilai atribut terakhir menimpa nilai sebelumnya.

## 10) Debug Story
- Gejala: menu dropdown bisa dibuka dengan klik, tapi pengguna keyboard tidak bisa akses submenu.
- Akar masalah: trigger memakai elemen non-semantik tanpa dukungan fokus/keyboard.
- Langkah debug: uji dengan `Tab`, `Enter`, `Space`, dan cek tree a11y di DevTools.
- Solusi: ganti ke `button`, tambahkan `aria-expanded` + relasi kontrol, dan pastikan fokus bergerak logis.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membuat interaksi utama berjalan penuh via keyboard.
- [ ] Bisa menerapkan ARIA dasar yang relevan tanpa overuse.
- [ ] Bisa memverifikasi sinkronisasi state visual dan state aksesibilitas.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: HTML semantik dasar dan event handling.
- Topik terkait: `advanced/17-testing-basics-dom-and-api-flow.md`.
- Referensi tambahan: MDN Accessibility, WAI-ARIA Authoring Practices.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: DevTools (Accessibility tree), ekstensi screen reader bila tersedia.
- File awal: komponen interaktif sederhana (accordion/menu).

## B) Langkah Praktik
1. Pastikan komponen memakai elemen semantik yang tepat.
2. Tambahkan atribut ARIA hanya untuk state/relasi yang dibutuhkan.
3. Uji flow keyboard end-to-end tanpa mouse.
4. Uji pesan status dengan `aria-live`.

## C) Expected Output
- UI yang diharapkan: semua aksi utama dapat dioperasikan via keyboard.
- Output console/network yang diharapkan: tidak ada error JS; state ARIA konsisten.

## D) Troubleshooting
- Masalah umum 1 + solusi: tidak bisa difokuskan, cek elemen interaktif dan urutan tab.
- Masalah umum 2 + solusi: screen reader tidak umumkan status, cek region `aria-live`.
- Masalah umum 3 + solusi: state tidak sinkron, pastikan update DOM dan ARIA dalam satu fungsi.
