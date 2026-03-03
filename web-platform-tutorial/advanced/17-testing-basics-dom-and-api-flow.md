# Testing Basics: DOM dan API Flow

## Meta
- Level: `Advanced`
- Estimasi durasi: `60-90 menit`
- Aturan level: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham state-render dan fetch async.
  - [ ] Paham skenario UI `loading`, `success`, dan `error`.
- Kamus mini:
  - `[baru]` test case: skenario terstruktur untuk memvalidasi perilaku.
  - `[baru]` assertion: pengecekan ekspektasi hasil.
  - `[baru]` mocking: mengganti dependensi eksternal dengan versi palsu untuk test.
  - `[baru]` deterministic test: test yang hasilnya konsisten di setiap run.

## 1) Pengantar Singkat Topik
Topik ini membahas pondasi testing frontend untuk alur DOM dan API. Fokusnya bukan framework tertentu, tetapi pola pikir testable: memisahkan logika, menyiapkan mock data, dan menguji jalur sukses maupun gagal.

## 2) Big Picture
Bug frontend sering lolos karena kita hanya uji manual pada "jalur normal". Testing dasar membantu memastikan state UI tetap benar saat API lambat, gagal, atau data kosong. Dengan test yang kecil namun tepat sasaran, refactor jadi lebih aman. Setelah modul ini, kamu bisa menulis test minimal untuk perilaku penting tanpa tergantung klik manual berulang.

## 3) Small Picture
- Pisahkan logika pure function dari side effect DOM/network.
- Uji transformasi data sebagai unit test ringan.
- Uji alur async: status loading -> success/error.
- Gunakan mock fetch agar test tidak bergantung jaringan real.

## 4) Wireframe Alur Konsep
```text
[trigger load]
  -> [set loading]
  -> [fetch (mock)]
      -> [success -> render data]
      -> [error -> render message]
```
- Alur utama: API sukses, UI menampilkan data.
- Alur jalan: API kosong, UI menampilkan empty state.
- Alur error: API gagal, UI menampilkan pesan error.

## 5) Analogi Dunia Nyata
Seperti simulasi kebakaran gedung: bukan menunggu kebakaran sungguhan, tapi menguji prosedur dengan skenario buatan agar tahu sistem benar-benar siap.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: fitur list dari API, form submit async, dashboard status.
- Alasan pakai: menurunkan regresi saat perubahan kode.
- Kapan tidak dipakai: hampir tidak ada; minimal test untuk flow kritis tetap disarankan.

## 7) Contoh Sederhana + Bedah Output
```js
// app.js
export function toViewModel(users) {
  return users.map((u) => ({ id: u.id, label: `${u.id} - ${u.name}` }));
}

export async function loadUsers(fetchImpl) {
  const res = await fetchImpl('/api/users');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const data = await res.json();
  return toViewModel(data);
}

// app.test.js (pseudo-runner sederhana)
function assertEqual(actual, expected, msg) {
  if (JSON.stringify(actual) !== JSON.stringify(expected)) {
    throw new Error(`FAIL: ${msg}`);
  }
  console.log(`PASS: ${msg}`);
}

async function runTests() {
  // Unit test pure function
  const vm = toViewModel([{ id: 1, name: 'Ana' }]);
  assertEqual(vm, [{ id: 1, label: '1 - Ana' }], 'toViewModel membentuk label');

  // Async success test dengan mock fetch
  const okFetch = async () => ({
    ok: true,
    status: 200,
    json: async () => [{ id: 2, name: 'Budi' }]
  });
  const users = await loadUsers(okFetch);
  assertEqual(users[0].label, '2 - Budi', 'loadUsers success flow');

  // Async error test dengan mock fetch gagal
  const badFetch = async () => ({ ok: false, status: 500, json: async () => [] });
  let errorHit = false;
  try {
    await loadUsers(badFetch);
  } catch (err) {
    errorHit = err.message === 'HTTP 500';
  }
  assertEqual(errorHit, true, 'loadUsers error flow');
}

runTests();
```
Bedah output:
1. Test pertama memvalidasi transformasi data tanpa DOM/network.
2. Test kedua memvalidasi alur sukses API dengan mock.
3. Test ketiga memvalidasi alur error API dan pesan error yang tepat.

## 8) Jebakan Umum (Pitfalls)
- Test masih memanggil API real sehingga flaky.
- Satu test mencakup terlalu banyak skenario sekaligus.
- Tidak menguji jalur error/empty state.
- Assertion lemah (misalnya hanya cek "tidak null").

## 9) Prediksi Output Drill + Kunci Jawaban
Catatan:
- Ikuti aturan drill di `../README.md` section `Aturan Level (Ringkas)`.

Drill:
```js
let called = 0;
const mockFetch = async () => {
  called += 1;
  return { ok: true, status: 200, json: async () => [] };
};

await mockFetch();
await mockFetch();
console.log(called);
```
Kunci:
- Output: `2`
- Alasan: setiap pemanggilan mock menaikkan counter satu kali.

## 10) Debug Story
- Gejala: test kadang lulus, kadang gagal tanpa perubahan kode.
- Akar masalah: test memakai API real sehingga tergantung jaringan dan data dinamis.
- Langkah debug: log sumber data test, isolasi call eksternal, cek ada/tidak race async.
- Solusi: gunakan mock fetch, data fixture tetap, dan assertion spesifik.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menulis test untuk transformasi data (pure function).
- [ ] Bisa menulis test async success/error dengan mock fetch.
- [ ] Bisa menjelaskan kenapa deterministic test penting untuk CI.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: `advanced/13-network-resilience-timeout-retry-backoff.md`.
- Topik terkait: `advanced/18-capstone-web-platform-app.md`.
- Referensi tambahan: konsep unit test, mock/stub, dan test pyramid dasar.

## A) Setup dan Lingkungan
- Browser target: modern browser untuk demo manual.
- Tools: Node.js + test runner pilihan (Vitest/Jest) atau runner sederhana manual.
- File awal: modul kecil berisi fungsi transform + fungsi fetch wrapper.

## B) Langkah Praktik
1. Pisahkan fungsi transformasi data (`pure`) dari fungsi network wrapper.
2. Tulis test unit untuk fungsi transform.
3. Tulis test async untuk jalur sukses dan error menggunakan mock fetch.
4. Tambah satu test empty state (`[]`) sesuai perilaku UI yang diinginkan.

## C) Expected Output
- UI yang diharapkan: status loading/success/error konsisten dengan hasil API.
- Output console/network yang diharapkan: test pass konsisten tanpa call API real.

## D) Troubleshooting
- Masalah umum 1 + solusi: promise tidak di-await, pastikan test async memakai `await`.
- Masalah umum 2 + solusi: mock tidak terpakai, pastikan dependency fetch di-inject.
- Masalah umum 3 + solusi: assertion terlalu umum, cek field spesifik yang kritis.
