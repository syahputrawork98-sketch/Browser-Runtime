# Mini Project: Async Workflow Inspector

## Meta
- Level: `Intermediate`
- Estimasi durasi: `75-120 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan modul `07` sampai `11`.
  - [ ] Paham cancellation, LWW guard, dan cleanup side effects.
- Kamus mini:
  - `[ulang]` workflow trace: jejak urutan aksi async dari start sampai final apply.
  - `[baru]` in-flight request: request yang sedang berjalan dan belum selesai.
  - `[baru]` inspector panel: panel observasi status runtime (queue/effect/state) secara eksplisit.

## 1) Pengantar Singkat Topik
Mini project ini menggabungkan semua skill Intermediate untuk menginspeksi alur async secara nyata. Tujuannya: kamu bisa melihat, bukan menebak, bagaimana request overlap, cancel, dan state guard bekerja.

## 2) Big Picture
Di produksi, masalah async jarang berdiri sendiri: ada race, cancellation, stale state, dan leak dalam satu fitur. Proyek ini melatih satu pola yang tahan gangguan: traceable, cancel-safe, dan state-consistent. Hasil akhirnya adalah playground yang bisa dijadikan alat debugging saat investigasi bug runtime.

## 3) Small Picture
- Bangun state terpusat (`query`, `status`, `items`, `latestRequestId`, `logs`).
- Tiap request punya request id + optional abort controller.
- Terapkan LWW guard saat respons datang.
- Sediakan panel log runtime (`start`, `abort`, `apply`, `ignore`, `error`).
- Pastikan cleanup listener/timer dilakukan saat inspector di-unmount.

## 4) Wireframe Alur Konsep
```text
[user input]
  -> [start request #n]
  -> [abort request lama]
  -> [loading render]
  -> [response datang]
      -> [if stale: ignore]
      -> [if latest: apply + render]
  -> [log semua transisi]
```
- Alur utama: hanya request terbaru yang boleh update UI.
- Alur jalan: request lama dibatalkan, log menunjukkan alasan.
- Alur error: stale response ter-apply karena tidak ada guard.

## 5) Analogi Dunia Nyata
Seperti ruang kontrol pengiriman: pesanan terbaru punya prioritas, pesanan lama dibatalkan, dan setiap perubahan dicatat agar audit mudah.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: simulator debug async flow sebelum implementasi ke fitur production.
- Alasan pakai: mengurangi trial-and-error saat menangani bug timing.
- Kapan tidak dipakai: aplikasi statis tanpa interaksi async dinamis.

## 7) Contoh Sederhana + Bedah Output
```html
<input id="q" placeholder="Type to search..." />
<button id="resetBtn">Reset Logs</button>
<p id="status"></p>
<ul id="list"></ul>
<pre id="logs"></pre>

<script>
  const state = {
    query: '',
    status: 'idle',
    items: [],
    latestRequestId: 0,
    controller: null,
    logs: []
  };

  const elQ = document.querySelector('#q');
  const elStatus = document.querySelector('#status');
  const elList = document.querySelector('#list');
  const elLogs = document.querySelector('#logs');
  const resetBtn = document.querySelector('#resetBtn');

  function log(msg) {
    state.logs.push(msg);
    elLogs.textContent = state.logs.join('\n');
  }

  function render() {
    elStatus.textContent = `status=${state.status}, request=${state.latestRequestId}, q=${state.query}`;
    elList.innerHTML = state.items.slice(0, 5).map((x) => `<li>${x.title || x}</li>`).join('');
  }

  async function search(query) {
    state.query = query;
    const requestId = ++state.latestRequestId;
    state.status = 'loading';
    render();
    log(`start #${requestId} q="${query}"`);

    if (state.controller) {
      state.controller.abort();
      log(`abort previous request`);
    }
    state.controller = new AbortController();

    try {
      const res = await fetch(
        `https://jsonplaceholder.typicode.com/posts?title_like=${encodeURIComponent(query)}`,
        { signal: state.controller.signal }
      );
      const data = await res.json();

      if (requestId !== state.latestRequestId) {
        log(`ignore stale #${requestId}`);
        return;
      }

      state.items = data;
      state.status = 'success';
      render();
      log(`apply #${requestId} count=${data.length}`);
    } catch (err) {
      if (err.name === 'AbortError') {
        log(`aborted #${requestId}`);
        return;
      }
      if (requestId !== state.latestRequestId) return;
      state.status = 'error';
      render();
      log(`error #${requestId} ${err.message}`);
    }
  }

  elQ.addEventListener('input', (e) => {
    search(e.target.value.trim());
  });

  resetBtn.addEventListener('click', () => {
    state.logs = [];
    elLogs.textContent = '';
  });

  render();
</script>
```
Bedah output:
1. Input cepat memicu request beruntun dengan id naik.
2. Request lama dibatalkan dan tercatat di log.
3. Respons stale diabaikan, hanya latest request di-apply.
4. Panel status/list/log selalu sinkron terhadap state terbaru.

## 8) Jebakan Umum (Pitfalls)
- Menggunakan LWW guard tapi lupa update `latestRequestId`.
- Membatalkan request tanpa membedakan abort error vs network error.
- Menambah listener berkali-kali saat re-mount inspector tanpa cleanup.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
let latest = 2;
const incoming = 1;
console.log(incoming === latest ? 'apply' : 'ignore');
```
Kunci:
- Output: `ignore`
- Alasan: request lama tidak boleh override state terbaru.

Drill 2:
```js
const c = new AbortController();
c.abort();
console.log(c.signal.aborted ? 'aborted' : 'running');
```
Kunci:
- Output: `aborted`
- Alasan: signal berubah menjadi aborted setelah `abort()`.

Drill 3:
```js
const logs = [];
function log(x) { logs.push(x); }
log('start #1');
log('abort #1');
log('start #2');
console.log(logs.join(' -> '));
```
Kunci:
- Output: `start #1 -> abort #1 -> start #2`
- Alasan: trace membantu audit urutan side effect secara deterministik.

## 10) Debug Story
- Gejala: list hasil kadang meloncat-loncat saat user mengetik cepat.
- Akar masalah: tidak ada guard stale response dan request lama tetap apply.
- Langkah debug: cek log `start/abort/apply/ignore` dengan request id.
- Solusi: terapkan kombinasi cancellation + LWW guard + source of truth tunggal.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membangun inspector panel yang menunjukkan state + log runtime.
- [ ] Bisa menerapkan cancellation dan last-write-wins secara bersamaan.
- [ ] Bisa menunjukkan cleanup side effects (listener/timer/controller) saat unmount.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `intermediate/08`, `intermediate/09`, `intermediate/10`, `intermediate/11`.
- Topik terkait: `advanced/13-scheduling-prioritas-dan-trade-off.md`.
- Referensi tambahan: DevTools network panel dan performance/memory panel.

## 13) Model Mental
- Entitas utama: input action, request id, abort signal, state commit, render, trace logs.
- Urutan proses: trigger -> assign id -> cancel lama -> fetch -> guard -> apply/ignore -> trace.
- Invariant (aturan yang selalu benar): hasil async tanpa identitas request tidak aman untuk langsung apply.

## 14) Failure Case
- Skenario gagal: inspector menampilkan `success`, tetapi data berasal dari request lama yang selesai terlambat.
- Akar penyebab runtime: tidak ada validasi id request saat commit state.
- Perbaikan: gunakan request id monotonic + guard `if (id !== latest) ignore`.

## 15) Spec Hint
- Section spec terkait (ringkas): fetch completion, abort signal handling, dan ordering di event loop.
- Kata kunci pencarian spec: `fetch`, `AbortSignal`, `event loop`, `promise resolution`.
