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
| **Generate** | `build_prompt()` menyisipkan hasil retrieval ke prompt, lalu `generate_answer()` mengirim prompt ke `gemini-flash-latest` (temperature 0.3). |

### Desain prompt
Prompt disusun dengan 4 bagian:
- **Persona:** asisten customer service e-commerce yang ramah, berbahasa Indonesia.
- **Task:** jawab HANYA berdasarkan FAQ di context; jika tidak ada, katakan informasinya belum tersedia dan sarankan menghubungi customer service.
- **Context:** 3 FAQ hasil semantic search, masing-masing diberi label `[FAQ #id]`.
- **Format:** maksimal 4 kalimat dan diakhiri `Sumber: FAQ #id` (atau `Sumber: -` jika tidak ada di FAQ).

---

## 3. Hasil Pengujian & Analisis Relevansi

### Query uji
| # | Tipe | Query | FAQ yang diharapkan |
|---|---|---|---|
| 1 | Ada di FAQ | Metode pembayaran apa saja yang tersedia? | #17 |
| 2 | Ada di FAQ | Bagaimana cara membatalkan pesanan? | #3 |
| 3 | Parafrase | Barang yang aku terima pecah dan kardusnya penyok, harus gimana? | #16 (paket rusak) |
| 4 | Parafrase | Kalau beli HP mahal bisa dicicil per bulan nggak? | #20 (cicilan) |
| 5 | Parafrase | Uang saya kapan balik kalau barangnya sudah saya retur? | #25 (lama refund) |
| 6 | Tidak ada di FAQ | Apakah ada toko offline yang bisa saya kunjungi di Bandung? | — |
| 7 | Tidak ada di FAQ | Apakah saya bisa membayar menggunakan Bitcoin atau crypto? | — (mirip FAQ #17, tapi crypto tidak disebut) |

### Hasil
> ⏳ **Belum diisi.** Akan diisi dari output notebook setelah dijalankan dengan `GEMINI_API_KEY`.

### Analisis
> ⏳ **Belum diisi.** Akan berisi: query mana yang retrieval-nya akurat dan mana yang kurang relevan, apakah query parafrase berhasil ditangkap, serta bagaimana LLM menjawab query yang tidak ada di FAQ.

---

## 4. Keterbatasan Sistem

- Sistem selalu mengambil 3 FAQ teratas walaupun tidak ada yang relevan, jadi penolakan query di luar FAQ sepenuhnya bergantung pada LLM yang mengikuti instruksi prompt.
- FAQ yang topiknya mirip (misal #3 batal manual vs #8 batal otomatis, #25 lama refund vs #27 tujuan refund) bisa tertukar urutannya.
- Pertanyaan yang butuh data pribadi pelanggan (misal "paket saya di mana?") hanya bisa dijawab dengan prosedur umum.
- Bergantung pada API Gemini (kuota/rate limit, dan model bisa di-deprecate).
- Pengujian masih sedikit (7 query), jadi belum bisa mengukur akurasi secara menyeluruh.

> Keterbatasan lain yang ditemukan saat pengujian akan ditambahkan setelah notebook dijalankan.

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
├── .env.example                                # template API key
├── .gitignore
├── README.md
└── requirements.txt
```
