# Cache Strategy: Memory dan localStorage

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan level: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham state-render, fetch, dan error handling.
  - [ ] Paham `localStorage` dasar dari modul Beginner.
- Kamus mini:
  - `[baru]` cache: penyimpanan sementara agar data bisa dipakai ulang.
  - `[baru]` memory cache: cache di memori runtime (hilang saat refresh).
  - `[baru]` persistent cache: cache yang bertahan antar refresh (localStorage).
  - `[baru]` TTL (time to live): umur maksimal data cache sebelum dianggap usang.
  - `[baru]` cache invalidation: aturan kapan data cache harus dihapus/diperbarui.

## 1) Pengantar Singkat Topik
Topik ini membahas strategi cache sederhana di browser dengan kombinasi memory dan `localStorage`. Tujuannya adalah respons UI lebih cepat tanpa mengorbankan konsistensi data.

## 2) Big Picture
Tanpa cache, app selalu menunggu network walau data sama sering diminta ulang. Cache membantu menurunkan latency dan mengurangi beban request, tapi ada trade-off: data bisa usang jika invalidation buruk. Strategi hybrid memory + `localStorage` biasanya cukup untuk app kecil-menengah. Setelah modul ini, kamu bisa memilih kapan baca dari cache, kapan refresh dari server, dan bagaimana menandai data expired.

## 3) Small Picture
- Cek memory cache dulu untuk respons tercepat.
- Jika miss, cek `localStorage` + validasi TTL.
- Jika cache tidak valid, fetch dari network lalu update kedua cache.
- Sediakan kebijakan invalidasi manual (misalnya tombol refresh paksa).

## 4) Wireframe Alur Konsep
```text
[request data]
  -> [cek memory]
      -> [hit] return cepat
      -> [miss] cek localStorage + TTL
          -> [valid] return + sinkronkan memory
          -> [invalid] fetch network -> simpan cache -> return
```
- Alur utama: memory hit, UI langsung responsif.
- Alur jalan: memory miss, localStorage hit valid.
- Alur error: cache expired + network gagal, tampilkan fallback error.

## 5) Analogi Dunia Nyata
Memory cache seperti catatan di meja kerja (paling cepat diakses), sedangkan `localStorage` seperti map arsip di lemari (lebih lambat tapi tetap ada saat kamu kembali bekerja besok).

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: data referensi, daftar produk, profil user non-sensitif, konfigurasi UI.
- Alasan pakai: mengurangi waktu tunggu dan request berulang.
- Kapan tidak dipakai: data sangat sensitif atau berubah per detik dan harus selalu real-time.

