# 📊 Analisis Sentimen Komentar YouTube Warga Indonesia terhadap Danantara Indonesia (Pipeline ELT)

Proyek ini mengimplementasikan pipeline data berbasis **ELT (Extract, Load, Transform)** untuk mengumpulkan, mengelola, membersihkan, dan menganalisis opini serta sentimen masyarakat Indonesia terhadap pembentukan **Danantara (Badan Pengelola Investasi Daya Anagata Nusantara)**. Data diekstraksi dari kolom komentar video YouTube Raymond Chin menggunakan **YouTube Data API v3**, disimpan ke dalam database NoSQL **MongoDB**, dan dianalisis menggunakan metode gabungan **InSet Lexicon-Based** serta **Support Vector Machine (SVM) dengan TF-IDF**.

---

## 📌 1. Pengenalan Proyek

### 1.1 Latar Belakang
Media sosial kini telah menjelma menjadi ruang publik digital utama bagi masyarakat Indonesia untuk mengekspresikan opini, emosi, kritik, maupun harapan terhadap setiap kebijakan strategis pemerintah. Salah satu topik ekonomi nasional yang paling hangat diperdebatkan baru-baru ini adalah peluncuran **Danantara Indonesia**. Badan ini dirancang secara khusus untuk mengonsolidasikan, mengintegrasikan, dan mengoptimalkan seluruh aset serta investasi kekayaan negara demi mendorong pertumbuhan ekonomi nasional agar setara dengan *sovereign wealth fund* global terkemuka (seperti Temasek atau GIC Singapura).

Melalui konten edukasi finansial dan analisis makroekonomi yang diunggah pada kanal YouTube Raymond Chin, isu Danantara memicu respons yang masif. Ribuan opini netizen dari berbagai latar belakang berkumpul di kolom komentar. Proyek ini hadir sebagai solusi berbasis data (*data-driven*) untuk menangkap suara publik tersebut, melakukan pemrosesan data tekstual dalam skala besar, serta mengekstrak wawasan (*insight*) kuantitatif mengenai tingkat penerimaan publik, kecemasan sistemis, dan harapan warga Indonesia terhadap masa depan Danantara.

### 1.2 Tujuan Proyek
1. **Otomatisasi Penambangan Data (Data Scraping):** Membangun skrip pipeline penarikan data komentar secara masif dari beberapa video YouTube bertopik spesifik secara *real-time* dengan memanfaatkan YouTube Data API v3.
2. **Arsitektur Penyimpanan Terpusat (Data Ingestion):** Mengelola penyimpanan data mentah (*raw data*) berformat semi-terstruktur ke dalam basis data NoSQL MongoDB demi menjamin skalabilitas dan performa integritas data.
3. **Pembersihan Data Kontekstual (Text Preprocessing):** Membangun algoritma pembersihan teks khusus untuk menangani karakteristik unik bahasa netizen Indonesia, termasuk pemetaan kata gaul (*slang*), penanganan singkatan, serta eliminasi bot *spam* promosi judi online yang sering mengotori kolom komentar media sosial.
4. **Analisis Sentimen Multi-Metode:** Mengklasifikasikan data komentar ke dalam tiga polaritas sentimen (Positif, Netral, Negatif) menggunakan pendekatan Leksikon Bahasa Indonesia (*InSet Lexicon*).
5. **Ekstraksi Fitur & Identifikasi Topik Utama:** Menggunakan pembobotan matematika *Term Frequency-Inverse Document Frequency (TF-IDF)* dikombinasikan dengan *Support Vector Machine (SVM)* untuk menemukan 20 kata kunci teratas (*Top 20 Words*) yang paling memengaruhi dinamika tren opini publik.

---

## 🏗️ 2. Arsitektur Sistem & Alur Kerja (Pipeline ELT)

Proyek ini mengadopsi paradigma desain **ELT (Extract, Load, Transform)**. Pendekatan ini dipilih agar seluruh data mentah (*raw JSON*) aman tersimpan di dalam database terlebih dahulu sebelum melalui proses manipulasi string, normalisasi teks, dan pemodelan machine learning yang intensif.

