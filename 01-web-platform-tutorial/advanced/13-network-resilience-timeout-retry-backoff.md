# Network Resilience: Timeout, Retry, dan Backoff

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan level: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham `fetch`, `AbortController`, dan handling error async.
  - [ ] Paham state UI (`loading/success/error`) dan render terpusat.
- Kamus mini:
  - `[baru]` timeout: batas waktu request sebelum dianggap gagal.
  - `[baru]` retry: mengulang request saat kegagalan tertentu.
  - `[baru]` backoff: jeda bertahap antar percobaan retry.
  - `[baru]` transient error: error sementara yang sering bisa pulih saat dicoba ulang.

## 1) Pengantar Singkat Topik
Topik ini membahas pola ketahanan network agar aplikasi tetap responsif saat koneksi tidak stabil. Fokusnya adalah timeout yang tegas, retry terkontrol, dan jeda backoff agar sistem tidak membanjiri server.

## 2) Big Picture
Di produksi, kegagalan network adalah hal normal: timeout, putus sementara, atau server sibuk. Jika request gagal ditangani asal-asalan, user akan melihat loading tak berujung atau klik berulang tanpa hasil jelas. Dengan timeout + retry + backoff, aplikasi punya perilaku yang bisa diprediksi dan lebih ramah terhadap server. Dampaknya: UX lebih stabil, debugging lebih mudah, dan risiko spike traffic karena retry liar berkurang.

## 3) Small Picture
- Terapkan timeout per request, jangan menunggu tanpa batas.
- Retry hanya untuk error yang layak diulang (transient), bukan semua error.
- Tambahkan backoff bertahap dan batas percobaan.
- Pisahkan status `aborted`, `timeout`, dan `network/server error` di UI.

## 4) Wireframe Alur Konsep
```text
[klik load]
  -> [attempt #1 + timeout]
      -> [sukses] -> [render sukses]
      -> [gagal transient] -> [delay backoff] -> [attempt berikutnya]
      -> [gagal final] -> [render error]
```
- Alur utama: percobaan pertama sukses.
- Alur jalan: percobaan awal gagal sementara, retry berikutnya sukses.
- Alur error: semua percobaan habis atau error non-retryable langsung gagal.

## 5) Analogi Dunia Nyata
Seperti menghubungi call center saat jam sibuk: kamu menunggu batas waktu tertentu, lalu coba lagi setelah jeda. Jeda makin panjang agar tidak menambah kepadatan jalur.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: load data dashboard, sinkronisasi daftar, request API penting non-kritis real-time.
- Alasan pakai: mencegah UI hang, menstabilkan hasil di jaringan tidak menentu, mengurangi spam request.
- Kapan tidak dipakai: aksi non-idempotent sensitif (misal pembayaran) tanpa desain deduplikasi/idempotency key.

## 7) Contoh Sederhana + Bedah Output
```html
<button id="loadBtn">Load Profile</button>
<p id="statusText">Idle</p>
<pre id="resultText"></pre>

<script>
  const loadBtn = document.querySelector('#loadBtn');
  const statusText = document.querySelector('#statusText');
  const resultText = document.querySelector('#resultText');

  const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

  function isRetryableError(err) {
    return err.name === 'AbortError' || err.message.startsWith('HTTP 5');
  }

  async function fetchWithTimeout(url, timeoutMs) {
    const controller = new AbortController();
    const timer = setTimeout(() => controller.abort(), timeoutMs);
    try {
      const res = await fetch(url, { signal: controller.signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } finally {
      clearTimeout(timer);
    }
  }

  async function requestWithRetry(url, options) {
    const { retries, timeoutMs, backoffBaseMs } = options;
    let attempt = 0;

    while (attempt <= retries) {
      try {
        return await fetchWithTimeout(url, timeoutMs);
      } catch (err) {
        if (!isRetryableError(err) || attempt === retries) {
          throw err;
        }
        const delay = backoffBaseMs * Math.pow(2, attempt);
        await sleep(delay);
        attempt += 1;
      }
    }
  }

  loadBtn.addEventListener('click', async () => {
    statusText.textContent = 'Loading...';
    resultText.textContent = '';
    try {
      const data = await requestWithRetry(
        'https://jsonplaceholder.typicode.com/users/1',
        { retries: 2, timeoutMs: 3000, backoffBaseMs: 400 }
      );
      statusText.textContent = 'Success';
      resultText.textContent = JSON.stringify(data, null, 2);
    } catch (err) {
      if (err.name === 'AbortError') {
        statusText.textContent = 'Timeout';
      } else {
        statusText.textContent = `Failed: ${err.message}`;
      }
    }
  });
</script>
```
Bedah output:
1. Klik tombol menampilkan `Loading...` lalu sukses jika request lancar.
2. Jika percobaan awal gagal sementara, sistem retry dengan jeda backoff.
3. Jika timeout berulang atau error final, status menjadi `Timeout`/`Failed: ...`.

## 8) Jebakan Umum (Pitfalls)
- Retry semua error termasuk `4xx`, padahal umumnya bukan transient.
- Tidak memberi batas retry sehingga request loop tidak berujung.
- Timeout terlalu pendek/terlalu panjang tanpa data observasi.
- Tidak membedakan timeout vs error server di UI.

## 9) Prediksi Output Drill + Kunci Jawaban
Catatan:
- Ikuti aturan drill di `../README.md` section `Aturan Level (Ringkas)`.

Drill:
```js
const base = 200;
for (let attempt = 0; attempt < 3; attempt += 1) {
  console.log(base * Math.pow(2, attempt));
}
```
Kunci:
- Output:
  - `200`
  - `400`
  - `800`
- Alasan: backoff eksponensial menggandakan jeda tiap percobaan.

## 10) Debug Story
- Gejala: user lihat loading lama, lalu gagal tanpa kepastian status.
- Akar masalah: request tidak punya timeout dan retry tidak terukur.
- Langkah debug: log `attempt`, `delay`, waktu mulai/selesai request, dan jenis error.
- Solusi: terapkan timeout pasti, retry maksimal, dan klasifikasi error retryable.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membedakan error retryable vs non-retryable.
- [ ] Bisa implementasi timeout + retry + backoff dengan batas percobaan.
- [ ] Bisa menjelaskan trade-off retry terhadap beban server dan UX.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/04-fetch-advanced-loading-retry-abort.md`.
- Topik terkait: `advanced/14-cache-strategy-memory-localstorage.md`.
- Referensi tambahan: MDN Fetch API, AbortController, HTTP status code classes.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools (`Network`, `Console`).
- File awal: satu `index.html` dengan tombol load + status + output.

## B) Langkah Praktik
1. Implement `fetchWithTimeout()` dengan `AbortController`.
2. Bungkus dalam `requestWithRetry()` dengan `retries` dan backoff eksponensial.
3. Pisahkan handler UI untuk status `success`, `timeout`, dan `failed`.
4. Uji dengan throttling network di DevTools.

## C) Expected Output
- UI yang diharapkan: user melihat status final yang jelas, tidak loading tanpa akhir.
- Output console/network yang diharapkan: ada pola percobaan retry yang terbatas.

## D) Troubleshooting
- Masalah umum 1 + solusi: retry tidak jalan, cek kondisi `isRetryableError`.
- Masalah umum 2 + solusi: timeout tidak aktif, pastikan `fetch` memakai `signal` dari controller.
- Masalah umum 3 + solusi: delay tidak terasa, log nilai backoff per attempt.
