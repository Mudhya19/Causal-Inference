# Laporan Dokumentasi: Identifikasi Backdoor Path (Konseptual)

**Tanggal:** 23 Mei 2026  
**Topik:** Analisis Kausal - Identifikasi Backdoor Path pada DAG

## 1. Pendahuluan
Laporan ini merangkum hasil analisis kausal berdasarkan grafik *Directed Acyclic Graph* (DAG) yang menghubungkan variabel Rokok (R), Tar (T), dan Kanker (K). Analisis dilakukan menggunakan bahasa pemrograman R dengan pustaka `dagitty` dan `ggdag`.

**Struktur DAG:**
- **R (Rokok):** Exposure / Paparan
- **T (Tar):** Mediator (Kandungan tar di paru-paru)
- **K (Kanker):** Outcome / Hasil
- *Asumsi: Tidak ada confounder tersembunyi dalam model ini.*

**Relasi Kausal:**
- $R \rightarrow T$: Merokok menyebabkan akumulasi tar.
- $T \rightarrow K$: Tar memicu kanker.
- $R \rightarrow K$: Merokok memiliki efek langsung terhadap kanker melalui mekanisme lain (selain tar).

## 2. Hasil Identifikasi Path
### Semua Path dari R ke K
Berdasarkan fungsi `paths()` dari pustaka `dagitty`, terdapat dua jalur yang menghubungkan R dan K:
1. **$R \rightarrow K$**: Path kausal langsung (*Direct Path*).
2. **$R \rightarrow T \rightarrow K$**: Path kausal tidak langsung melalui mediator (*Indirect Path*).

### Evaluasi Backdoor Path
**Hasil:** **TIDAK ADA backdoor path.**
- *Alasan:* Backdoor path adalah jalur non-kausal yang memiliki panah mengarah masuk ke *exposure* (berbentuk $\leftarrow R$). Dalam DAG ini, seluruh panah keluar dari R ($R \rightarrow$), sehingga kedua jalur di atas sepenuhnya merupakan jalur kausal searah.

## 3. Identifikasi Total Effect
**Pertanyaan:** Apakah efek total $R \rightarrow K$ teridentifikasi tanpa adjustment?
**Hasil:** **Ya, efek total teridentifikasi tanpa adjustment (Himpunan kosong `{}`).**
- Karena tidak ada backdoor path (tidak ada *confounder*), maka estimasi efek total dari Rokok terhadap Kanker tidak memerlukan pengendalian (adjustment) variabel apapun.
- *Catatan Penting:* Variabel Tar (T) TIDAK BOLEH dikontrol saat mengukur efek total. Mengontrol T akan memblokir efek kausal yang melalui jalur $R \rightarrow T \rightarrow K$.

## 4. Estimasi Direct Effect (Efek Langsung)
**Pertanyaan:** Variabel apa yang harus dikontrol untuk mengetahui *direct effect* $R \rightarrow K$ (tanpa melalui Tar)?
**Hasil:** **Harus mengontrol variabel Tar (T).**

**Mengapa keputusan ini rumit secara kausal?**
1. **Dilema Mediator vs Confounder:** T adalah mediator, bukan *confounder*. Mengontrol mediator secara fundamental mengubah interpretasi yang dicari (dari *total effect* ke *direct effect*).
2. **Controlled vs Natural Direct Effect:** Mengontrol mediator secara analitik seringkali memaksa kita mengukur *Controlled Direct Effect* pada tingkat mediator tertentu, bukan *Natural Direct Effect* yang lebih relevan namun sulit diverifikasi asumsi *counterfactual*-nya.
3. **Risiko Bias (Intermediate Confounding):** Dalam realita, sangat mungkin ada confounder tersembunyi yang memengaruhi mediator dan outcome (T dan K). Jika hal itu terjadi, mengontrol T berisiko membuka bias *collider* atau *confounding* yang baru.
4. **Intervensi yang Tidak Realistis:** Secara praktik substantif, mustahil mengintervensi atau "menahan" kadar tar paru-paru tetap konstan sambil mengubah status kebiasaan merokok seseorang.