```text
+------------------+       EXTRACT       +-------------------+
| YouTube Platform | ------------------> | YouTube API v3    |
+------------------+                     +-------------------+
                                                   |
                                                   | Penarikan Data Komentar
                                                   v
+------------------+        LOAD         +-------------------+
| MongoDB Database | <------------------ | Python Dataframe  |
+------------------+                     +-------------------+
        |
        | Read Raw Document
        v
+------------------------------------------------------------+
|                       TRANSFORM                            |
|                                                            |
|  1. Text Cleansing & Regex (Hapus Emoji, URL, Bot Spam)   |
|  2. Case Folding (Menyeragamkan huruf menjadi kecil)       |
|  3. Tokenization & Filtering (Pemberhentian Stopwords)     |
|  4. Normalization (Slang-word & Typo Mapping)              |
|  5. Sentiment Labeling (InSet Lexicon-Based Scoring)       |
|  6. Feature Engineering & Weighting (TF-IDF Vectorizer)   |
+------------------------------------------------------------+
        |
        | Pemodelan SVM & Visualisasi Data
        v
+------------------------------------------------------------+
|                    INSIGHTS & OUTPUT                       |
|                                                            |
|  - Metrik Distribusi Sentimen Publik (Persentase)         |
|  - Bobot Kata Kunci Dominan (Top 20 Words - SVM/TFIDF)     |
|  - Rekomendasi Kebijakan & Transparansi Tata Kelola       |
+------------------------------------------------------------+

```

### Detail Implementasi Komponen ELT:

1. **Extract (Ekstraksi):**
* Menggunakan modul `google-api-python-client` untuk membuat koneksi terotentikasi melalui token Developer API Key.
* Mengimplementasikan fungsi iterasi `getcomments(video_id)` yang secara rekursif melacak parameter `nextPageToken` untuk mengekstrak seluruh halaman komentar tanpa batasan limit default.
* Ekstraksi menyasar data metadata esensial: `author` (nama akun), `updated_at` (stempel waktu), `like_count` (jumlah disukai), `text` (konten teks komentar mentah), `video_id`, dan status `public`.
* Video target mencakup 5 ID video utama Raymond Chin yang membahas Danantara secara intensif, antara lain: `TYxqBTdOq24`, `QOcP5OvSwlI`, `Lfzu74XDyco`, `TiS6vnju_mI`, dan `cYwioeHu_OU`.


2. **Load (Pemuatan):**
* Data mentah hasil penarikan langsung diubah menjadi dokumen BSON/JSON dan dimasukkan secara *bulk insert* menggunakan driver `pymongo` ke kluster **MongoDB**.
* Pemilihan database NoSQL memastikan arsitektur fleksibel, aman dari kehilangan struktur data awal, dan mempermudah query data analitik secara berkala tanpa membutuhkan skema tabel relasional yang kaku.


3. **Transform (Transformasi Teks & Preprocessing):**
* **Data Cleansing:** Menggunakan ekspresi reguler (`regex`) untuk melenyapkan karakter non-alfabet, emotikon, tautan eksternal (URL), dan pola teks buatan bot spam iklan keuangan/saldo judi (contoh: teks promosi terselubung bermotif "AXL777").
* **Case Folding:** Mengonversi seluruh teks menjadi huruf kecil (*lowercase*) demi menghindari redudansi kata akibat variasi kapitalisasi.
* **Tokenization & Stopwords Removal:** Memecah teks komentar menjadi deretan token kata mandiri dan mengeliminasi kata fungsional yang tidak membawa muatan makna emosi (seperti kata depan: *di, ke, dari, yang, ini, itu*).
* **Slang & Formalization Mapping:** Mencocokkan token teks dengan kamus kustom untuk menormalisasi kata tidak baku netizen (contoh: `"ga"`, `"gk"` $\rightarrow$ `"tidak"`; `"ntar"` $\rightarrow$ `"nanti"`; `"koh"` $\rightarrow$ `"koko"`).
* **Labeling (InSet Lexicon):** Setiap token kata dicocokkan dengan Kamus Sentimen Bahasa Indonesia *InSet*. Kata bermuatan positif diberi nilai $+1$ hingga $+5$, sedangkan bermuatan negatif $-1$ hingga $-5$. Skor total akhir akumulatif dari kalimat menentukan label sentimen:
* Skor $> 0$: **Sentimen Positif**
* Skor $= 0$: **Sentimen Netral**
* Skor $< 0$: **Sentimen Negatif**