## 7) Contoh Sederhana + Bedah Output
```html
<button id="loadBtn">Load Posts</button>
<button id="refreshBtn">Force Refresh</button>
<p id="statusText">Idle</p>
<pre id="out"></pre>

<script>
  const CACHE_KEY = 'posts-cache-v1';
  const CACHE_TTL_MS = 60 * 1000; // 1 menit
  const memoryCache = new Map();

  const loadBtn = document.querySelector('#loadBtn');
  const refreshBtn = document.querySelector('#refreshBtn');
  const statusText = document.querySelector('#statusText');
  const out = document.querySelector('#out');

  function readLocalCache(key) {
    const raw = localStorage.getItem(key);
    if (!raw) return null;
    try {
      const parsed = JSON.parse(raw);
      const isExpired = Date.now() - parsed.savedAt > CACHE_TTL_MS;
      if (isExpired) return null;
      return parsed.data;
    } catch (_) {
      return null;
    }
  }

  function writeLocalCache(key, data) {
    const payload = { savedAt: Date.now(), data };
    localStorage.setItem(key, JSON.stringify(payload));
  }

  async function loadPosts(forceRefresh = false) {
    const cacheKey = CACHE_KEY;

    if (!forceRefresh && memoryCache.has(cacheKey)) {
      statusText.textContent = 'Loaded from memory cache';
      return memoryCache.get(cacheKey);
    }

    if (!forceRefresh) {
      const local = readLocalCache(cacheKey);
      if (local) {
        memoryCache.set(cacheKey, local);
        statusText.textContent = 'Loaded from localStorage cache';
        return local;
      }
    }

    statusText.textContent = 'Fetching from network...';
    const res = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=3');
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();

    memoryCache.set(cacheKey, data);
    writeLocalCache(cacheKey, data);
    statusText.textContent = 'Loaded from network';
    return data;
  }

  async function render(forceRefresh = false) {
    out.textContent = '';
    try {
      const posts = await loadPosts(forceRefresh);
      out.textContent = JSON.stringify(posts, null, 2);
    } catch (err) {
      statusText.textContent = `Failed: ${err.message}`;
    }
  }

  loadBtn.addEventListener('click', () => render(false));
  refreshBtn.addEventListener('click', () => render(true));
</script>
```
Bedah output:
1. Klik awal biasanya `Loaded from network`, data disimpan ke memory + `localStorage`.
2. Klik berikutnya `Loaded from memory cache`, respons paling cepat.
3. Setelah refresh halaman, hit biasanya dari `localStorage` (jika TTL belum habis).
4. `Force Refresh` selalu fetch ulang dan menimpa cache.

## 8) Jebakan Umum (Pitfalls)
- Menyimpan cache tanpa TTL sehingga data lama tampil terlalu lama.
- Tidak menandai versi key cache (`v1`/`v2`) saat format data berubah.
- Meng-cache data sensitif di `localStorage`.
- Lupa invalidasi saat user logout/pergantian akun.

## 9) Prediksi Output Drill + Kunci Jawaban
Catatan:
- Ikuti aturan drill di `../README.md` section `Aturan Level (Ringkas)`.

Drill:
```js
const now = 100000;
const savedAt = 99900;
const ttl = 50;
console.log(now - savedAt > ttl ? 'expired' : 'valid');
```
Kunci:
- Output: `expired`
- Alasan: umur cache `100` lebih besar dari TTL `50`.

## 10) Debug Story
- Gejala: user melihat data lama walau server sudah berubah.
- Akar masalah: TTL terlalu panjang dan force refresh tidak pernah dipakai.
- Langkah debug: log sumber data (`memory/local/network`) dan timestamp `savedAt`.
- Solusi: pendekkan TTL, tambah tombol refresh paksa, dan invalidasi saat event penting.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan perbedaan memory cache vs `localStorage` cache.
- [ ] Bisa implementasi TTL sederhana dan fallback network.
- [ ] Bisa menjelaskan trade-off kecepatan vs kesegaran data.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `beginner/05-local-storage-dan-state-persistence.md`.
- Topik terkait: `advanced/13-network-resilience-timeout-retry-backoff.md`.
- Referensi tambahan: MDN Web Storage API dan konsep caching dasar.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: DevTools (`Application` untuk localStorage, `Network` untuk request).
- File awal: satu `index.html` berisi dua tombol + status + output.

## B) Langkah Praktik
1. Implement memory cache dengan `Map`.
2. Implement baca/tulis `localStorage` dengan payload `{ savedAt, data }`.
3. Tambahkan TTL check sebelum memakai cache.
4. Tambahkan `forceRefresh` untuk bypass cache.

## C) Expected Output
- UI yang diharapkan: status jelas menunjukkan sumber data (`memory`, `localStorage`, `network`).
- Output console/network yang diharapkan: request berkurang setelah cache hit.

## D) Troubleshooting
- Masalah umum 1 + solusi: selalu network hit, cek key cache dan logika TTL.
- Masalah umum 2 + solusi: parse error localStorage, bungkus `JSON.parse` dengan `try/catch`.
- Masalah umum 3 + solusi: data beda akun tercampur, gunakan key cache per user.