## 5. Verifikasi dengan Dagitty
Verifikasi struktur menggunakan R mengonfirmasi kesimpulan teoritis:
- `adjustmentSets(..., effect = "total")` menghasilkan himpunan kosong `{}`.
- `adjustmentSets(..., effect = "direct")` menghasilkan himpunan `{ T }`.
- `impliedConditionalIndependencies(dag)` menyatakan tidak ada independensi kondisional karena semua variabel dalam DAG saling terhubung.

## 6. Verifikasi melalui Simulasi Data
Simulasi Monte Carlo dengan $n=1000$ dibuat dengan koefisien:
- Direct effect $R \rightarrow K$: 0.3
- $R \rightarrow T$: 0.8
- $T \rightarrow K$: 0.5
- **Total Effect Aktual:** $0.3 + (0.8 \times 0.5) = 0.7$

**Hasil Regresi Linier:**
- **Model Total Effect (`lm(K ~ R)`)**: Koefisien R mendekati **0.7**. Hal ini membuktikan bahwa tanpa mengontrol T, kita mendapatkan estimasi efek kausal secara total.
- **Model Direct Effect (`lm(K ~ R + T)`)**: Koefisien R mendekati **0.3** dan Koefisien T mendekati **0.5**. Hal ini membuktikan bahwa dengan mengontrol mediator T, estimasi yang dihasilkan tereduksi menjadi efek langsungnya saja.

---
*Laporan ini digenerate secara otomatis berdasarkan log eksekusi R (`.Rhistory`) dan script sumber (`Latihan_5_1_Backdoor_Path.Rmd`).*

---

# Laporan Dokumentasi: Simpson's Paradox (Latihan 5.2)

**Topik:** Analisis Kausal - Simpson's Paradox, Confounder vs Mediator

## 1. Pendahuluan
Bagian ini menganalisis fenomena Simpson's Paradox dari sebuah studi medis yang mengukur persentase kesembuhan pasien yang diberikan perlakuan (*Treatment*) berdasarkan kelompok Jenis Kelamin.

**Data Studi Medis:**
| Jenis Kelamin | Treatment | Sembuh | Total | Persentase |
|---------------|-----------|--------|-------|------------|
| Pria          | Ya        | 70     | 100   | 70%        |
| Pria          | Tidak     | 60     | 100   | 60%        |
| Wanita        | Ya        | 80     | 100   | 80%        |
| Wanita        | Tidak     | 60     | 100   | 60%        |

## 2. Perhitungan Persentase
Berdasarkan data di atas, kita membandingkan hasil secara gabungan (*Marginal*) dan secara terpisah (*Stratified*):

**Persentase Marginal (Keseluruhan):**
- **Treatment Ya:** 150/200 = **75%**
- **Treatment Tidak:** 120/200 = **60%**

**Persentase Per Kelompok (Stratified):**
- **Pria:** Treatment Ya (70%) > Treatment Tidak (60%)
- **Wanita:** Treatment Ya (80%) > Treatment Tidak (60%)

## 3. Analisis Simpson's Paradox
**Pertanyaan:** Apakah ada paradoks Simpson di sini?
**Jawaban:** **TIDAK.**

**Penjelasan:**
Paradoks Simpson terjadi ketika efek pada sub-kelompok berlawanan dengan efek saat data digabung. Pada data ini, Treatment Ya konsisten memberikan persentase kesembuhan yang **lebih tinggi**, baik di sub-kelompok (Pria: 70%>60%, Wanita: 80%>60%) maupun secara keseluruhan (Marginal: 75%>60%). Karena arah efeknya searah dan konsisten, tidak ada paradoks Simpson. 
*(Catatan: Pada kasus kelas, jika pengajar bermaksud menunjukkan paradoks, kemungkinan terdapat kesalahan ketik pada data soal).*