* **Feature Extraction (TF-IDF):** Mentransformasikan data teks bersih menjadi matriks fitur numerik berdimensi tinggi menggunakan metode *Term Frequency-Inverse Document Frequency* untuk mengukur derajat kepentingan suatu kata unik dalam korpus dokumen.



---

## 🛠️ 3. Teknologi dan Tools yang Digunakan

* **Bahasa Pemrograman:** Python v3.x
* **Database & Storage:** MongoDB & MongoDB Compass (Alat GUI Pemantauan Koleksi)
* **API Connector:** YouTube Data API v3 (Google Cloud Platform Developer Console)
* **Library Ekstraksi & Manipulasi Data:** `pandas`, `numpy`, `google-api-python-client`, `pymongo`
* **Library Analisis & Machine Learning:** `scikit-learn` (TF-IDF Vectorizer, Pemodelan SVM)
* **Library Visualisasi Grafik:** `matplotlib`, `seaborn`
* **Lingkungan Pengembangan:** Jupyter Notebook (`.ipynb`) & Visual Studio Code

---

## 📁 4. Struktur Direktori Proyek

```text
📦 analisis-sentimen-danantara
├── 📁 data
│   ├── Data_Scrapped_Youtube_RaymondChin.csv   # Dataset mentah hasil scraping perdana
│   └── data_cleaned_Youtube_RaymondChin.csv    # Dataset bersih hasil preprocessing & transformasi
├── 📁 notebooks
│   ├── Scrapped_Youtube_Data.ipynb             # Notebook untuk tahap Extract & Load ke MongoDB
│   └── SentimentAnalysis_Danantara.ipynb       # Notebook untuk tahap Transform, Labeling, & Modeling SVM
├── 📄 Report.pdf                               # Dokumen presentasi laporan komprehensif proyek ELT
├── 📄 README.md                                # Dokumentasi lengkap repositori proyek (File ini)
└── 📄 .env.example                             # Template kredensial rahasia untuk API Key & URI Database

```

---

## ⚙️ 5. Langkah Instalasi dan Konfigurasi

### Prasyarat Sistem

Pastikan perangkat lokal Anda telah terpasang:

* Python versi 3.8 atau yang terbaru.
* MongoDB Community Server berjalan aktif di lokal (`localhost:27017`) atau menggunakan akun MongoDB Atlas.
* Kredensial aktif YouTube Data API Key dari Google Cloud Console.

### Panduan Instalasi:

1. **Clone Repositori Proyek:**
```bash
git clone [https://github.com/username/analisis-sentimen-danantara.git](https://github.com/username/analisis-sentimen-danantara.git)
cd analisis-sentimen-danantara

```


2. **Instalasi Semua Library Dependensi Python:**
Jalankan perintah pip berikut untuk memasang seluruh kebutuhan pustaka:
```bash
pip install pandas numpy pymongo scikit-learn matplotlib seaborn google-api-python-client

```


3. **Konfigurasi Environment Kredensial:**
Salin berkas konfigurasi `.env.example` menjadi `.env` lalu lengkapi isinya:
```env
DEV_API_KEY=KUNCI_API_YOUTUBE_DATA_V3_ANDA
MONGO_URI=mongodb://localhost:27017/

```



---

## 🚀 6. Panduan Menjalankan Pipeline

### Langkah 1: Ekstraksi Data & Ingesti ke Database (Extract & Load)

1. Jalankan aplikasi MongoDB Server Anda di latar belakang.
2. Buka notebook `notebooks/Scrapped_Youtube_Data.ipynb`.
3. Inisialisasi variabel API Key dengan kredensial Anda.
4. Eksekusi seluruh sel (*cells*). Skrip akan melakukan hit API ke YouTube, menarik total **1.758 komentar**, memformatnya menjadi dokumen database, dan menyimpannya secara otomatis ke dalam database MongoDB.

### Langkah 2: Preprocessing, Analisis, dan Pemodelan (Transform)

1. Buka notebook `notebooks/SentimentAnalysis_Danantara.ipynb`.
2. Skrip akan memanggil fungsi interkoneksi database untuk membaca koleksi data mentah dari MongoDB.
3. Jalankan blok *cleansing* teks untuk mengeksekusi penyaringan karakter, penghapusan spam iklan, penyelarasan kata tidak baku (*slang*), dan pelabelan otomatis berbasis *InSet Lexicon*.
4. Jalankan sel ekstraksi fitur TF-IDF dan algoritma klasifikasi SVM untuk menghasilkan pembobotan fitur kata kunci terkuat.
5. Tampilkan grafik visualisasi distribusi sentimen publik.

