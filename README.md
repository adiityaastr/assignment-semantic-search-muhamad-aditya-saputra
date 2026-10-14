# Semantic Search & LLM Application — Asisten FAQ E-Commerce

Prototype asisten customer service berbasis **semantic search + LLM**. FAQ diubah menjadi embedding (Gemini Embeddings API), disimpan di vector database (FAISS), dicari berdasarkan kemiripan makna, lalu dijadikan context untuk Gemini dalam menyusun jawaban.

---

## 1. Problem Statement

### Objective
Tim customer service sebuah e-commerce menerima ratusan pertanyaan repetitif setiap hari seputar pemesanan, pengiriman, pembayaran, dan pengembalian barang. Prototype ini dibuat untuk:

1. Mencari FAQ yang paling relevan dengan pertanyaan pelanggan, **walaupun pelanggan tidak memakai kata-kata yang sama** dengan FAQ.
2. Menyusun jawaban yang natural menggunakan LLM, **hanya berdasarkan FAQ yang ditemukan**.
3. Memberi tahu pelanggan jika informasinya tidak ada di FAQ, bukan mengarang jawaban.

### Sumber & struktur knowledge base
- **Sumber:** `data/ecommerce_faq_knowledge_base.csv` dari brief assignment. File tidak diubah.
- **Ukuran:** 59 entri FAQ, 4 kolom, tidak ada missing value.

| Kolom | Keterangan |
|---|---|
| `faq_id` | ID unik FAQ (1–59) |
| `category` | Kategori topik |
| `question` | Pertanyaan FAQ |
| `answer` | Jawaban FAQ |

Jumlah FAQ per kategori: Pemesanan (9), Pengiriman (9), Pembayaran (8), Pengembalian (8), Akun (7), Produk (7), Promo (6), Customer Service (5).

### Batasan & asumsi
- Sistem **hanya menjawab berdasarkan FAQ yang tersedia**, bukan pengetahuan umum LLM.
- Satu entri FAQ = satu unit yang di-retrieve. Teks FAQ pendek, jadi tidak perlu chunking.
- Query dan jawaban dalam Bahasa Indonesia.
- Sistem tidak punya akses ke data pesanan pelanggan (status, resi, dll.) dan belum menyimpan riwayat percakapan.

---

## 2. Alur Sistem

```
INGEST  →  EMBED  →  STORE  →  RETRIEVE  →  GENERATE
```

| Tahap | Yang dilakukan |
|---|---|
| **Ingest** | Load CSV dengan Pandas, cek struktur data, lalu gabungkan kategori + question + answer menjadi satu teks per FAQ (kolom `content`). |
| **Embed** | Semua teks FAQ diubah menjadi vektor dengan `gemini-embedding-001` (task type `RETRIEVAL_DOCUMENT`) dalam satu request batch. Query pengguna di-embed dengan task type `RETRIEVAL_QUERY`. Model `text-embedding-004` di starter sudah tidak tersedia, jadi diganti. |
| **Store** | Vektor dimasukkan ke `faiss.IndexFlatL2`, dicek jumlahnya sama dengan jumlah FAQ (59), lalu disimpan ke `vector_store/faq_index.faiss`. |
| **Retrieve** | `semantic_search(query, top_k=3)` mengembalikan 3 FAQ terdekat beserta distance L2 (semakin kecil = semakin mirip). |
| **Generate** | `build_prompt()` menyisipkan hasil retrieval ke prompt, lalu `generate_answer()` mengirim prompt ke `gemini-flash-lite-latest` (temperature 0.3). |

### Desain prompt
Prompt disusun dengan 4 bagian:
- **Persona:** asisten customer service e-commerce yang ramah, berbahasa Indonesia.
- **Task:** jawab HANYA berdasarkan FAQ di context; jika tidak ada, katakan informasinya belum tersedia dan sarankan menghubungi customer service.
- **Context:** 3 FAQ hasil semantic search, masing-masing diberi label `[FAQ #id]`.
- **Format:** maksimal 4 kalimat dan diakhiri `Sumber: FAQ #id` (atau `Sumber: -` jika tidak ada di FAQ).

---