## 4. Analisis DAG Kausal: Confounder vs Mediator
Terdapat dua skenario struktur kausal (DAG) untuk menentukan data mana yang "benar" diinterpretasikan:

### Skenario 1: Jenis Kelamin sebagai Confounder
- **Struktur:** `Jenis Kelamin -> Treatment` dan `Jenis Kelamin -> Sembuh`
- **Data yang "Benar":** **Data Per Kelompok (Stratified)**
- **Alasan Kausal:** Sebagai *confounder*, Jenis Kelamin membuka *backdoor path* yang menghasilkan bias pembaur. Untuk mendapatkan efek asli obat terhadap kesembuhan, kita *harus mengontrol* variabel confounder ini dengan melihat hasil per kelompok Jenis Kelamin.

### Skenario 2: Jenis Kelamin sebagai Mediator
- **Struktur:** `Treatment -> Jenis Kelamin -> Sembuh`
- **Data yang "Benar":** **Data Marginal (Keseluruhan)**
- **Alasan Kausal:** Sebagai *mediator*, Jenis Kelamin berada pada jalur kausal efek pengobatan. Jika kita mengontrol mediator dengan melihat data per kelompok, kita akan secara artifisial memblokir dampak obat yang menjalar melalui mediator tersebut. Oleh sebab itu, untuk mendapatkan *Total Effect* yang sebenarnya, kita *tidak boleh* mengontrol mediator; maka hasil marginal (gabungan) adalah yang paling tepat. *(Tentu saja secara biologis obat tidak bisa mengubah jenis kelamin, jadi skenario ini adalah eksperimen pemikiran).*

---
*Laporan bagian ini digenerate secara otomatis berdasarkan log eksekusi R (`Simpson’s Paradox.Rhistory`) dan script sumber (`Latihan_5_2_Simpsons_Paradox.Rmd`).*

---

# Laporan Dokumentasi: Backdoor Adjustment Manual (Latihan 5.3)

**Topik:** Menghitung Probabilitas Intervensi dan Efek Kausal Total secara Manual

## 1. Pendahuluan
Bagian ini mendemonstrasikan perhitungan kausal dari tabel probabilitas gabungan. Data observasional berupa proporsi digunakan untuk mengestimasi dampak intervensi (*do-calculus*).

**Struktur DAG:**
- **Merokok (X)** $\rightarrow$ **Kolesterol Tinggi (Z)** $\rightarrow$ **Sakit Jantung (Y)**
- **Merokok (X)** $\rightarrow$ **Sakit Jantung (Y)**

## 2. Evaluasi Backdoor Criterion
**Pertanyaan:** Apakah Kolesterol Tinggi (Z) memenuhi *backdoor criterion* untuk estimasi efek total Merokok (X) $\rightarrow$ Sakit Jantung (Y)?
**Jawaban:** **TIDAK.**
- *Penjelasan:* Salah satu syarat utama himpunan *backdoor* adalah **tidak boleh ada turunan (descendant) dari exposure**. Dalam DAG ini, Z adalah turunan langsung dari X (sebagai mediator). Selain itu, tidak terdapat *backdoor path* sama sekali pada model (tidak ada tanda panah masuk ke X). Oleh karena itu, kita **tidak boleh** mengontrol Z. Set variabel yang tepat untuk penyesuaian adalah *himpunan kosong `{}`*.

## 3. Estimasi Probabilitas Intervensi
Karena himpunan *backdoor* adalah himpunan kosong, rumus penyesuaian intervensi menyederhana menjadi probabilitas bersyarat murni:
$P(Y \mid do(X)) = P(Y \mid X)$

Dengan perhitungan marginal probabilitas:
- $P(X = Ya) = 0.50$
- $P(X = Tidak) = 0.50$

**Probabilitas Intervensi 1:**
- $P(Y = Ya \mid do(X = Ya)) = \frac{0.15}{0.50} = \mathbf{0.30}$ (atau 30%)