---

## 📊 7. Hasil Analisis, Metrik & Wawasan Terbimbing

Berdasarkan pemrosesan analitis terhadap korpus data sebanyak **1.758 baris komentar** netizen Indonesia, diperoleh hasil temuan kuantitatif dan kualitatif sebagai berikut:

### 7.1 Distribusi Klasifikasi Sentimen (InSet Lexicon Based)

| Kelas Sentimen | Jumlah Komentar | Persentase Distribusi | Indikator Visual |
| --- | --- | --- | --- |
| **Sentimen Negatif** | 927 Komentar | **52.76%** | 🔴 Mayoritas Dominan |
| **Sentimen Netral** | 428 Komentar | **24.36%** | 🟡 Moderat |
| **Sentimen Positif** | 402 Komentar | **22.88%** | 🟢 Minoritas Optimis |

### 7.2 Analisis Bobot Kata Kunci Utama (Top 20 Words - SVM-TFIDF)

Berdasarkan pembobotan matematis algoritma TF-IDF yang dikonfirmasi melalui bobot fitur model linear SVM, deretan kata kunci teratas yang paling memengaruhi dinamika tren perbincangan publik meliputi:

> *danantara, indonesia, negara, rakyat, korupsi, tidak, masalah, bisa, manfaat, kita, saya, akan, dari, untuk, nya, ini, itu, ada, ya, jk.*

### 💡 7.3 Interpretasi Wawasan Strategis (*Strategic Insights*)

1. **Tingkat Skeptisisme Publik yang Sangat Tinggi (Sentimen Negatif - 52.76%):**
Angka sentimen negatif yang menembus lebih dari separuh total opini publik mengindikasikan adanya **krisis kepercayaan (*public trust crisis*)** historis yang kuat dari masyarakat terhadap komitmen tata kelola (*governance*) lembaga keuangan milik pemerintah. Kemunculan kata kunci berbobot ekstrem seperti **"korupsi"**, **"tidak"**, dan **"masalah"** membuktikan bahwa ingatan kolektif netizen Indonesia masih trauma dengan kasus-kasus mega korupsi pengelolaan dana investasi masa lalu (seperti Jiwasraya, Asabri, atau Taspen). Warga mengkhawatirkan bahwa jargon-jargon megah pembentukan *superholding* investasi global ini berpotensi amburadul pada level eksekusi di lapangan dan rentan dijadikan alat penyelewengan dana baru jika pengawasannya lemah.
2. **Harapan Kolektif yang Tersisa (Sentimen Positif - 22.88%):**
Meskipun diliputi awan skeptisisme, ruang optimisme publik tetap terekam lewat kemunculan kata-kata berbobot positif seperti **"bisa"**, **"manfaat"**, **"negara"**, dan **"rakyat"**. Kluster sentimen ini mewakili kelompok masyarakat yang memandang Danantara secara objektif berdasarkan konsep teoritisnya. Mereka percaya bahwa apabila badan pengelola ini dijalankan secara independen, profesional, dipimpin oleh talenta kredibel, serta dibersihkan dari intervensi politik praktis, Danantara benar-benar mampu mendongkrak daya saing ekonomi Indonesia secara signifikan demi kemakmuran rakyat banyak.
3. **Rekomendasi Kebijakan bagi Pemangku Kepentingan (*Stakeholders*):**
Hasil analisis sentimen ini bertindak sebagai indikator sinyal awal (*early warning signal*) bagi manajemen Danantara dan pemerintah. Diperlukan reformasi strategi komunikasi publik yang transparan. Pemerintah wajib memaparkan regulasi kepatuhan hukum (*compliance*) secara gamblang, memperketat audit internal-eksternal secara berkala, serta membuktikan langkah-langkah konkret pencegahan korupsi guna mengikis kecemasan publik dan mengubah skeptisisme menjadi dukungan nasional yang solid.

---

## 👤 8. Identitas Pengembang / Peneliti

* **Nama Lengkap:** Al Fitra Nur Ramadhani (Dhani)
* **Nomor Induk Mahasiswa:** 202210370311264
* **Mata Kuliah / Kelas:** Data, Information, and Knowledge B
* **Program Studi:** Informatika
* **Institusi:** Universitas Muhammadiyah Malang (UMM)