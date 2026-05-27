q# 📘 RAG Parameter Exploration Assignment

Pada tugas ini kamu akan bereksperimen dengan **6 parameter utama Retrieval-Augmented Generation (RAG)** menggunakan **RAG Playground**.

### 🎯 Tujuan Pembelajaran

Kamu akan:

✅ Memahami **apa fungsi** masing-masing parameter RAG

✅ Mengamati **pengaruh** tuning terhadap kualitas jawaban vs kecepatan

✅ Belajar **trade-off** antara retrieval & generation

✅ Menulis **insight praktis** dari eksperimen yang kamu lakukan

> **Tools sudah disediakan.** Kamu **tidak perlu ngoding**, hanya ubah nilai parameter di file `app.py` & restart server.

### ⚠️ Notes Penting

- **Ikuti video di LMS jika kamu kebingungan.**
- **Sediakan penyimpanan at least 2GB**, akan ada download model yang cukup besar ke laptopmu.
- **Hapus proyek ini kalau sudah selesai**, biar laptop-mu tidak tercekik.
- **Ketika proyek ini sudah diisi & dikerjakan di bawah, upload ke GitHub.**

---

## ⚙️ Parameter yang Diuji

Semua parameter berikut dapat ditemukan di file `app.py`:

```python
# Buka app.py, cari bagian KONFIGURASI INTI
CHUNK_SIZE = 300
TOP_K_INITIAL = 20
FINAL_TOP_M = 5
USE_RERANKER = True
TEMPERATURE = 0.2
GROQ_MODEL = "llama-3.1-8b-instant"
```

---

### 📏 1. CHUNK_SIZE

```python
CHUNK_SIZE = 300  # ubah angka ini
```

**Apa itu CHUNK_SIZE?**

Saat dokumen PDF diupload, teksnya dipotong-potong menjadi bagian kecil (chunk) agar bisa diproses dan dicari. `CHUNK_SIZE` menentukan berapa banyak **kata** per potongan.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari CHUNK_SIZE)*

> Jawab: `CHUNK_SIZE` mengatur panjang setiap potongan teks saat dokumen dipecah menjadi chunk sebelum di-embedding dan dicari. Semakin tepat ukuran chunk, semakin seimbang antara fokus isi chunk dan kelengkapan konteks yang bisa diambil saat retrieval.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (100–200)** | *(Chunk menjadi lebih fokus sehingga presisi pencarian bisa naik untuk pertanyaan spesifik, tetapi konteks mudah terpecah dan informasi penting bisa tersebar di beberapa chunk.)* |
| **Sedang (250–400)** ⭐ Recommended | *(Range ini biasanya paling seimbang karena masih cukup ringkas untuk retrieval yang presisi, tetapi tetap menyimpan konteks yang cukup utuh untuk menjawab pertanyaan yang butuh penjelasan.)* |
| **Besar (500–1000)** | *(konteks lebih luas, tetapi pencarian bisa menjadi kurang tajam karena satu chunk memuat terlalu banyak topik dan noise, sehingga relevansi hasil retrieval dapat turun.)* |

**Pertanyaan Refleksi:**

* Chunk besar atau kecil yang lebih tepat untuk dokumen yang kamu tes? Mengapa?
  > Jawab: ...0.2, 0.84, 0.81. chunk sedang paling bagus, namun jika pilihannya antara yg besar dan kecil saya memilih besar. karena konteks lebih terjaga dan tidak ada informasi penting yg terlewat seperti pembagian pada chunk kecil.

* Apa yang terjadi kalau jawaban terasa "terpotong" atau kurang konteks? Parameter mana yang harus diubah?
  > Jawab: biasanya anatara chunk terlalu kecil atau transisi pembagian chunk antarbagian dokumen tidak terjaga. coba naikin`CHUNK_SIZE`, kalau masih terasa terpotong dan kurang konteks coba naikin `CHUNK_OVERLAP` agar sebagian isi chunk sebelumnya ikut terbawa ke chunk berikutnya

---

### 🔍 2. TOP_K_INITIAL

```python
TOP_K_INITIAL = 20
```

**Apa itu TOP_K_INITIAL?**

Setelah pertanyaan diubah jadi vektor, sistem mencari chunk yang paling mirip. `TOP_K_INITIAL` menentukan berapa banyak **kandidat awal** yang diambil dari hasil pencarian vektor sebelum di-filter/rerank.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari TOP_K_INITIAL)*