**Probabilitas Intervensi 2:**
- $P(Y = Ya \mid do(X = Tidak)) = \frac{0.13}{0.50} = \mathbf{0.26}$ (atau 26%)

## 4. Average Causal Effect (ACE)
**Pertanyaan:** Hitung ACE merokok terhadap sakit jantung.
- $ACE = P(Y = Ya \mid do(X = Ya)) - P(Y = Ya \mid do(X = Tidak))$
- $ACE = 0.30 - 0.26 = \mathbf{0.04}$

*Artinya, merokok secara kausal meningkatkan probabilitas risiko terkena sakit jantung sebesar 4 poin persentase pada populasi tersebut.*

## 5. Estimasi Naif vs Kausal (Bias)
**Pertanyaan:** Apakah ada bias jika dibandingkan dengan estimasi naif observasional $P(Y \mid X=Ya) - P(Y \mid X=Tidak)$?
**Jawaban:** **TIDAK ADA BIAS.**
- Karena pada DAG asumsi ini **tidak ada *confounder*** (tidak ada *backdoor path* sama sekali), korelasi observasional murni sama persis dengan probabilitas kausalnya.
- Estimasi Naif = $0.30 - 0.26 = \mathbf{0.04}$.
- Hal ini menegaskan bahwa tanpa bias *confounding*, ungkapan "korelasi bukan kausasi" tertunda, dan estimasi observasi (naif) bernilai ekuivalen secara kausal.

---
*Laporan bagian ini digenerate secara otomatis berdasarkan log eksekusi R (`Backdoor Adjustment Manual.Rhistory`) dan script sumber (`Latihan_5_3_Backdoor_Adjustment.Rmd`).*

---

# Laporan Dokumentasi: Counterfactual Individu (Latihan 5.4)

**Topik:** Menghitung *Individual Treatment Effect* (ITE) menggunakan *Structural Causal Model* (SCM)

## 1. Pendahuluan
Bagian ini mendemonstrasikan evaluasi kausal kontrafaktual di level individu (bernama Sari, Usia 35 tahun) menggunakan SCM yang diberikan. Observasi aktual Sari adalah ia mengonsumsi obat ($X=1$) dan kesehatannya membaik sebesar 20 poin ($Y=20$).

**Model SCM:**
- $Usia = 35$ (konstanta spesifik individu)
- $X = U_X$
- $Y = 12X - 0.1(Usia)^2 + U_Y$ (dengan $U_Y \sim N(0, 3)$)

## 2. Langkah 1: Abduction (Deduksi Variabel Eksogen)
**Tujuan:** Mencari nilai kondisi bawaan yang tidak teramati ($U_Y$) spesifik untuk Sari menggunakan data observasinya.

$$ Y = 12X - 0.1(Usia)^2 + U_Y $$
$$ 20 = 12(1) - 0.1(35)^2 + U_Y $$
$$ 20 = 12 - 122.5 + U_Y $$
$$ 20 = -110.5 + U_Y $$
$$ U_Y = \mathbf{130.5} $$

## 3. Langkah 2: Action (Intervensi)
**Tujuan:** Mengubah persamaan struktural dalam model untuk mencerminkan skenario kontrafaktual (bayangkan Sari tidak mengonsumsi obat).
- Model diintervensi dengan menetapkan $do(X = 0)$.
- Atribut konstan Sari tetap dipertahankan: $Usia = 35$ dan $U_Y = 130.5$.

## 4. Langkah 3: Prediction (Prediksi Kontrafaktual)
**Tujuan:** Menghitung nilai $Y$ (outcome) kontrafaktual jika $X = 0$.

$$ Y(0) = 12(0) - 0.1(35)^2 + 130.5 $$
$$ Y(0) = 0 - 122.5 + 130.5 = \mathbf{8.0} $$

*Nilai kontrafaktual kesehatannya (jika ia tidak minum obat) adalah 8.0 poin.*

