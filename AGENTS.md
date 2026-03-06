# AI Contribution Guidelines

Dokumen ini menjelaskan aturan untuk AI tools yang membantu mengembangkan repository ini.

---

# Tujuan AI Contribution

AI digunakan untuk membantu:

* menjelaskan konsep Web Platform
* merapikan dokumentasi
* menambahkan contoh kode
* membantu pembaruan knowledge base

AI **tidak boleh mengubah arsitektur repository tanpa izin eksplisit**.

---

# Struktur Repository

AI harus selalu menggunakan struktur berikut:

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

AI **tidak boleh membuat folder baru di root** tanpa pembaruan dokumentasi arsitektur.

---

# Terminologi

AI harus menggunakan terminologi resmi dari:

* HTML Standard
* DOM Standard
* Fetch Standard
* ECMAScript Specification

---

# Penulisan

Setiap topik harus menjelaskan:

1. definisi formal
2. mental model
3. runtime perspective
4. contoh kode
5. common misconceptions
6. pitfall dan best practices

---

# Larangan

AI tidak boleh:

* mencampur konsep ECMAScript dengan Web APIs
* menulis informasi yang tidak sesuai dengan spesifikasi
* membuat struktur folder baru secara sembarangan

---

# Tujuan Akhir

AI harus membantu repository ini menjadi **knowledge base teknis yang akurat tentang browser runtime**.
