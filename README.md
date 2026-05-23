# Proyek Analisis Kausal (Causal Inference) - Data Science Course R

Repositori/direktori ini berisi serangkaian penyelesaian tugas dan latihan terkait materi **Causal Inference** (Inferensi Kausal) yang disusun dalam bahasa pemrograman R. Proyek ini mendemonstrasikan berbagai metode modern dalam memahami, mengidentifikasi, dan mengukur hubungan sebab-akibat (*causality*), bukan sekadar korelasi.

## 📌 Daftar Latihan

Proyek ini terbagi menjadi 6 (enam) latihan utama yang dikerjakan menggunakan R Markdown (`.Rmd`). Dokumentasi lengkap dan rangkuman hasil dari semua latihan ini dikonsolidasikan dalam file [`Laporan Causal Inference.md`](Laporan%20Causal%20Inference.md).

### [Latihan 1: Identifikasi Backdoor Path (Konseptual)](Latihan_5_1_Backdoor_Path.Rmd)
- **Topik**: Penggunaan *Directed Acyclic Graph* (DAG) dengan package `dagitty`.
- **Deskripsi**: Mengidentifikasi *Total Effect* dan *Direct Effect*, evaluasi *Backdoor Path*, dan menganalisis mengapa mengontrol variabel mediator terkadang memberikan bias (*Collider Bias*).

### [Latihan 2: Simpson's Paradox](Latihan_5_2_Simpsons_Paradox.Rmd)
- **Topik**: Analisis Paradoks Simpson, Confounder vs Mediator.
- **Deskripsi**: Membuktikan ada atau tidaknya *Simpson's Paradox* dalam dataset medis fiktif. Menganalisis skenario kausal yang benar untuk interpretasi data apakah variabel kontrol (*Jenis Kelamin*) berkedudukan sebagai *Confounder* atau *Mediator*.

### [Latihan 3: Backdoor Adjustment Manual](Latihan_5_3_Backdoor_Adjustment.Rmd)
- **Topik**: Perhitungan probabilitas intervensi ($do$-calculus).
- **Deskripsi**: Melakukan evaluasi *backdoor criterion* dan secara manual menghitung probabilitas jika dilakukan intervensi (*Average Causal Effect* / ACE) berdasarkan data observasional terkait merokok dan penyakit jantung.

### [Latihan 4: Counterfactual Individu](Latihan_5_4_Counterfactual_Individu.Rmd)
- **Topik**: *Structural Causal Model* (SCM) dan *Individual Treatment Effect* (ITE).
- **Deskripsi**: Menerapkan 3 langkah dasar kausalitas tipe kontrafaktual (*Abduction*, *Action*, *Prediction*) untuk mencari efek intervensi spesifik terhadap satu individu (ITE) dan membedakannya dari *Average Treatment Effect* (ATE).

### [Latihan 5: CausalImpact — Interpretasi Output](Latihan_5_5_CausalImpact.Rmd)
- **Topik**: *Bayesian Structural Time-Series* (BSTS) pada data Runtun Waktu (*Time-Series*).
- **Deskripsi**: Menggunakan package `CausalImpact` (dari Google) untuk mengestimasi dampak dari sebuah intervensi di tengah berjalannya waktu. Melakukan analisis sensitivitas (*Post-Treatment Bias*) ketika variabel kontrol ikut terdampak oleh intervensi.

### [Latihan 6 (Tantangan): Mediation Analysis Lengkap](Latihan_5_6_Mediation_Analysis.Rmd)
- **Topik**: Mediasi Kausal, Bootstrap, dan Analisis Sensitivitas.
- **Deskripsi**: Eksplorasi mendalam terkait mediasi emosi dalam perilaku politik. Menggabungkan regresi tradisional *Baron-Kenny* dan package `mediation` untuk mengekstrak *Average Causal Mediation Effect* (ACME), dibuktikan dengan visualisasi diagram jalur (Path Diagram).

## 🛠️ Alat dan Pustaka (Libraries)
Latihan ini sangat bergantung pada beberapa pustaka krusial di ekosistem R untuk analisis kausal:
- `dagitty` & `ggdag` untuk identifikasi grafik kausal dan kriteria penyesuaian (*adjustment*).
- `CausalImpact` untuk analisis intervensi *time-series*.
- `mediation` untuk evaluasi *indirect effects* dan analisis sensitivitas.
- `ggplot2` untuk semua kebutuhan visualisasi.

## 📄 Cara Membaca
Anda dapat membaca ulasan lengkap dan kesimpulan hasil analisis pada file pelaporan utama:
👉 **[`Laporan Causal Inference.md`](Laporan%20Causal%20Inference.md)**

Atau Anda dapat mengeksekusi langsung *source code* R Markdown (*.Rmd*) yang tersedia untuk memproduksi ulang visualisasi atau model statistiknya. Setiap interaksi juga memiliki rekaman log historisnya sendiri (berformat `.Rhistory`).
# Causal-Inference