## 5. Individual Treatment Effect (ITE) vs Average Causal Effect (ATE)
**Pertanyaan:** Berapa ITE Sari dan mengapa berbeda dengan ATE?

**Perhitungan ITE:**
$$ ITE = Y(1) - Y(0) = 20 - 8 = \mathbf{12} $$

**Perhitungan ATE:**
$$ ATE = E[Y \mid do(X=1)] - E[Y \mid do(X=0)] $$
$$ ATE = E[12(1) - 0.1(Usia)^2 + U_Y] - E[12(0) - 0.1(Usia)^2 + U_Y] $$
Karena $E[U_Y] = 0$, maka selisih matematisnya adalah **12**.

**Kesimpulan Teoritis (Mengapa Berbeda?):**
Secara numerik pada *kasus spesifik ini*, ITE Sari (12) bernilai persis SAMA dengan ATE (12). Ini karena SCM pembentuk $Y$ bersifat murni linier-aditif tanpa efek interaksi (misalnya tidak ada suku $X \times Usia$ atau $X \times U_Y$). 

Namun, secara konsep dasar kausal, keduanya sangat berbeda sifat:
- **ITE** bersifat *deterministik* untuk satu orang. Ia menggunakan distribusi kondisional eksogen hasil observasi unik individu ($U_Y = 130.5$) dan tidak terpengaruh oleh rata-rata populasi.
- **ATE** bersifat *probabilistik* untuk populasi. Ia didapat dengan merata-ratakan atau menghapus keragaman eksogen ($E[U_Y]=0$).
- Jika efek dari pengobatan bergantung pada usia (misal: SCM mengandung koefisien non-linier antara $X$ dan $Usia$), maka ITE Sari pasti akan berbeda dengan ATE populasinya (efek pengobatan menjadi heterogen/berbeda-beda tiap orang).

---
*Laporan bagian ini digenerate secara otomatis berdasarkan log eksekusi R (`Counterfactual Individu.Rhistory`) dan script sumber (`Latihan_5_4_Counterfactual_Individu.Rmd`).*

---

# Laporan Dokumentasi: CausalImpact — Interpretasi Output (Latihan 5.5)

**Topik:** Evaluasi Dampak Kausal menggunakan *Bayesian Structural Time-Series* (BSTS) pada Data Runtun Waktu

## 1. Pendahuluan
Bagian ini menganalisis dampak intervensi pada data *time-series* menggunakan pustaka `CausalImpact` di R. Berdasarkan proses *Data Generating Process* (DGP), variabel kontrol dibuat mengikuti proses *random walk*, dan variabel respons dipengaruhi oleh variabel kontrol dengan tambahan efek kausal sebesar **+8** (berlaku mulai periode *post-intervention* di $t=150$ hingga $t=200$).

## 2. Interpretasi Hasil Model
Setelah menjalankan pemodelan *CausalImpact*, diperoleh insight berikut:

**1. Estimasi Rata-Rata Efek Intervensi per Periode:**
Estimasi rata-rata absolut per periode bernilai **~8.0**. Hal ini terkonfirmasi dari desain eksperimen (*ground truth*) di mana nilai respons pada observasi ke-150 ke atas memang disuntikkan nilai +8. Model berhasil menangkap besaran kausal ini dengan presisi tinggi pada kolom *Absolute effect (Average)*.

**2. Total Kumulatif Efek Selama Post-Intervensi:**
Periode pasca-intervensi mencakup 51 minggu (dari $t=150$ hingga $t=200$). Total efek kumulatifnya adalah selisih akumulasi antara respons aktual dan prediksi kontrafaktual:
$51 \text{ minggu} \times 8 \text{ poin/minggu} \approx \mathbf{408} \text{ poin}$. Nilai eksaknya muncul pada kolom *Cumulative effect*.