> Jawab: TOP_K_INITIAL  menentukan berapa banyak kandidat chunk awal yang diambil dari hasil similarity search sebelum dipilih lagi untuk tahap berikutnya. Parameter ini penting untuk mengatur keseimbangan antara recall yang luas dan waktu proses retrieval

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (5–10)** | *(Lebih cepat karena kandidat yang dicek sedikit, tetapi recall bisa turun karena beberapa chunk relevan mungkin tidak ikut masuk ke daftar awal. )* |
| **Sedang (15–25)** ⭐ Recommended | *(Range ini cukup luas untuk menangkap kandidat relevan tambahan, tetapi belum terlalu berat untuk proses reranking dan generasi. )* |
| **Besar (30–50)** | *(Recall biasanya meningkat karena lebih banyak kandidat diperiksa, tetapi waktu proses juga naik dan risiko noise ikut bertambah sebelum reranking dilakukan.)* |

> 💡 **Catatan:** Nilai ini hanya berpengaruh signifikan jika `USE_RERANKER = True`. Tanpa reranker, yang dipakai langsung hanya `FINAL_TOP_M` teratas.

**Pertanyaan Refleksi:**

* Apakah top-k kecil (5–10) cukup untuk dataset kamu? Jelaskan alasannya.
  > Jawab: Top-k kecil bisa cukup jika dokumennya pendek, topiknya sempit, dan pertanyaannya sangat langsung. Namun jika jawaban tersebar di beberapa bagian dokumen atau istilah pertanyaannya tidak persis sama dengan isi PDF, top-k kecil lebih berisiko melewatkan chunk yang sebenarnya penting.

* Apa hubungan antara `TOP_K_INITIAL` dan `USE_RERANKER`?
  > Jawab: TOP_K_INITIAL  paling terasa manfaatnya saat  USE_RERANKER = True  karena reranker butuh kumpulan kandidat awal yang cukup banyak untuk dipilah ulang menjadi konteks terbaik. Jika reranker dimatikan, penambahan kandidat awal biasanya tidak terlalu membantu karena sistem tidak punya tahap penilaian ulang yang lebih akurat.

---

### 📋 3. FINAL_TOP_M

```python
FINAL_TOP_M = 5
```

**Apa itu FINAL_TOP_M?**

Dari sekian banyak kandidat yang ditemukan, hanya **M teratas** (setelah reranking) yang dikirim ke LLM sebagai konteks untuk menjawab pertanyaan.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari FINAL_TOP_M)*

> Jawab: ...

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Sedikit (2–3)** | *(Jelaskan pengaruhnya terhadap fokus jawaban dan risiko konteks terlewat)* |
| **Sedang (4–6)** ⭐ Recommended | *(Jelaskan mengapa range ini direkomendasikan)* |
| **Banyak (7–10)** | *(Jelaskan pengaruhnya terhadap kelengkapan, kecepatan, dan potensi kebingungan LLM)* |

> 💡 **Catatan:** Semakin banyak konteks = semakin banyak token yang dikirim ke LLM = latency lebih tinggi. Jaga agar tetap efisien.

**Pertanyaan Refleksi:**

* Benarkah "semakin banyak konteks semakin bagus"? Jelaskan berdasarkan eksperimenmu.
  > Jawab: Tidak selalu. Dari hasil eksperimen 4, 7, dan 6, saya melihat bahwa konteks yang terlalu banyak belum tentu memberi jawaban terbaik karena model bisa menerima informasi yang berlebihan atau kurang fokus, sehingga pilihan menengah lebih efektif.

* Apa yang terjadi kalau konteks yang dikirim ke LLM saling bertentangan?
  > Jawab: Jika chunk yang dikirim saling bertentangan, LLM bisa menghasilkan jawaban yang ragu-ragu, mencampur dua informasi, atau memilih salah satu tanpa dasar yang jelas. Karena itu, pemilihan konteks akhir harus tetap ketat agar hanya chunk yang paling relevan dan konsisten yang masuk ke prompt.

---

### 🎯 4. USE_RERANKER

```python
USE_RERANKER = True  # atau False
```

**Apa itu Reranker?**

Reranker adalah model tambahan (Cross-Encoder) yang mengevaluasi ulang pasangan `[pertanyaan, chunk]` untuk mengurutkan relevansi lebih akurat daripada hanya cosine similarity biasa.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari USE_RERANKER)*

> Jawab: ...

**Pengaruhnya:**

| Mode | Efek |
| --- | --- |
| `True` | *(Jelaskan pengaruhnya terhadap presisi, relevansi konteks, dan latency)* |
| `False` | *(Jelaskan apa yang terjadi jika hanya mengandalkan cosine similarity)* |

> 💡 **Catatan:** Untuk dokumen teknis atau pertanyaan yang ambigu, reranker sangat membantu. Untuk kecepatan maksimal, matikan.

**Pertanyaan Refleksi:**