## 3. Hasil Pengujian & Analisis Relevansi

Pengujian memakai 7 query: 2 yang jelas ada di FAQ, 3 parafrase/bahasa informal, dan 2 yang tidak ada di FAQ. Distance = jarak L2, **semakin kecil semakin mirip**. Output lengkap ada di Section 8 notebook, dan detail jawaban LLM per query ada di [`documentation/retrieval_test_results.md`](documentation/retrieval_test_results.md).

### Hasil
| # | Tipe | Query | FAQ diharapkan | Top-3 hasil retrieval (distance) | Jawaban LLM |
|---|---|---|---|---|---|
| 1 | Ada di FAQ | Metode pembayaran apa saja yang tersedia? | #17 | **#17 (0.41)**, #22 (0.54), #20 (0.57) | ✅ Benar, Sumber: FAQ #17 |
| 2 | Ada di FAQ | Bagaimana cara membatalkan pesanan? | #3 | **#3 (0.42)**, #24 (0.56), #8 (0.58) | ✅ Benar, Sumber: FAQ #3 |
| 3 | Parafrase | Barang yang aku terima pecah dan kardusnya penyok, harus gimana? | #16 | **#16 (0.43)**, #54 (0.53), #24 (0.58) | ✅ Benar, Sumber: FAQ #16 |
| 4 | Parafrase | Kalau beli HP mahal bisa dicicil per bulan nggak? | #20 | **#20 (0.50)**, #40 (0.68), #57 (0.69) | ✅ Benar, Sumber: FAQ #20 |
| 5 | Parafrase | Uang saya kapan balik kalau barangnya sudah saya retur? | #25 | **#25 (0.42)**, #27 (0.53), #24 (0.54) | ✅ Benar, menggabungkan #25 dan #27 |
| 6 | Tidak ada di FAQ | Apakah ada toko offline yang bisa saya kunjungi di Bandung? | — | #58 (0.72), #46 (0.74), #2 (0.74) | ✅ "Belum memiliki informasi", Sumber: - |
| 7 | Tidak ada di FAQ | Apakah saya bisa membayar menggunakan Bitcoin atau crypto? | — | #17 (0.61), #18 (0.68), #22 (0.69) | ✅ "Belum memiliki informasi", Sumber: - |

**Ringkasan:** FAQ yang benar ada di peringkat 1 pada **5/5** query yang jawabannya tersedia, termasuk 3/3 query parafrase. LLM menjawab sesuai FAQ pada **7/7** query, dan tidak berhalusinasi pada 2 query di luar FAQ.

### Analisis
**1. Retrieval akurat, termasuk untuk parafrase.**
Query yang wording-nya mirip FAQ (1 dan 2) mendapat distance paling kecil (0.41–0.42). Query parafrase juga berhasil: "pecah dan kardusnya penyok" tidak memakai kata "rusak" sama sekali tetapi tetap cocok dengan FAQ #16 (paket rusak), dan "uang kapan balik ... retur" cocok dengan FAQ #25 tentang refund. Artinya embedding menangkap **makna**, bukan sekadar kata kunci yang sama.

**2. Query informal dengan informasi tambahan lebih "jauh".**
Query 4 punya distance top-1 terbesar (0.50) di antara query valid karena ada detail yang tidak ada di FAQ ("HP mahal", "per bulan") dan bahasa informal ("nggak"). Meski begitu, selisihnya dengan peringkat 2 (0.68) masih besar, jadi FAQ #20 tetap jelas yang paling relevan.

**3. Peringkat 2–3 sering kurang relevan.**
Karena sistem selalu mengambil top-3, FAQ yang tidak berhubungan ikut masuk context. Contohnya, query 4 ikut mengambil #40 (info promo) dan #57 (bandingkan produk). Pada query 2, #24 (pengembalian barang) berada di atas #8 (pesanan dibatalkan otomatis) yang secara topik lebih dekat. Untungnya LLM mengabaikan FAQ yang tidak relevan ini.

