# Mini Project: Dashboard Browser Sederhana

## Meta
- Level: `Beginner`
- Estimasi durasi: `60-90 menit`
- Status drill: `opsional ringan`

## 0) Prasyarat dan Kamus Mini
- Prasyarat:
  - [ ] Selesai materi `01` sampai `05`.
  - [ ] Paham fetch, state-render, validasi, dan localStorage dasar.
- Kamus mini:
  - `[ulang]` state: sumber data utama UI.
  - `[ulang]` loading/error: status proses async di UI.
  - `[baru]` dashboard card: blok ringkas untuk menampilkan informasi.

## 1) Pengantar Singkat Topik
Topik ini menggabungkan seluruh materi Beginner menjadi satu mini project dashboard yang interaktif dan persist.

## 2) Big Picture
Belajar per topik terpisah sering belum cukup untuk membangun aplikasi utuh. Mini project ini memaksa kita menyatukan event, state, fetch, validasi, error handling, dan persistence dalam satu alur yang konsisten. Setelah selesai, kamu punya baseline arsitektur frontend kecil yang siap ditingkatkan ke level Intermediate.

## 3) Small Picture
- Definisikan state aplikasi untuk input, list task, dan data API.
- Render dashboard dari satu fungsi `render()`.
- Hubungkan event form dan tombol.
- Simpan task ke localStorage.
- Load data eksternal dengan fetch + loading/error.

## 4) Wireframe Alur Konsep
```text
[app start]
  -> [load local state]
  -> [render awal]
  -> [fetch summary API]
  -> [render sukses/error]
  -> [user input/update]
  -> [save + render ulang]
```
- Alur utama: app load -> data lokal + API tampil -> interaksi user tersimpan.
- Alur jalan: API gagal -> UI tetap jalan dengan data lokal.
- Alur error: state berubah tapi tidak disave/render, dashboard jadi tidak sinkron.

## 5) Analogi Dunia Nyata
Seperti papan kontrol harian: ada catatan lokal yang kamu tulis sendiri dan ada data eksternal yang diambil otomatis.

## 6) Dipakai untuk Apa + Alasan
- Dipakai untuk: latihan final level Beginner.
- Alasan pakai: menguji kemampuan integrasi, bukan cuma potongan konsep.
- Kapan tidak dipakai: kalau masih belum nyaman dengan materi 01-05.

## 7) Contoh Sederhana + Bedah Output
```html
<section>
  <h2>Dashboard Mini</h2>
  <p id="apiStatus">Loading summary...</p>
  <pre id="apiCard"></pre>
</section>

<section>
  <form id="taskForm">
    <input id="taskInput" placeholder="Tambah task" />
    <button type="submit">Add</button>
  </form>
  <p id="taskError"></p>
  <ul id="taskList"></ul>
  <button id="clearBtn">Clear Tasks</button>
</section>

<script>
  const STORAGE_KEY = 'dashboard-tasks';
  const state = {
    tasks: [],
    api: { loading: false, error: '', data: null }
  };

  const apiStatus = document.querySelector('#apiStatus');
  const apiCard = document.querySelector('#apiCard');
  const taskForm = document.querySelector('#taskForm');
  const taskInput = document.querySelector('#taskInput');
  const taskError = document.querySelector('#taskError');
  const taskList = document.querySelector('#taskList');
  const clearBtn = document.querySelector('#clearBtn');

  function saveTasks() {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state.tasks));
  }

  function loadTasks() {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return;
    try {
      const parsed = JSON.parse(raw);
      if (Array.isArray(parsed)) state.tasks = parsed;
    } catch (_) {
      state.tasks = [];
    }
  }

  function render() {
    taskList.innerHTML = '';
    for (const task of state.tasks) {
      const li = document.createElement('li');
      li.textContent = task;
      taskList.appendChild(li);
    }

    if (state.api.loading) {
      apiStatus.textContent = 'Loading summary...';
      apiCard.textContent = '';
    } else if (state.api.error) {
      apiStatus.textContent = `Error: ${state.api.error}`;
      apiCard.textContent = '';
    } else if (state.api.data) {
      apiStatus.textContent = 'Summary loaded';
      apiCard.textContent = JSON.stringify(state.api.data, null, 2);
    }
  }

  async function loadSummary() {
    state.api.loading = true;
    state.api.error = '';
    render();
    try {
      const res = await fetch('https://jsonplaceholder.typicode.com/todos/1');
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      state.api.data = await res.json();
    } catch (err) {
      state.api.error = err.message;
    } finally {
      state.api.loading = false;
      render();
    }
  }

  taskForm.addEventListener('submit', (event) => {
    event.preventDefault();
    taskError.textContent = '';
    const value = taskInput.value.trim();
    if (!value) {
      taskError.textContent = 'Task tidak boleh kosong.';
      return;
    }
    state.tasks.push(value);
    taskInput.value = '';
    saveTasks();
    render();
  });

  clearBtn.addEventListener('click', () => {
    state.tasks = [];
    saveTasks();
    render();
  });

  loadTasks();
  render();
  loadSummary();
</script>
```
Bedah output:
1. Saat awal, task lokal dimuat dari storage.
2. Ringkasan API menampilkan loading lalu hasil/error.
3. Tambah task valid langsung tampil dan persist setelah reload.
4. Task kosong menampilkan error validasi.

## 8) Jebakan Umum (Pitfalls)
- Render terpisah-pisah tanpa pusat `render()`.
- Tidak reset error setelah input valid.
- Fetch error menyebabkan seluruh dashboard tidak bisa dipakai.

## 9) Prediksi Output Drill + Kunci Jawaban
Drill (opsional):
```js
const state = { tasks: ['A'] };
state.tasks.push('B');
console.log(state.tasks.length);
```
Kunci:
- Output: `2`
- Alasan: array bertambah satu elemen.

## 10) Debug Story
- Gejala: task baru muncul, tapi hilang setelah refresh.
- Akar masalah: task ditambah ke state, tetapi `saveTasks()` tidak dipanggil.
- Langkah debug: cek localStorage key setelah submit.
- Solusi: pastikan alur event submit selalu `update state -> save -> render`.

## 11) Checkpoint Kelulusan Topik
- [ ] Bisa menyatukan fetch + state + validasi + storage dalam satu app.
- [ ] Bisa menjaga UI tetap responsif saat API gagal.
- [ ] Bisa menjelaskan alur data dashboard dari start sampai interaksi user.

## 12) Jika Masih Bingung, Baca Ini Dulu
- Materi prasyarat: modul `01` sampai `05`.
- Topik terkait: `intermediate/07-dom-component-pattern-tanpa-framework.md`.
- Referensi tambahan: MDN fetch, localStorage, dan form submit.

## A) Setup dan Lingkungan
- Browser target: Chrome/Edge/Firefox versi modern.
- Tools: browser DevTools (Console + Application + Network).
- File awal: satu `index.html` dengan 2 section (summary + tasks).

## B) Langkah Praktik
1. Implementasi state dan render untuk panel task.
2. Tambahkan validasi form + persistence localStorage.
3. Tambahkan fetch summary + loading/error state.
4. Uji refresh, offline mode, dan clear task.

## C) Expected Output
- UI yang diharapkan: dashboard tetap berfungsi walau API gagal.
- Output console/network yang diharapkan: tidak ada error JS yang tidak ditangani.

## D) Troubleshooting
- Masalah umum 1 + solusi: data API null terus, cek URL dan status `res.ok`.
- Masalah umum 2 + solusi: task tidak persist, cek `saveTasks()` dan key localStorage.