* Pada query jenis apa reranker paling terasa manfaatnya? Berikan contoh.
  > Jawab: ...

* Berapa kira-kira tambahan latency saat reranker dinyalakan di eksperimenmu?
  > Jawab: ...

---

### 🌡️ 5. TEMPERATURE

```python
TEMPERATURE = 0.2  # range: 0.0 - 1.0
```

**Apa itu Temperature?**

Temperature mengatur distribusi probabilitas saat LLM memilih kata berikutnya. Semakin rendah, semakin "pasti" dan konsisten jawabannya.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari TEMPERATURE)*

> Jawab: ...

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **0.0–0.2** ⭐ Recommended untuk RAG | *(Jelaskan sifat jawaban di range ini: konsistensi, faktualitas)* |
| **0.3–0.5** | *(Jelaskan sifat jawaban di range ini)* |
| **0.6–0.8** | *(Jelaskan sifat jawaban di range ini: kreativitas, variasi)* |
| **0.9–1.0** | *(Jelaskan risiko jawaban di range ini: halusinasi, keacakan)* |

> ⚠️ **Catatan:** Untuk RAG (jawaban berdasarkan dokumen), gunakan temperature rendah (0.1–0.3) agar jawaban tetap *grounded* pada konteks yang diberikan.

**Pertanyaan Refleksi:**

* Untuk QA berbasis dokumen perusahaan, temperatur ideal berapa? Jelaskan alasannya.
  > Jawab: ...

* Apa yang terjadi pada jawaban saat kamu naikkan temperature ke 0.9?
  > Jawab: ...

---

### 🤖 6. GROQ_MODEL

```python
GROQ_MODEL = "llama-3.1-8b-instant"  # model default
```

**Model lain yang bisa dicoba:**

```python
# Ganti value GROQ_MODEL ke salah satu di bawah:
GROQ_MODEL = "openai/gpt-oss-120b"
GROQ_MODEL = "llama-3.3-70b-versatile"
```

**Apa bedanya tiap model?**

Setiap model punya ukuran parameter dan kemampuan yang berbeda. Isi tabel di bawah berdasarkan eksperimenmu:

| Model | Ukuran | Pengaruh terhadap Kecepatan & Kualitas |
| --- | --- | --- |
| `llama-3.1-8b-instant` (Default) | 8B parameter | *(Jelaskan kecepatan, kualitas jawaban, dan kapan cocok dipakai)* |
| `openai/gpt-oss-120b` | 120B parameter | *(Jelaskan kecepatan, kualitas jawaban, dan kapan cocok dipakai)* |
| `llama-3.3-70b-versatile` | 70B parameter | *(Jelaskan kecepatan, kualitas jawaban, dan kapan cocok dipakai)* |

> 💡 **Tips:** Mulai dengan `llama-3.1-8b-instant` untuk development. Saat butuh akurasi tinggi, switch ke `llama-3.3-70b-versatile` atau `openai/gpt-oss-120b`.

**Pertanyaan Refleksi:**

* Apakah peningkatan kualitas jawaban dari model besar sebanding dengan penambahan waktu? Jelaskan.
  > Jawab: ...

* Model mana yang paling cocok untuk use-case dokumenmu? Mengapa?
  > Jawab: ...

---

## ⚙️ Parameter Tambahan (Non-Tunable)

Parameter ini sudah di-set optimal. Ubah hanya jika paham dampaknya.

```python
CHUNK_OVERLAP = 60     # overlap antar chunk (kata)
SCORE_THRESHOLD = 0.30 # skor minimum cosine similarity
```

**Penjelasan:**

| Parameter | Nilai Default | Fungsi |
| --- | --- | --- |
| `CHUNK_OVERLAP` | 60 | *(Jelaskan fungsi overlap antar chunk dan apa dampaknya jika terlalu besar/kecil)* |
| `SCORE_THRESHOLD` | 0.30 | *(Jelaskan fungsi threshold skor dan kapan harus dinaikkan/diturunkan)* |

---

## 📝 Tips Eksperimen

* Pakai PDF minimal 1 halaman, maksimal 10 halaman, biar gak berat
* Coba berbagai jenis pertanyaan: **definisi**, **konteks**, dan **reasoning**
* Catat kapan sistem salah & mengapa
* Ubah **satu parameter pada satu waktu** agar bisa melihat pengaruhnya secara jelas
* Prioritaskan kualitas insight dalam laporan

---

## 🔗 Referensi

* Dokumentasi lengkap parameter: buka `docs.html` di browser atau akses via menu **📖 Dokumentasi** di RAG Playground
* Source code: `app.py` — semua parameter ada di bagian `KONFIGURASI INTI`