**4. Ada celah distance antara query yang ada dan tidak ada di FAQ.**
Top-1 query yang ada di FAQ: **0.40–0.50**. Top-1 query di luar FAQ: **0.61–0.72**. Jadi ambang distance sekitar **0.55** bisa dipakai untuk membuang context yang tidak relevan sebelum dikirim ke LLM. Angka ini masih perlu diuji dengan lebih banyak query.

**5. LLM patuh pada context.**
- Semua jawaban sesuai isi FAQ, singkat, dan mencantumkan sumber, sehingga mudah diverifikasi.
- Query 5: LLM menggabungkan FAQ #25 (lama refund) dan #27 (ke mana dana dikembalikan) menjadi jawaban yang lebih lengkap. Ini contoh baik penggunaan beberapa context sekaligus.
- Query 7 (crypto) adalah kasus *near-miss*: FAQ #17 (metode pembayaran) ikut terambil, tetapi LLM **tidak mengarang** apakah crypto didukung atau tidak, dan memilih jawaban fallback. Ini aman, walaupun akan lebih membantu jika sekaligus menyebutkan metode pembayaran yang tersedia.
- Query 6 (toko offline): context sama sekali tidak relevan, dan LLM dengan tepat menjawab bahwa informasinya belum tersedia lalu menyarankan menghubungi customer service.

---

## 4. Keterbatasan Sistem

- **Selalu mengambil top-3.** Belum ada ambang distance, jadi FAQ yang tidak relevan tetap masuk context. Penolakan query di luar FAQ sepenuhnya bergantung pada LLM yang mengikuti instruksi prompt.
- **FAQ bertetangga bisa tertukar urutan**, misalnya #24 (retur) muncul di atas #8 (batal otomatis) untuk query pembatalan pesanan.
- **Format output belum 100% konsisten**: sapaan berbeda-beda ("Halo!" vs "Halo Kak!"), dan format sumber kadang menyimpang ("FAQ #25 dan #27" bukan "FAQ #25, FAQ #27").
- **Jawaban fallback kurang membantu pada kasus near-miss** (query crypto): sistem tidak menyebutkan informasi terdekat yang sebenarnya tersedia.
- **Tidak ada akses data pelanggan.** Pertanyaan seperti "paket saya di mana?" hanya bisa dijawab dengan prosedur umum.
- **Ketergantungan pada API Gemini.** Saat pengerjaan, `gemini-2.5-flash` sudah tidak tersedia untuk akun baru, dan beberapa model Flash lain sering error 503 (server sibuk), sehingga dipakai `gemini-flash-lite-latest` ditambah retry sederhana. Kuota embedding free tier juga terbatas (100 request/menit, dihitung per teks), jadi notebook sebaiknya tidak dijalankan berkali-kali dalam waktu berdekatan.
- **Pengujian masih sedikit** (7 query), jadi belum bisa mengukur akurasi secara menyeluruh.

---

## 5. Cara Menjalankan

**Prasyarat:** Python 3.10+ dan Gemini API key (gratis di <https://aistudio.google.com/apikey>).

```bash
# 1. Install dependency
pip install -r requirements.txt

# 2. Siapkan API key
Copy-Item .env.example .env     # Windows PowerShell
# cp .env.example .env          # macOS / Linux
#    lalu buka .env dan ganti your_api_key_here dengan API key Anda

# 3. Jalankan notebook
jupyter notebook notebook/starter_notebook_semantic_search.ipynb
#    lalu pilih Kernel → Restart & Run All
```

Notebook memakai path relatif (`../data/...`), jadi pastikan dibuka dari folder `notebook/` seperti perintah di atas. Setelah dijalankan, FAISS index tersimpan di `vector_store/faq_index.faiss`.

---

## 6. Struktur Repository

```
semantic-search-assignment/
├── data/
│   └── ecommerce_faq_knowledge_base.csv        # knowledge base asli (tidak diubah)
├── notebook/
│   └── starter_notebook_semantic_search.ipynb  # seluruh pipeline + pengujian
├── vector_store/
│   └── faq_index.faiss                         # FAISS index (hasil notebook)
├── documentation/
│   └── retrieval_test_results.md               # detail hasil uji + jawaban LLM
├── .env.example                                # template API key
├── .gitignore
├── README.md
└── requirements.txt
```
