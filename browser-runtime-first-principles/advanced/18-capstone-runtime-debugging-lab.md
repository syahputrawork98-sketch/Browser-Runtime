# Capstone: Runtime Debugging Lab

## Meta
- Level: `Advanced`
- Estimasi durasi: `90-150 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Selesai modul `13` sampai `17`.
  - [ ] Paham scheduling, batching, backpressure, observability, dan testing ordering.
- Kamus mini:
  - `[ulang]` capstone: proyek integrasi akhir lintas modul.
  - `[baru]` runtime contract: aturan perilaku async yang harus tetap benar.
  - `[baru]` incident replay: merekonstruksi urutan kejadian dari trace/log.
  - `[baru]` stabilization loop: siklus ukur -> analisis -> perbaiki -> ukur ulang.

## 1) Pengantar Singkat Topik
Capstone ini menggabungkan semua prinsip runtime ke satu lab debugging end-to-end. Fokusnya bukan sekadar fitur jalan, tetapi sistem yang stabil, terukur, dan bisa diuji secara deterministik.

## 2) Big Picture
Di produksi, bug async jarang datang sendirian: biasanya kombinasi burst request, ordering yang salah, dan observability lemah. Capstone memaksa kamu menangani semua aspek itu sekaligus. Target utamanya adalah membangun pola kerja engineer runtime: mendeteksi gejala, membuktikan akar masalah, lalu memperbaiki dengan trade-off jelas. Output akhir harus bisa dijadikan playbook saat menghadapi insiden nyata.

## 3) Small Picture
- Bangun mini app dengan alur `input -> fetch -> apply -> render`.
- Tambahkan kontrol runtime: cancellation, concurrency limit, dan batching DOM update.
- Tambahkan observability: trace id, phase log, timing per fase.
- Buat test untuk ordering/timing kontrak kritis.
- Lakukan post-mortem ringkas: gejala, akar masalah, solusi, dan risiko sisa.

## 4) Wireframe Alur Konsep
```text
[user input burst]
  -> [debounce + cancel previous]
  -> [queue with concurrency limit]
  -> [network request + trace/timing]
  -> [last-write-wins guard]
  -> [batched DOM commit]
  -> [assert ordering in tests]
```
- Alur utama: interaksi tetap responsif dan hasil render konsisten.
- Alur jalan: request lama dibatalkan, request relevan diprioritaskan.
- Alur error: jika observability/test absen, regresi timing sulit dideteksi.

## 5) Analogi Dunia Nyata
Seperti pusat komando bandara: banyak kejadian paralel harus tetap tertib, terpantau, dan dapat diaudit ketika terjadi gangguan.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: validasi kesiapan debugging runtime tingkat lanjut.
- Alasan pakai: melatih integrasi konsep, bukan pemahaman parsial per modul.
- Kapan tidak dipakai: jika modul prasyarat belum stabil dikuasai.

## 7) Contoh Sederhana + Bedah Output
```js
const state = { query: '', items: [], loading: false, error: '' };
let activeToken = 0;

async function search(raw) {
  const token = ++activeToken;
  state.query = raw.trim();
  state.loading = true;
  state.error = '';
  const traceId = `t-${token}`;
  const t0 = performance.now();
  console.log(`[${traceId}] start query=${state.query}`);

  try {
    const data = await fakeApi(state.query, token);
    if (token !== activeToken) return; // last-write-wins guard

    state.items = data;
    console.log(`[${traceId}] apply items=${data.length}`);
  } catch (err) {
    if (token !== activeToken) return;
    state.error = err.message;
    console.log(`[${traceId}] error=${err.message}`);
  } finally {
    if (token !== activeToken) return;
    state.loading = false;
    console.log(`[${traceId}] done ${(performance.now() - t0).toFixed(1)}ms`);
    render();
  }
}
```
Bedah output:
1. Setiap request punya token + trace id untuk korelasi.
2. Hanya request terbaru yang boleh commit state (`last-write-wins`).
3. Timing akhir selalu tercatat pada jalur sukses maupun error.

## 8) Jebakan Umum (Pitfalls)
- Menyatukan semua concern di satu fungsi tanpa boundary (sulit dites).
- Tidak mendefinisikan runtime contract yang wajib dijaga.
- Memakai observability hanya saat error, bukan sejak awal desain.
- Menjalankan test async tanpa kontrol waktu sehingga flakey.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
let latest = 0;
function issue() { latest += 1; return latest; }
const a = issue();
const b = issue();
console.log(a === b ? 'same' : 'newer exists');
```
Kunci:
- Output: `newer exists`
- Alasan: token kedua menandakan request terbaru.

Drill 2:
```js
const seq = [];
Promise.resolve().then(() => seq.push('M'));
setTimeout(() => seq.push('T'), 0);
seq.push('S');
setTimeout(() => console.log(seq.join('')), 0);
```
Kunci:
- Output: `SMT`
- Alasan: sinkron dulu, lalu microtask, lalu task timeout.

Drill 3:
```js
const q = [];
q.push('r1');
q.push('r2');
q.shift();
console.log(q[0]);
```
Kunci:
- Output: `r2`
- Alasan: item pertama keluar dari antrean, item kedua jadi kepala antrean.

## 10) Debug Story
- Gejala: hasil pencarian sering melompat ke data lama saat user mengetik cepat.
- Akar masalah: request overlap tanpa guard commit state.
- Langkah debug: tambah trace id, log phase (`start`, `response`, `apply`), dan token request.
- Solusi: terapkan cancellation + last-write-wins guard + test ordering untuk mencegah regresi.

## 11) Checkpoint Kelulusan Topik
- [ ] Mampu mendefinisikan runtime contract untuk alur async utama.
- [ ] Mampu mengintegrasikan cancellation, backpressure, dan batching pada satu flow.
- [ ] Mampu menulis test ordering/timing yang deterministik.
- [ ] Mampu menyusun incident replay dari trace/log/timing.
- [ ] Mampu menjelaskan trade-off desain yang dipilih.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `advanced/13` sampai `advanced/17`.
- Topik terkait: `expert-bridge/README.md`.
- Referensi tambahan: async incident response checklist dan deterministic async testing patterns.

## 13) Model Mental
- Entitas utama: trigger event, scheduler, queue, token guard, observer, assertion.
- Urutan proses: trigger -> schedule -> execute async -> guard commit -> observe -> validate.
- Invariant (aturan yang selalu benar): hanya hasil yang masih relevan boleh mengubah state/UI.

## 14) Failure Case
- Skenario gagal: timeout/retry sudah ada, tapi hasil lama tetap overwrite hasil baru.
- Akar penyebab runtime: tidak ada guard relevansi saat commit state.
- Perbaikan: kombinasikan token guard, cancellation, dan assertion ordering di test.

## 15) Spec Hint
- Section spec terkait (ringkas): event loop task model, promise jobs, timer, dan fetch abort semantics.
- Kata kunci pencarian spec: `event loop`, `microtask checkpoint`, `setTimeout`, `AbortController`, `fetch`.
