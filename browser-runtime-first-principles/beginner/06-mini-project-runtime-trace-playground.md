# Mini Project: Runtime Trace Playground

## Meta
- Level: `Beginner`
- Estimasi durasi: `60-90 menit`
- Aturan track: lihat `../README.md` section `Aturan Level (Ringkas)`.

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Menyelesaikan modul `01` sampai `05`.
  - [ ] Paham dasar event loop, promise jobs, timer, event dispatch, dan render tick.
- Kamus mini:
  - `[ulang]` trace: jejak urutan eksekusi yang dicatat.
  - `[baru]` timeline event: daftar berurutan kejadian runtime.
  - `[baru]` phase label: penanda fase seperti `sync`, `microtask`, `macrotask`, `raf`.

## 1) Pengantar Singkat Topik
Mini project ini menggabungkan seluruh materi Beginner dalam satu playground untuk melacak urutan runtime secara visual. Tujuannya agar model mental kamu tervalidasi lewat eksperimen yang bisa diulang.

## 2) Big Picture
Belajar teori runtime tanpa alat observasi sering membuat pemahaman semu. Playground trace memaksa kita membandingkan prediksi vs eksekusi aktual. Dari sini, kamu bisa mengisolasi miskonsepsi seperti urutan Promise vs timer, efek bubbling, dan timing render. Hasil akhir mini project ini menjadi fondasi sebelum masuk kasus runtime yang lebih kompleks.

## 3) Small Picture
- Buat panel log runtime dengan urutan nomor.
- Tambahkan skenario trigger: sync, Promise, timer, event bubble, rAF.
- Setiap trigger menulis label fase ke timeline.
- Sediakan tombol reset agar eksperimen bisa diulang konsisten.

## 4) Wireframe Alur Konsep
```text
[klik Run Scenario]
  -> [sync log]
  -> [schedule promise/timer/raf]
  -> [drain microtask]
  -> [jalankan timer]
  -> [render tick + raf]
  -> [timeline final]
```
- Alur utama: timeline menunjukkan urutan runtime nyata.
- Alur jalan: event child memicu parent lewat bubbling.
- Alur error: urutan log salah karena label fase tidak konsisten.

## 5) Analogi Dunia Nyata
Seperti black box recorder: semua kejadian penting dicatat berurutan agar kita tahu apa yang sebenarnya terjadi.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: latihan prediksi runtime dan debugging urutan eksekusi.
- Alasan pakai: mempercepat pembentukan mental model berbasis bukti.
- Kapan tidak dipakai: tidak ada, ini alat belajar inti untuk track ini.

## 7) Contoh Sederhana + Bedah Output
```html
<button id="runBtn">Run Scenario</button>
<button id="resetBtn">Reset</button>
<ul id="timeline"></ul>

<div id="parent" style="padding:8px; border:1px solid #999; margin-top:8px;">
  Parent
  <button id="child">Child Click</button>
</div>

<script>
  const runBtn = document.querySelector('#runBtn');
  const resetBtn = document.querySelector('#resetBtn');
  const timeline = document.querySelector('#timeline');
  const parent = document.querySelector('#parent');
  const child = document.querySelector('#child');

  let step = 0;
  function trace(label) {
    step += 1;
    const li = document.createElement('li');
    li.textContent = `${step}. ${label}`;
    timeline.appendChild(li);
  }

  function resetTimeline() {
    step = 0;
    timeline.innerHTML = '';
  }

  parent.addEventListener('click', () => trace('event: parent bubble'));
  child.addEventListener('click', (e) => {
    trace('event: child target');
    // e.stopPropagation(); // aktifkan untuk melihat perubahan urutan
  });

  runBtn.addEventListener('click', () => {
    trace('sync: start');

    Promise.resolve().then(() => {
      trace('microtask: promise then');
    });

    setTimeout(() => {
      trace('macrotask: timer 0');
    }, 0);

    requestAnimationFrame(() => {
      trace('raf: before paint');
    });

    trace('sync: end');
    child.click(); // simulasi event dispatch
  });

  resetBtn.addEventListener('click', resetTimeline);
</script>
```
Bedah output:
1. `sync: start` dan `sync: end` selalu muncul lebih dulu.
2. Event child/parent muncul sesuai flow dispatch.
3. Promise then muncul sebelum timer.
4. Callback rAF tampil pada fase frame berikutnya (sebelum paint).

## 8) Jebakan Umum (Pitfalls)
- Mencampur label fase sehingga trace tidak bisa dianalisis.
- Mengira simulasi sekali jalan sudah cukup tanpa reset dan uji ulang.
- Tidak memisahkan skenario event bubbling dari skenario async queue.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill 1:
```js
trace('sync A');
Promise.resolve().then(() => trace('microtask B'));
setTimeout(() => trace('timer C'), 0);
trace('sync D');
```
Kunci:
- Urutan: `sync A`, `sync D`, `microtask B`, `timer C`
- Alasan: sync -> microtask -> macrotask.

Drill 2:
```js
parent.addEventListener('click', () => trace('P'));
child.addEventListener('click', () => trace('C'));
child.click();
```
Kunci:
- Urutan: `C`, `P`
- Alasan: target child dulu, lalu bubble ke parent.

Drill 3:
```js
trace('before');
requestAnimationFrame(() => trace('raf'));
Promise.resolve().then(() => trace('microtask'));
trace('after');
```
Kunci:
- Urutan umum: `before`, `after`, `microtask`, `raf`
- Alasan: microtask di-drain setelah sync, rAF pada fase frame berikutnya.

## 10) Debug Story
- Gejala: hasil trace beda tiap kali run sehingga sulit dipercaya.
- Akar masalah: timeline tidak di-reset, listener dobel terpasang, atau skenario bercampur.
- Langkah debug: reset state sebelum run, cek listener hanya sekali, dan jalankan skenario terpisah.
- Solusi: jadikan `reset -> run -> observasi` sebagai prosedur baku playground.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa membuat timeline runtime yang repeatable.
- [ ] Bisa membedakan label `sync`, `microtask`, `macrotask`, `raf`, dan `event`.
- [ ] Bisa menjelaskan minimal 3 temuan dari trace hasil eksperimen.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: modul `01` sampai `05`.
- Topik terkait: `intermediate/07-microtask-starvation-dan-fairness.md`.
- Referensi tambahan: event loop visualizers dan DevTools performance timeline.

## 13) Model Mental
- Entitas utama: call stack, microtask queue, macrotask queue, event dispatch flow, render tick.
- Urutan proses: trigger -> sync log -> schedule jobs -> queue drain -> frame callback.
- Invariant (aturan yang selalu benar): tanpa trace berlabel fase, debugging runtime mudah menipu intuisi.

## 14) Failure Case
- Skenario gagal: developer menyimpulkan "timer lebih cepat dari promise" dari satu eksperimen yang tidak terkontrol.
- Akar penyebab runtime: trace tidak dipisah per fase dan hasil tercampur oleh skenario event lain.
- Perbaikan: gunakan playground dengan label fase eksplisit dan uji per-skenario.

## 15) Spec Hint
- Section spec terkait (ringkas): event loop processing model, timer task source, Promise jobs, dan animation frame callbacks.
- Kata kunci pencarian spec: `event loop`, `queue a task`, `microtask`, `requestAnimationFrame`, `dispatch event`.
