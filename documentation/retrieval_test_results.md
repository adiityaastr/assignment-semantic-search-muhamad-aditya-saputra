# Hasil Pengujian Retrieval & Response AI

Output asli dari Section 8 notebook `notebook/starter_notebook_semantic_search.ipynb` (tidak diedit).
Embedding: `gemini-embedding-001` · Index: FAISS `IndexFlatL2` · LLM: `gemini-flash-lite-latest` · top_k = 3.
Distance = jarak L2, semakin kecil semakin mirip. Analisis lengkap ada di README.md bagian 3.

## 1. [Ada di FAQ] Metode pembayaran apa saja yang tersedia?

**FAQ yang diharapkan:** #17

**Hasil retrieval (top-3):**

1. FAQ #17 - Metode pembayaran apa saja yang tersedia? (distance: 0.4068)
2. FAQ #22 - Apakah bisa membayar menggunakan lebih dari satu metode sekaligus? (distance: 0.5411)
3. FAQ #20 - Apakah saya bisa membayar dengan cicilan? (distance: 0.5736)

**Jawaban LLM:**

> Halo! Kami mendukung beberapa metode pembayaran, yaitu transfer bank, kartu kredit/debit, e-wallet, dan pembayaran melalui minimarket rekanan. 
> 
> Sumber: FAQ #17

## 2. [Ada di FAQ] Bagaimana cara membatalkan pesanan?

**FAQ yang diharapkan:** #3

**Hasil retrieval (top-3):**

1. FAQ #3 - Bagaimana cara membatalkan pesanan yang sudah dibuat? (distance: 0.4239)
2. FAQ #24 - Bagaimana prosedur pengembalian barang yang tidak sesuai? (distance: 0.5618)
3. FAQ #8 - Kenapa pesanan saya otomatis dibatalkan sistem? (distance: 0.5842)

**Jawaban LLM:**

> Halo! Pesanan dapat dibatalkan melalui halaman 'Pesanan Saya' selama status pesanan masih 'Diproses'. Namun, jika status sudah berubah menjadi 'Dikirim', pembatalan tidak dapat dilakukan lagi. 
> 
> Sumber: FAQ #3

## 3. [Parafrase] Barang yang aku terima pecah dan kardusnya penyok, harus gimana?

**FAQ yang diharapkan:** #16

**Hasil retrieval (top-3):**

1. FAQ #16 - Apa yang harus dilakukan jika paket diterima dalam kondisi rusak? (distance: 0.4291)
2. FAQ #54 - Saya menerima barang yang salah, apa yang harus saya lakukan? (distance: 0.5337)
3. FAQ #24 - Bagaimana prosedur pengembalian barang yang tidak sesuai? (distance: 0.5784)

**Jawaban LLM:**

> Halo Kak! Jika paket diterima dalam kondisi rusak, Kakak harus segera mengambil foto atau video kondisi paket saat diterima. Setelah itu, laporkan melalui menu komplain dalam waktu 2x24 jam sejak barang diterima ya.
> 
> Sumber: FAQ #16

## 4. [Parafrase] Kalau beli HP mahal bisa dicicil per bulan nggak?

**FAQ yang diharapkan:** #20

**Hasil retrieval (top-3):**

1. FAQ #20 - Apakah saya bisa membayar dengan cicilan? (distance: 0.5024)
2. FAQ #40 - Bagaimana cara mendapatkan info promo terbaru? (distance: 0.6842)
3. FAQ #57 - Bagaimana cara membandingkan beberapa produk sekaligus? (distance: 0.6902)

**Jawaban LLM:**

> Halo! Fitur cicilan tersedia untuk transaksi dengan kartu kredit tertentu pada nominal minimal yang ditentukan oleh pihak bank. 
> 
> Sumber: FAQ #20

## 5. [Parafrase] Uang saya kapan balik kalau barangnya sudah saya retur?

**FAQ yang diharapkan:** #25

**Hasil retrieval (top-3):**

1. FAQ #25 - Berapa lama proses refund setelah pengembalian disetujui? (distance: 0.4241)
2. FAQ #27 - Ke mana dana refund akan dikembalikan? (distance: 0.5329)
3. FAQ #24 - Bagaimana prosedur pengembalian barang yang tidak sesuai? (distance: 0.5436)

**Jawaban LLM:**

> Halo! Dana refund biasanya akan diproses dalam waktu 3 hingga 7 hari kerja setelah barang yang dikembalikan diterima dan diverifikasi oleh penjual. Dana tersebut akan dikembalikan ke metode pembayaran awal Anda atau ke saldo akun, tergantung pada opsi yang Anda pilih saat pengajuan.
> 
> Sumber: FAQ #25 dan #27

## 6. [Tidak ada di FAQ] Apakah ada toko offline yang bisa saya kunjungi di Bandung?

**FAQ yang diharapkan:** —

**Hasil retrieval (top-3):**

1. FAQ #58 - Apakah bisa memesan dalam jumlah besar untuk kebutuhan reseller? (distance: 0.72)
2. FAQ #46 - Di mana saya bisa melihat ukuran/size chart produk pakaian? (distance: 0.7375)
3. FAQ #2 - Apakah saya bisa memesan tanpa membuat akun? (distance: 0.7376)

**Jawaban LLM:**

> Mohon maaf, saya belum memiliki informasi tersebut. Silakan menghubungi customer service kami untuk mendapatkan bantuan lebih lanjut mengenai hal ini.
> 
> Sumber: -

## 7. [Tidak ada di FAQ] Apakah saya bisa membayar menggunakan Bitcoin atau crypto?

**FAQ yang diharapkan:** —

**Hasil retrieval (top-3):**

1. FAQ #17 - Metode pembayaran apa saja yang tersedia? (distance: 0.6137)
2. FAQ #18 - Apakah ada biaya tambahan untuk pembayaran menggunakan kartu kredit? (distance: 0.6761)
3. FAQ #22 - Apakah bisa membayar menggunakan lebih dari satu metode sekaligus? (distance: 0.6872)

**Jawaban LLM:**

> Mohon maaf, saat ini kami belum memiliki informasi tersebut di dalam sistem kami. Silakan hubungi customer service kami untuk mendapatkan bantuan lebih lanjut.
> 
> Sumber: -
