# Repository Architecture

Dokumen ini menjelaskan **arsitektur struktur repository Browser Runtime**.

Tujuan struktur ini adalah agar repository tetap:

* konsisten
* mudah dinavigasi
* mudah diperluas
* selaras dengan arsitektur Web Platform

---

# Prinsip Struktur

Struktur repository mengikuti **arsitektur runtime browser**, bukan level belajar.

Artinya folder dibagi berdasarkan **domain sistem browser**, bukan:

```
beginner
intermediate
advanced
```

Pendekatan ini membuat repository lebih cocok sebagai:

* knowledge base jangka panjang
* referensi teknis
* dokumentasi arsitektur runtime

---

# Domain Utama

Repository ini dibagi menjadi beberapa domain utama:

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
```

---

# Penjelasan Domain

### Web Platform Architecture

Menjelaskan hubungan antara:

* ECMAScript
* JavaScript Engine
* Web APIs
* Browser Engine

---

### JavaScript Runtime

Menjelaskan runtime JavaScript di browser:

* call stack
* execution context
* memory model

---

### Event Loop and Scheduling

Menjelaskan mekanisme scheduling browser:

* event loop
* task queue
* microtask queue
* rendering opportunity
* timer clamping

---

### DOM and Event System

Menjelaskan:

* DOM tree
* EventTarget
* event propagation
* capturing / bubbling

---

### Rendering Pipeline

Menjelaskan pipeline rendering:

```
Style → Layout → Paint → Composite
```

---

### Networking

Menjelaskan:

* Fetch API
* HTTP
* request lifecycle
* streaming

---

### Storage

Menjelaskan:

* localStorage
* sessionStorage
* IndexedDB
* Cache API

---

### Workers and Concurrency

Menjelaskan concurrency model browser:

* Web Workers
* MessageChannel
* structured clone
* transferable objects

---

### Security Model

Menjelaskan batas keamanan Web Platform:

* Same-Origin Policy
* CORS
* Secure Context
* Permissions

---

### Page Lifecycle

Menjelaskan lifecycle halaman:

* DOMContentLoaded
* load
* visibilitychange
* Page Lifecycle API
* bfcache

---

### Performance

Menjelaskan observability dan performa runtime:

* Long tasks
* frame budget
* rendering performance
* memory leak patterns

---

# Case Studies

Folder `12-case-studies` berisi analisis kasus nyata.

---

# Spec Companion

Folder `13-spec-companion` berisi catatan dari spesifikasi resmi.

---

# Evolusi Repository

Repository ini dirancang untuk berkembang secara bertahap.

Topik baru dapat ditambahkan selama:

* sesuai dengan domain yang ada
* mengikuti aturan dalam `CONTRIBUTING.md`
