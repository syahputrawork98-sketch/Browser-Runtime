# Runtime Model

Bagian ini menjelaskan bagaimana topik dalam buku ini bekerja dari perspektif runtime browser.

---

# Event Loop Interaction

Browser menggunakan event loop untuk menjadwalkan eksekusi pekerjaan.

Topik yang dibahas dalam buku ini biasanya akan berinteraksi dengan:

- task queue
- microtask queue
- rendering opportunity

---

# Thread Model

Sebagian besar JavaScript dijalankan di **main thread**.

Namun beberapa fitur Web Platform dapat berjalan di thread lain, misalnya melalui Web Workers.

---

# Dampak terhadap Rendering

Beberapa operasi dapat mempengaruhi rendering pipeline, misalnya:

- perubahan DOM
- perubahan style
- layout recalculation