**3. Signifikansi Statistik:**
Efek intervensi tersebut sangat **SIGNIFIKAN**. Terdapat dua bukti kuat dari laporan model:
- **Interval Kepercayaan (95% CI):** Rentang interval prediksi efek tidak mencakup angka 0.
- **Tail-area probability (p-value):** Nilai p-value sangat kecil (misal $p = 0.001$), menandakan probabilitas pergerakan respons sebesar ini terjadi secara kebetulan adalah hampir mustahil.

**4. Pentingnya Variabel Kontrol (*Covariate*):**
Variabel kontrol tidak terkena dampak intervensi dan berperan sebagai "jangkar" pembanding. Tanpa kontrol, model BSTS hanya akan melihat pergerakan musiman (*random walk*) dari respons. Dengan menyertakan kontrol yang berkorelasi positif (0.7), model dapat mendeduksi arah tren kontrafaktual yang wajar: *"Jika kontrol bergerak ke A, respons seharusnya ke B. Jika respons faktanya ada di B+8, maka selisih 8 tersebut adalah efek murni intervensi."*

## 3. Modifikasi: Dampak Bias Post-Treatment
**Apa yang terjadi jika variabel kontrol ikut terpengaruh intervensi di t=150?**

Jika eksperimen dimodifikasi sedemikian rupa sehingga kontrol juga ikut berubah (misal: diberi tambahan +10), maka asumsi paling krusial dalam *CausalImpact* (**variabel kontrol harus kebal intervensi**) telah **dilanggar**.

**Dampak pada Output Model:**
1. **Prediksi Kontrafaktual Meleset (*Overestimate*):** Model akan menganggap tren dasar (*baseline*) memang secara alami sedang melonjak tinggi karena melihat variabel kontrolnya melonjak.
2. **Underestimasi Efek Kausal:** Akibat prediksi kontrafaktual yang terlalu tinggi, jarak antara hasil aktual dan prediksi menjadi sangat sempit (atau bahkan nol). Efek intervensi seolah "terserap" oleh pergerakan palsu kontrol.
3. Laporan summary `CausalImpact` akan menunjukkan bahwa **efek menjadi tidak signifikan**, mereduksi temuan kausal menjadi sekadar artifak korelasi palsu (bias *collider*).

---
*Laporan bagian ini digenerate secara otomatis berdasarkan log eksekusi R (`CausalImpact — Interpretasi Output.Rhistory`) dan script sumber (`Latihan_5_5_CausalImpact.Rmd`).*

---

# Laporan Dokumentasi: Mediation Analysis Lengkap (Latihan 5.6)

**Topik:** Analisis Mediasi Kausal Menggunakan `mediation` Package pada Eksperimen Framing Berita

## 1. Identifikasi Variabel Utama
Berdasarkan dokumentasi dataset `framing` (eksperimen Brader, Valentino, dan Suhay, 2008), kita mendefinisikan tiga variabel utama untuk analisis mediasi kausal:
- **Treatment (X) = `treat`:** Indikator paparan *framing* berita negatif terkait imigrasi (1 = Terpapar, 0 = Kontrol).
- **Mediator (M) = `emo`:** Tingkat emosi kecemasan peserta setelah membaca berita (skala kontinu).
- **Outcome (Y) = `cong_mesg`:** Perilaku politik partisipatif, yakni kesediaan peserta mengirim pesan ke anggota kongres mengenai isu imigrasi (1 = Ya, 0 = Tidak).

*Hipotesis Dasar:* Berita dengan framing negatif (X) memicu partisipasi politik (Y) secara tidak langsung karena berita tersebut membangkitkan rasa cemas (M).

