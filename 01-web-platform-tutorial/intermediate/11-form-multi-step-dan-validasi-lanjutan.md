# Form Multi-Step dan Validasi Lanjutan

## Meta
- Level: `Intermediate`
- Estimasi durasi: `55-75 menit`
- Status drill: `wajib`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Paham validasi form dasar.
  - [ ] Paham state render dasar dan event submit.
- Kamus mini:
  - `[baru]` multi-step form: form dibagi beberapa langkah.
  - `[baru]` step validation: validasi per langkah sebelum lanjut.
  - `[baru]` final validation: validasi keseluruhan sebelum submit final.

## 1) Pengantar Singkat Topik
Topik ini membahas bagaimana mengelola form panjang dengan beberapa langkah agar input lebih terarah dan validasi lebih terkontrol.

## 2) Big Picture
Form panjang dalam satu layar cenderung membuat user kewalahan dan error tinggi. Multi-step form memecah beban input, sementara validasi bertahap mencegah error menumpuk di akhir. Hasilnya UX lebih baik dan data yang masuk lebih bersih.

## 3) Small Picture
- Simpan `currentStep` dan nilai field di state.
- Render hanya step aktif.
- Saat tombol next diklik, jalankan validasi step aktif dulu.
- Sebelum submit final, jalankan validasi seluruh data.

## 4) Wireframe Alur Konsep
```text
[step 1 input] -> [validasi step 1] -> [step 2 input] -> [validasi step 2] -> [submit final]
```
- Alur utama: tiap step valid -> lanjut -> submit berhasil.
- Alur jalan: jika invalid, tetap di step saat ini dan tampilkan error.
- Alur error: user bisa lompat step tanpa validasi sehingga data rusak.

## 5) Analogi Dunia Nyata
Seperti proses check-in bandara: isi identitas dulu, lalu data bagasi, lalu konfirmasi akhir. Tiap tahap dicek sebelum lanjut ke tahap berikutnya.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: registrasi, onboarding, checkout.
- Alasan pakai: mengurangi beban kognitif user dan meningkatkan kualitas data.
- Kapan tidak dipakai: form sangat pendek (1-2 field sederhana).

## 7) Contoh Sederhana + Bedah Output
```html
<div id="step1">
  <h3>Step 1 - Identitas</h3>
  <input id="nameInput" placeholder="Nama" />
  <p id="nameError"></p>
</div>

<div id="step2" style="display:none;">
  <h3>Step 2 - Kontak</h3>
  <input id="emailInput" placeholder="Email" />
  <p id="emailError"></p>
</div>

<button id="prevBtn">Prev</button>
<button id="nextBtn">Next</button>
<button id="submitBtn" style="display:none;">Submit</button>
<p id="resultText"></p>

<script>
  const state = {
    step: 1,
    form: { name: '', email: '' }
  };

  const step1 = document.querySelector('#step1');
  const step2 = document.querySelector('#step2');
  const nameInput = document.querySelector('#nameInput');
  const emailInput = document.querySelector('#emailInput');
  const nameError = document.querySelector('#nameError');
  const emailError = document.querySelector('#emailError');
  const prevBtn = document.querySelector('#prevBtn');
  const nextBtn = document.querySelector('#nextBtn');
  const submitBtn = document.querySelector('#submitBtn');
  const resultText = document.querySelector('#resultText');

  function validateStep(step) {
    nameError.textContent = '';
    emailError.textContent = '';

    if (step === 1) {
      if (!state.form.name.trim()) {
        nameError.textContent = 'Nama wajib diisi.';
        return false;
      }
      return true;
    }

    if (step === 2) {
      if (!state.form.email.includes('@')) {
        emailError.textContent = 'Email tidak valid.';
        return false;
      }
      return true;
    }
    return true;
  }

  function validateAll() {
    return validateStep(1) && validateStep(2);
  }

  function render() {
    step1.style.display = state.step === 1 ? 'block' : 'none';
    step2.style.display = state.step === 2 ? 'block' : 'none';
    prevBtn.disabled = state.step === 1;
    nextBtn.style.display = state.step === 1 ? 'inline-block' : 'none';
    submitBtn.style.display = state.step === 2 ? 'inline-block' : 'none';
    nameInput.value = state.form.name;
    emailInput.value = state.form.email;
  }

  nameInput.addEventListener('input', (e) => {
    state.form.name = e.target.value;
  });

  emailInput.addEventListener('input', (e) => {
    state.form.email = e.target.value;
  });

  nextBtn.addEventListener('click', () => {
    if (!validateStep(1)) return;
    state.step = 2;
    render();
  });

  prevBtn.addEventListener('click', () => {
    state.step = 1;
    render();
  });

  submitBtn.addEventListener('click', () => {
    if (!validateAll()) return;
    resultText.textContent = `Submit berhasil untuk ${state.form.name}`;
  });

  render();
</script>
```
Bedah output:
1. Step 1 wajib valid sebelum bisa lanjut ke Step 2.
2. Step 2 validasi email sebelum submit final.
3. Error ditampilkan spesifik per step.

## 8) Jebakan Umum (Pitfalls)
- Validasi hanya di akhir, bukan per step.
- State form hilang saat pindah step.
- Tombol navigasi tidak sinkron dengan step aktif.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill:
```js
const state = { step: 1 };
state.step += 1;
console.log(state.step === 2 ? 'step2' : 'other');
```
Kunci:
- Output: `step2`
- Alasan: step naik dari 1 ke 2.

## 10) Debug Story
- Gejala: user berhasil ke step akhir meski field wajib kosong.
- Akar masalah: handler next langsung ubah `state.step` tanpa memanggil validasi.
- Langkah debug: log urutan `validateStep()` dan perubahan step.
- Solusi: jadikan guard validasi sebelum setiap perpindahan step.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membangun minimal 2-step form dengan state terpusat.
- [ ] Bisa menerapkan validasi per step dan validasi final.
- [ ] Bisa menjaga data tetap ada saat berpindah step.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: filter/sort/pagination dan form validation dasar.
- Topik terkait: `12-mini-project-search-filter-list.md`.
- Referensi tambahan: strategi validasi form bertahap.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools.
- File awal: 2 container step + tombol navigasi + area hasil.

## B) Langkah Praktik
1. Definisikan `state.step` dan `state.form`.
2. Render step aktif berdasarkan state.
3. Tambahkan validasi per step di handler Next.
4. Tambahkan validasi final sebelum submit.

## C) Expected Output
- UI yang diharapkan: user hanya bisa lanjut jika step valid.
- Output console/network yang diharapkan: tidak ada error saat pindah step bolak-balik.

## D) Troubleshooting
- Masalah umum 1 + solusi: step tidak berubah, cek guard validasi dan update `state.step`.
- Masalah umum 2 + solusi: nilai field hilang saat pindah step, pastikan value disimpan di state.
