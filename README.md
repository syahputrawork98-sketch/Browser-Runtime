# Web API Learning Hub

Web API Learning Hub adalah sebuah **rak buku pembelajaran Web API pada browser** yang disusun dalam bentuk **monorepo**.

Repository ini berisi kumpulan buku yang membahas berbagai **Web APIs yang tersedia di Web Platform** seperti DOM, Fetch API, Storage API, WebSocket, Service Worker, dan banyak API browser lainnya.

Setiap folder buku di repository ini merupakan **sebuah buku yang fokus pada satu kelompok Web API tertentu**, sehingga materi dapat dipelajari secara modular dan terstruktur.

Repository ini dirancang agar dapat berkembang secara bertahap dan diperbarui seiring dengan perkembangan spesifikasi Web Platform.

---

# Filosofi Repository

Web API Learning Hub menggunakan pendekatan **rak buku (bookshelf architecture)**.

Struktur konsepnya adalah sebagai berikut:

* Repository ini adalah **rak buku utama**
* Setiap folder `book-*` adalah **sebuah buku**
* Setiap buku fokus pada **satu domain Web API**
* Setiap buku memiliki dokumentasi, aturan, dan roadmap sendiri

Pendekatan ini membantu menjaga:

* struktur pembelajaran tetap modular
* topik tidak saling tercampur
* pembaruan materi lebih mudah dilakukan

---

# Struktur Repository

Struktur utama repository ini:

```
web-api-learning-hub
│
├─ README.md
├─ docs/
│
├─ book-01-web-platform-fundamentals
├─ book-02-dom-fundamentals
├─ book-03-dom-manipulation
├─ book-04-dom-events-system
├─ book-05-fetch-api-and-http-requests
├─ book-06-browser-storage-apis
├─ book-07-cookies-api
├─ book-08-timing-apis
├─ book-09-url-and-navigation-apis
├─ book-10-history-api
├─ book-11-form-apis
├─ book-12-file-and-blob-apis
├─ book-13-drag-and-drop-api
├─ book-14-clipboard-api
├─ book-15-canvas-graphics-api
├─ book-16-media-apis-audio-and-video
├─ book-17-geolocation-api
├─ book-18-web-workers-api
├─ book-19-websocket-api
└─ book-20-service-worker-and-offline-apis
```

---

# Daftar Buku

Repository ini saat ini berisi koleksi buku berikut.

| No | Buku                          |
| -- | ----------------------------- |
| 1  | Web Platform Fundamentals     |
| 2  | DOM Fundamentals              |
| 3  | DOM Manipulation              |
| 4  | DOM Events System             |
| 5  | Fetch API & HTTP Requests     |
| 6  | Browser Storage APIs          |
| 7  | Cookies API                   |
| 8  | Timing APIs                   |
| 9  | URL & Navigation APIs         |
| 10 | History API                   |
| 11 | Form APIs                     |
| 12 | File & Blob APIs              |
| 13 | Drag and Drop API             |
| 14 | Clipboard API                 |
| 15 | Canvas Graphics API           |
| 16 | Media APIs (Audio & Video)    |
| 17 | Geolocation API               |
| 18 | Web Workers API               |
| 19 | WebSocket API                 |
| 20 | Service Worker & Offline APIs |

Setiap buku memiliki:

* `README.md` untuk menjelaskan buku tersebut
* `docs/` untuk aturan penulisan dan scope buku
* struktur materi yang berdiri sendiri

---

# Pembaruan Konten

Web Platform dan Web APIs berkembang cukup cepat. Oleh karena itu repository ini dirancang agar dapat:

* diperbarui secara berkala
* mengikuti perubahan spesifikasi Web
* menerima pembaruan otomatis jika diperlukan

Perubahan dapat berasal dari:

* update spesifikasi Web
* update dokumentasi resmi
* revisi materi pembelajaran
* kontribusi dari komunitas

Beberapa bagian dokumentasi juga dapat **diperbarui secara otomatis** dengan bantuan alat atau AI untuk menjaga materi tetap relevan dengan spesifikasi terbaru.

---

# Sumber Referensi

Materi dalam repository ini disusun berdasarkan berbagai sumber resmi dan dokumentasi teknis Web Platform.

Sumber utama yang digunakan antara lain:

### Spesifikasi Web

* WHATWG HTML Standard
  https://html.spec.whatwg.org/

* WHATWG DOM Standard
  https://dom.spec.whatwg.org/

* Fetch Standard
  https://fetch.spec.whatwg.org/

* WebSocket Standard
  https://websockets.spec.whatwg.org/

* Service Worker Specification
  https://w3c.github.io/ServiceWorker/

---

### Dokumentasi Referensi

* MDN Web Docs
  https://developer.mozilla.org/

* Web.dev
  https://web.dev/

---

### Standar Web

* W3C
  https://www.w3.org/

* WHATWG
  https://whatwg.org/

---

# Tujuan Proyek

Tujuan utama dari Web API Learning Hub adalah:

* membangun referensi pembelajaran Web API yang terstruktur
* memecah topik Web API menjadi buku-buku kecil yang fokus
* menyediakan dokumentasi yang mudah diperbarui
* mempermudah eksplorasi Web Platform bagi developer

---

# Dokumentasi Tambahan

Dokumentasi aturan repository dapat ditemukan di folder berikut:

```
docs/
```

Folder ini berisi:

* aturan struktur repository
* aturan penulisan materi
* scope global repository
* pedoman kontribusi

---

# Lisensi

Repository ini menggunakan lisensi yang akan ditentukan pada tahap selanjutnya.
