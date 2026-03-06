# Browser Runtime

**Browser Runtime** adalah sebuah **knowledge base jangka panjang** yang mendokumentasikan bagaimana browser menjalankan aplikasi web.

Repository ini berfungsi sebagai **rak buku teknis (technical bookshelf)** yang menjelaskan berbagai konsep fundamental dari **Web Platform dan Browser Runtime**.

Fokus utama repository ini adalah memahami bagaimana browser bekerja di level runtime, termasuk:

* JavaScript runtime
* Event loop dan scheduling
* DOM dan event system
* Rendering pipeline
* Networking (Fetch / HTTP)
* Storage APIs
* Workers dan concurrency
* Security model (Origin / CORS / Permissions)
* Page lifecycle
* Performance dan observability

Repository ini **bukan sekadar tutorial**, tetapi dirancang sebagai:

* **knowledge base jangka panjang**
* **arsitektur referensi Web Platform**
* **roadmap belajar runtime browser**

---

# Tujuan Repository

Tujuan utama project ini adalah:

1. Memahami **Web Platform sebagai sistem runtime**, bukan sekadar API yang digunakan oleh JavaScript.

2. Membangun **mental model yang benar tentang browser runtime**, termasuk hubungan antara:

```
ECMAScript (JavaScript Language)
        ↓
JavaScript Engine (V8 / SpiderMonkey / JavaScriptCore)
        ↓
Web Platform APIs
        ↓
Browser Engine (Blink / WebKit / Gecko)
```

3. Menghilangkan miskonsepsi umum seperti:

* JavaScript melakukan networking sendiri
* JavaScript mengatur event loop sendiri
* JavaScript mengatur rendering sendiri

Padahal sebenarnya:

| Fitur              | Dimana Didefinisikan |
| ------------------ | -------------------- |
| Fetch / Networking | Fetch Standard       |
| Timers             | HTML Standard        |
| DOM                | DOM Standard         |
| Event Loop         | HTML Standard        |
| Rendering          | Browser Engine       |

---

# Struktur Repository

Repository ini dibagi berdasarkan **arsitektur Web Platform**.

```
01-web-platform-architecture
02-javascript-runtime
03-event-loop-and-scheduling
04-dom-and-event-system
05-rendering-pipeline
06-networking
07-storage
08-workers-and-concurrency
09-security-model
10-page-lifecycle
11-performance
12-case-studies
13-spec-companion
```

Setiap folder mewakili **domain utama dalam browser runtime**.

Contoh:

* `03-event-loop-and-scheduling` → bagaimana browser menjadwalkan task
* `05-rendering-pipeline` → bagaimana browser merender halaman
* `06-networking` → bagaimana Fetch dan HTTP bekerja
* `09-security-model` → origin, CORS, permissions

---

# Cara Menggunakan Repository Ini

Repository ini dapat digunakan dengan dua cara:

## 1. Mode Belajar (Learning Path)

Ikuti urutan folder:

```
01 → 02 → 03 → ... → 11
```

Ini memberikan **alur belajar yang sistematis** tentang browser runtime.

---

## 2. Mode Referensi

Jika kamu ingin mencari konsep tertentu:

* Event loop → `03-event-loop-and-scheduling`
* Rendering → `05-rendering-pipeline`
* Networking → `06-networking`

---

# Case Studies

Folder `12-case-studies` berisi analisis masalah nyata seperti:

* Scroll jank
* Layout thrashing
* Large DOM performance
* Long task UI freeze
* Animation pipeline issues

Tujuan case studies adalah memahami **bagaimana konsep runtime berdampak pada performa aplikasi nyata**.

---

# Spec Companion

Folder `13-spec-companion` berisi catatan ketika membaca spesifikasi seperti:

* HTML Standard
* DOM Standard
* Fetch Standard
* Streams Standard

Tujuan folder ini adalah menjembatani antara:

**spesifikasi resmi ↔ pemahaman praktis developer**

---

# Target Pembaca

Repository ini ditujukan untuk:

* Web developers
* Frontend engineers
* JavaScript engineers
* Browser platform learners

yang ingin memahami **bagaimana browser benar-benar bekerja di balik layar**.

---

# Kontribusi

Jika ingin berkontribusi:

Silakan baca terlebih dahulu:

```
CONTRIBUTING.md
```

---

# AI Contribution

Repository ini juga dirancang agar dapat dikembangkan menggunakan AI tools.

AI yang berkontribusi harus mengikuti aturan dalam:

```
AGENTS.md
```

---

# Lisensi

Open for educational purposes.
