# Capstone: Web Platform App

## Meta
- Level: `Advanced`
- Estimasi durasi: `90-150 menit`
- Aturan level: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Selesai modul `13` sampai `17`.
  - [ ] Paham state, async flow, cache, performance, accessibility, dan testing dasar.
- Kamus mini:
  - `[ulang]` capstone: proyek integrasi akhir lintas modul.
  - `[baru]` acceptance criteria: syarat minimal agar fitur dianggap selesai.
  - `[baru]` non-functional requirement: kebutuhan kualitas (performance, a11y, reliability).

## 1) Pengantar Singkat Topik
Capstone ini menggabungkan seluruh kompetensi track ke satu aplikasi browser yang siap dipakai sebagai baseline production mindset. Fokusnya bukan fitur paling banyak, tapi kualitas alur end-to-end yang stabil.

## 2) Big Picture
Proyek kecil sering terasa bagus sampai menghadapi kondisi nyata: network lambat, data usang, interaksi cepat, dan akses keyboard. Capstone memaksa semua trade-off itu ditangani sekaligus. Dengan ini, kamu melatih cara berpikir engineer: bukan hanya "fitur jalan", tetapi "fitur tahan gangguan, terukur, dan bisa diuji". Hasil akhirnya harus bisa dijadikan portofolio teknis.

## 3) Small Picture
- Rancang arsitektur sederhana: `state -> action -> render`.
- Terapkan fetch tahan gangguan (timeout, retry terbatas).
- Tambahkan cache memory + `localStorage` dengan TTL.
- Jaga performa interaksi (`debounce`/`throttle`) untuk event padat.
- Pastikan akses keyboard + ARIA untuk komponen kunci.
- Tulis test dasar untuk alur data dan error path.

## 4) Wireframe Alur Konsep
```text
[app start]
  -> [load cache + parse URL state]
  -> [render awal]
  -> [fetch network resilient]
  -> [render success/empty/error]
  -> [user interaksi: search/filter/pagination]
  -> [sync URL + cache + re-render]
```
- Alur utama: app cepat tampil, data sinkron, interaksi halus.
- Alur jalan: network gagal sementara, app fallback ke cache.
- Alur error: semua sumber gagal, app tetap tampilkan error state yang dapat dipahami.

## 5) Analogi Dunia Nyata
Seperti membangun ruang kontrol operasional: data harus cepat terlihat, tetap berguna saat koneksi terganggu, bisa dioperasikan semua staf, dan punya checklist uji rutin.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: latihan final track dan artefak portofolio.
- Alasan pakai: mengukur kesiapan implementasi end-to-end, bukan konsep terpisah.
- Kapan tidak dipakai: jika modul prasyarat belum stabil dikuasai.

## 7) Contoh Sederhana + Bedah Output
```js
// pseudo architecture inti capstone
const state = {
  query: '',
  page: 1,
  items: [],
  view: { loading: false, error: '', source: 'network' }
};

async function bootstrap() {
  hydrateFromUrl();
  hydrateFromCache();
  render();
  await reloadData({ allowCache: true });
}

async function reloadData({ allowCache }) {
  state.view.loading = true;
  state.view.error = '';
  render();

  try {
    const data = await loadItemsResilient({ allowCache });
    state.items = data.items;
    state.view.source = data.source; // memory/localStorage/network
  } catch (err) {
    state.view.error = err.message;
  } finally {
    state.view.loading = false;
    render();
  }
}

function onSearchInput(raw) {
  state.query = raw.trim();
  state.page = 1;
  syncUrl();
  render();
}
```
Bedah output:
1. App punya start flow yang deterministik: hydrate -> render -> reload.
2. UI menampilkan sumber data dan status (`loading/error`) secara eksplisit.
3. Interaksi user langsung sinkron ke URL dan state tanpa chaos.

## 8) Jebakan Umum (Pitfalls)
- Menggabungkan semua logika di satu file besar tanpa batas tanggung jawab.
- Tidak punya acceptance criteria, sehingga "selesai" jadi subjektif.
- Fokus ke tampilan, tapi jalur error/loading/a11y tidak tuntas.
- Menulis test hanya untuk happy path.

## 9) Prediksi Output Drill + Kunci Jawaban
Catatan:
- Ikuti aturan drill di `../README.md` section `Aturan Level (Ringkas)`.

Drill:
```js
const state = { loading: false, error: '' };
async function run() {
  state.loading = true;
  try {
    throw new Error('boom');
  } catch (e) {
    state.error = e.message;
  } finally {
    state.loading = false;
  }
  console.log(state.loading, state.error);
}
run();
```
Kunci:
- Output: `false boom`
- Alasan: `finally` selalu menutup loading, `catch` menyimpan pesan error.

## 10) Debug Story
- Gejala: user melaporkan app kadang kosong tanpa pesan saat jaringan lambat.
- Akar masalah: fallback cache gagal, tapi error state tidak dirender.
- Langkah debug: tracing state transition tiap action (`loading/error/source/items`).
- Solusi: pastikan semua cabang async menulis state final dan punya UI message yang jelas.

## 11) Checkpoint Kelulusan Topik
- [ ] Punya aplikasi dengan flow lengkap: loading, success, empty, error.
- [ ] Punya minimal satu fitur performa (debounce/throttle) yang terukur.
- [ ] Punya keyboard flow dan ARIA dasar yang tervalidasi.
- [ ] Punya test dasar untuk jalur success + error.
- [ ] Bisa jelaskan trade-off arsitektur yang dipilih.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `advanced/13` sampai `advanced/17`.
- Topik terkait: `expert-bridge/README.md`.
- Referensi tambahan: checklist release frontend (reliability, performance, accessibility, testing).

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox modern.
- Tools: DevTools (Network, Performance, Accessibility).
- Opsional: test runner (Vitest/Jest) untuk validasi otomatis.
- Struktur minimal:
  - `index.html`
  - `app.js`
  - `state.js`
  - `api.js`
  - `cache.js`
  - `a11y.js`
  - `tests/`

## B) Langkah Praktik
1. Definisikan acceptance criteria (fungsional + non-fungsional).
2. Implement bootstrap app + state management sederhana.
3. Implement API flow resilient (timeout/retry) + cache strategy.
4. Tambahkan URL state untuk search/filter/page.
5. Optimasi event padat dengan debounce/throttle.
6. Audit keyboard + ARIA komponen utama.
7. Tulis test untuk unit data flow dan async error flow.

## C) Expected Output
- UI yang diharapkan: aplikasi tetap usable pada kondisi network normal maupun terganggu.
- Output console/network yang diharapkan: state transition jelas, request tidak berlebihan, error tertangani.
- Artefak akhir: README mini proyek berisi scope, trade-off, dan checklist kualitas.

## D) Troubleshooting
- Masalah umum 1 + solusi: source data tidak jelas, tampilkan indikator `source` di UI.
- Masalah umum 2 + solusi: hasil pencarian stale, reset page dan sinkronkan URL saat query berubah.
- Masalah umum 3 + solusi: jank saat scroll/filter, audit handler dan kurangi kerja per frame.
- Masalah umum 4 + solusi: sulit dites, pisahkan pure function dari DOM/network side effect.
