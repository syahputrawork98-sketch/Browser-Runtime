# Runtime Side Effects dan State Consistency

## Meta
- Level: `Intermediate`
- Estimasi durasi: `45-60 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham race condition dan last write wins.
  - [ ] Paham state update dasar di UI browser.
- Kamus mini:
  - `[baru]` side effect: aksi yang mengubah dunia luar function (DOM, network, storage, timer).
  - `[baru]` state consistency: kondisi saat state internal, UI, dan sumber data tetap selaras.
  - `[baru]` source of truth: sumber kebenaran utama yang menjadi referensi update.
  - `[ulang]` stale state: state lama yang masih dipakai saat seharusnya sudah invalid.

## 1) Pengantar Singkat Topik
Topik ini membahas bagaimana side effect async bisa merusak konsistensi state jika tidak diatur. Fokusnya adalah membuat alur update yang deterministik dan mudah diaudit.

## 2) Big Picture
Dalam aplikasi nyata, satu aksi user bisa memicu banyak side effect sekaligus: network call, update DOM, save storage, log analytics. Jika urutannya tidak jelas, state mudah pecah: UI menampilkan A tapi storage berisi B. Dengan pola konsisten (`update state -> render -> side effect terkontrol`), kita menurunkan bug sinkronisasi dan mempermudah debugging.

## 3) Small Picture
- Tetapkan satu source of truth (object state terpusat).
- Pisahkan pure state transition dari side effect.
- Jalankan side effect berdasarkan state terbaru, bukan snapshot lama.
- Gunakan guard terhadap response lama atau effect yang sudah tidak relevan.

## 4) Wireframe Alur Konsep
```text
[user action]
  -> [compute next state]
  -> [commit state]
  -> [render]
  -> [run side effects terkontrol]
```
- Alur utama: state dan UI selalu sinkron.
- Alur jalan: side effect gagal tidak merusak state inti.
- Alur error: side effect lama menimpa state baru.

## 5) Analogi Dunia Nyata
Seperti sistem gudang: stok resmi harus diupdate dulu di sistem pusat, baru label rak, laporan, dan notifikasi mengikuti data pusat itu.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: form submit async, dashboard realtime ringan, sinkronisasi UI-storage-network.
- Alasan pakai: mencegah mismatch antar lapisan state.
- Kapan tidak dipakai: script sangat kecil sekali-jalan tanpa state berkelanjutan.

## 7) Contoh Sederhana + Bedah Output
```js
const state = {
  query: '',
  items: [],
  status: 'idle',
  requestId: 0
};

function render() {
  console.log('render:', state.status, state.items.length, state.query);
}

async function search(query) {
  state.query = query;
  state.status = 'loading';
  const requestId = ++state.requestId;
  render();

  try {
    const res = await fetch(
      `https://jsonplaceholder.typicode.com/posts?title_like=${encodeURIComponent(query)}`
    );
    const data = await res.json();

    // Guard side effect lama
    if (requestId !== state.requestId) return;

    state.items = data;
    state.status = 'success';
    render();
  } catch (err) {
    if (requestId !== state.requestId) return;
    state.status = 'error';
    render();
  }
}
```
Bedah output:
1. Saat action dimulai, state loading di-commit lalu render.
2. Respons hanya boleh apply jika request masih aktif.
3. UI selalu merefleksikan state terakhir yang valid.

## 8) Jebakan Umum (Pitfalls)
- Mengubah DOM langsung di banyak tempat tanpa lewat state tunggal.
- Menjalankan side effect berdasarkan closure state lama.
- Menulis storage/network dulu, baru commit state, sehingga data silang tidak sinkron.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
const state = { n: 1 };
function inc() {
  state.n += 1;
}
inc();
console.log(state.n);
```
Kunci:
- Output: `2`
- Alasan: source of truth tunggal di-update sekali.

Drill 2:
```js
let latest = 1;
function apply(id) {
  if (id !== latest) return 'ignore';
  return 'apply';
}
console.log(apply(0));
console.log(apply(1));
```
Kunci:
- Output:
  - `ignore`
  - `apply`
- Alasan: hanya side effect dengan id terbaru yang valid.

Drill 3:
```js
const state = { status: 'idle' };
state.status = 'loading';
Promise.resolve().then(() => {
  state.status = 'success';
  console.log(state.status);
});
console.log(state.status);
```
Kunci:
- Output:
  - `loading`
  - `success`
- Alasan: log sync dulu, update di Promise job setelah microtask drain.

## 10) Debug Story
- Gejala: indikator status menunjukkan `success`, tapi list item masih data lama.
- Akar masalah: state item di-update dari respons stale setelah status terbaru di-commit.
- Langkah debug: log `requestId`, `status`, dan jumlah item setiap commit render.
- Solusi: gunakan guard request aktif dan satu jalur commit state sebelum render.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menjelaskan bahaya side effect tanpa source of truth.
- [ ] Bisa menerapkan guard untuk mencegah stale effect apply.
- [ ] Bisa menjaga urutan `state commit -> render -> side effect`.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `intermediate/09-race-condition-dan-last-write-wins.md`.
- Topik terkait: `intermediate/11-memory-leak-dasar-listener-timer-closure.md`.
- Referensi tambahan: state management basics dan deterministic UI updates.

## 13) Model Mental
- Entitas utama: source of truth, action, commit state, side effect, render.
- Urutan proses: action -> hitung next state -> commit -> render -> side effect bersyarat.
- Invariant (aturan yang selalu benar): side effect yang tidak merepresentasikan state terbaru harus diabaikan.

## 14) Failure Case
- Skenario gagal: callback async lama menulis `localStorage` dan DOM setelah user pindah konteks.
- Akar penyebab runtime: side effect tidak divalidasi terhadap state terkini.
- Perbaikan: simpan version/token state aktif dan validasi sebelum menjalankan side effect.

## 15) Spec Hint
- Section spec terkait (ringkas): urutan penyelesaian Promise/fetch tidak menjamin sinkron dengan urutan aksi user.
- Kata kunci pencarian spec: `promise reaction`, `fetch completion`, `event loop`.