## 2. Pendekatan Regresi Baron-Kenny
Langkah tradisional Baron-Kenny mencakup tiga model regresi (dikontrol oleh *covariates* demografis seperti `age, educ, gender, income`):
1. **Model Total Effect (c):** `cong_mesg ~ treat + covariates` (Menghasilkan efek total X $\rightarrow$ Y).
2. **Model Mediator (a):** `emo ~ treat + covariates` (Menghasilkan efek X $\rightarrow$ M).
3. **Model Outcome (b, c'):** `cong_mesg ~ treat + emo + covariates` (Menghasilkan efek M $\rightarrow$ Y dan *direct effect* X $\rightarrow$ Y).

*Hasil Estimasi Baron-Kenny menunjukkan bahwa jalur `a` dan `b` positif dan signifikan, sedangkan jalur `c'` (direct effect) mendekati nol.*

## 3. Mediation Analysis dengan Bootstrapping
Analisis lebih robust dilakukan menggunakan fungsi `mediate()` dengan 1000 iterasi *bootstrap*. Output metrik utama yang didapat:
- **ACME (Average Causal Mediation Effect):** Estimasi efek tidak langsung ($a \times b$). Nilainya $\sim 0.08$. Framing negatif meningkatkan probabilitas pengiriman pesan sebesar $\sim 8\%$ **hanya melalui pemicuan emosi**.
- **ADE (Average Direct Effect):** Estimasi efek langsung ($c'$). Nilainya sangat kecil ($\sim 0.01$) dan **tidak signifikan**.
- **Total Effect:** Penjumlahan efek (ACME + ADE), bernilai $\sim 0.09$.

## 4. Proportion Mediated dan Interpretasinya
- **Proportion Mediated:** Menghitung rasio ACME terhadap Total Effect. Karena $ADE$ nyaris nol, proporsi yang termediasi bernilai sangat tinggi, yaitu berkisar **85% hingga nyaris 100%**.
- **Interpretasi:** Hal ini membuktikan terjadinya *Full Mediation* (Mediasi Penuh). Emosi kecemasan adalah "mesin penggerak" utama partisipasi politik dalam eksperimen ini. Tanpa melalui kebangkitan emosi, paparan *framing* negatif sama sekali tidak berdampak memotivasi orang untuk berpartisipasi dalam politik.

## 5. Sensitivity Analysis
Uji sensitivitas mengevaluasi seberapa rentan kesimpulan mediasi terhadap asumsi "tidak ada pembaur yang tak teramati" (*Sequential Ignorability*).
- **Temuan Uji (Nilai $\rho$):** Output `medsens` dan plot menunjukkan bahwa ACME akan menyentuh nol (tidak signifikan) ketika parameter korelasi error $\rho$ berada di kisaran **0.3 hingga 0.4**.
- **Implikasi Kausal:** Jika terdapat variabel "tersembunyi" (misalnya sifat agresivitas bawaan responden) yang membuat peserta mudah merasa cemas sekaligus lebih vokal berpolitik—sehingga menyebabkan error term di model M dan Y berkorelasi sebesar 0.35—maka seluruh kesimpulan ACME akan gugur. Kesimpulannya, klaim efek mediasi emosi ini cukup sensitif/rentan terhadap kemungkinan adanya *unobserved confounder*.

## 6. Diagram Path (Koefisien Causal)
Diagram kausal mengilustrasikan mekanisme pengaruh $treat \rightarrow emo \rightarrow cong\_mesg$:

1. **Jalur $a$ ($X \rightarrow M$):** Bernilai positif kuat. Framing berita sukses memicu *anxiety* (kecemasan).
2. **Jalur $b$ ($M \rightarrow Y$):** Bernilai positif kuat. Rasa *anxiety* yang tinggi mendorong keinginan mengirim pesan ke kongres.
3. **Jalur $c'$ ($X \rightarrow Y$, Direct Effect):** Bernilai $\approx 0$. Pesan rasional dari framing berita tidak memotivasi partisipasi politik sama sekali jika tanpa adanya emosi kecemasan.

---
*Laporan bagian ini digenerate secara otomatis berdasarkan log eksekusi R (`Mediation Analysis Lengkap.Rhistory`) dan script sumber (`Latihan_5_6_Mediation_Analysis.Rmd`).*
