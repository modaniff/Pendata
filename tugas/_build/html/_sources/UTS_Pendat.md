# UTS Analisa Data Kesuburan Tanah
## NIM 240411100035
## Nama: Mohammad Dani Ferdiansyah
## Mata Kuliah Penambangan Data B
## Dosen Pengampu: Mula'ab, S.Si., M.Kom


# SOAL UTS
## Pendahuluan

Kesuburan tanah merupakan faktor penentu utama produktivitas pertanian. Analisis ini bertujuan mengklasifikasikan kondisi tanah sebagai **Subur** atau **Tidak Subur** menggunakan algoritma **K-Nearest Neighbors (KNN)** berdasarkan parameter fisik dan kimia tanah.

---

## Informasi Dataset

> **Sumber Dataset:** [Google Spreadsheet - Dataset Kesuburan Tanah](https://docs.google.com/spreadsheets/d/1_VTOGjavAI1Axd4gFRhXrIKRVVjY9zvM/edit?gid=1558601676)

### Deskripsi Umum

| Atribut | Keterangan |
|---------|------------|
| **Jumlah Sampel** | 2.000 baris |
| **Jumlah Fitur** | 10 fitur (9 numerik, 1 kategorikal) |
| **Jumlah Kelas** | 2 kelas |
| **Target / Label** | Subur / Tidak Subur |
| **Missing Values** | Ada (beberapa kolom) |

### Distribusi Kelas

```
Subur        : 1.000 sampel (50%)
Tidak Subur  : 1.000 sampel (50%)
Total        : 2.000 sampel
```

Dataset ini **balanced** (seimbang) - tidak ada bias jumlah kelas.

### Penjelasan Fitur

| No | Fitur | Satuan | Deskripsi | Nilai Subur | Nilai Tidak Subur |
|----|-------|--------|-----------|-------------|-------------------|
| 1 | **pH Tanah** | Skala 0-14 | Keasaman/kebasaan tanah | 6,0 - 7,5 | < 5,4 atau > 7,6 |
| 2 | **N Total** | % | Kandungan nitrogen total | 0,21 - 0,50% | 0,01 - 0,20% |
| 3 | **P Tersedia** | ppm | Fosfor tersedia | 15 - 60 ppm | 1 - 14 ppm |
| 4 | **K Tersedia** | meq/100g | Kalium tersedia | 0,30 - 0,80 | 0,05 - 0,29 |
| 5 | **C Organik** | % | Karbon organik | 2,0 - 5,0% | 0,2 - 1,9% |
| 6 | **KTK** | meq/100g | Kapasitas Tukar Kation | 20 - 45 | 5 - 19 |
| 7 | **Kejenuhan Basa** | % | Persentase kation basa | 60 - 100% | 10 - 59% |
| 8 | **Tekstur Tanah** | Kategorikal | Komposisi partikel tanah | Lempung, dll | Pasir, Liat, dll |
| 9 | **Kadar Air** | % | Persentase kadar air | 25 - 45% | < 20% atau > 55% |
| 10 | **Bulk Density** | g/cm3 | Kerapatan tanah | 0,9 - 1,2 | 1,4 - 1,9 |

### Definisi Kelas

| Label | Deskripsi |
|-------|-----------|
| **Subur** | Tanah dengan kondisi fisik, kimia, dan biologi optimal: pH seimbang, unsur hara cukup, tekstur ideal, struktur tanah baik. |
| **Tidak Subur** | Tanah dengan satu atau lebih kondisi pembatas: pH ekstrem, kekurangan unsur hara, tekstur buruk, kadar air tidak ideal, atau bulk density tinggi. |

---

# Jawaban UTS

---

## Workflow KNIME

Untuk mengerjakan soal ini, saya menggunakan KNIME Analytics Platform dengan menyusun node-node berikut secara berurutan:

```
Excel Reader -> Column Filter -> Missing Value -> One to Many -> Normalizer -> Table Partitioner -> K Nearest Neighbor -> Scorer -> Table View
```

---

## Penjelasan Setiap Node

### 1. Excel Reader

Node pertama yang saya gunakan adalah **Excel Reader**. Saya pakai node ini karena dataset yang diberikan berformat `.xlsx`, jadi saya load langsung dari file Excel ke dalam KNIME.

Setelah saya konfigurasi path file-nya, node ini menghasilkan tabel data mentah berisi 2.000 baris dengan 11 kolom, yaitu 10 fitur dan 1 kolom label. Dari sini saya bisa lihat bahwa beberapa kolom masih ada yang kosong, tanda dataset memang punya missing value yang perlu ditangani.

![Tampilan node Excel Reader setelah berhasil membaca dataset kesuburan tanah](Assets/UTS/Read-Data-CSV.png)

Dari gambar di atas, saya bisa konfirmasi bahwa data sudah berhasil masuk ke KNIME. Semua kolom fitur seperti pH Tanah, N Total, P Tersedia, dan lainnya sudah terbaca dengan tipe data yang sesuai.

---

### 2. Column Filter

Setelah data berhasil dibaca, saya sambungkan ke node **Column Filter**. Tujuannya satu: membuang kolom **ID** yang tidak saya butuhkan dalam proses klasifikasi.

Kolom ID itu cuma nomor urut baris, tidak ada informasi yang relevan soal kondisi tanah di sana. Kalau saya biarkan masuk ke model KNN, jaraknya bisa terganggu karena model akan ikut mempertimbangkan angka urut yang tidak ada hubungannya dengan kesuburan tanah.

Kolom yang saya pertahankan di panel **Includes**:
- pH Tanah
- N Total (%)
- P Tersedia (ppm)
- K Tersedia (meq/100g)
- C Organik (%)
- KTK (meq/100g)
- Kejenuhan Basa (%)
- Tekstur Tanah
- Kadar Air (%)
- Bulk Density (g/cm3)
- Label

![Konfigurasi node Column Filter, kolom ID sudah dipindah ke panel Excludes](Assets/UTS/Column-Filter.png)

Dari gambar di atas terlihat kolom `ID` sudah berhasil saya pindahkan ke panel **Excludes**, jadi tidak akan ikut ke proses selanjutnya.

---

### 3. Missing Value

Setelah kolom ID dibuang, saya sambungkan ke node **Missing Value** untuk menangani data yang hilang. Dari hasil eksplorasi awal, saya menemukan missing value di beberapa kolom, yaitu:

- N Total (%)
- P Tersedia (ppm)
- K Tersedia (meq/100g)
- C Organik (%)
- Tekstur Tanah
- Kadar Air (%)
- Bulk Density (g/cm3)

Untuk mengisinya, saya menggunakan dua pendekatan:
- Kolom **numerik**: saya isi dengan nilai **mean** (rata-rata) dari kolom tersebut.
- Kolom **kategorikal** yaitu Tekstur Tanah: saya isi dengan nilai yang paling sering muncul (**most frequent value**).

![Konfigurasi node Missing Value untuk menangani data kosong](Assets/UTS/missing-value.png)

Setelah node ini selesai dijalankan, semua baris data sudah bersih dari nilai kosong dan siap saya proses ke tahap berikutnya.

---

### 4. One to Many

Selanjutnya saya tambahkan node **One to Many** untuk mengubah kolom kategorikal menjadi representasi numerik berupa kolom-kolom biner (dummy variable).

Kolom yang saya ubah adalah **Tekstur Tanah**, yang berisi nilai seperti:
- Lempung
- Lempung Berpasir
- Lempung Berliat
- Pasir
- Liat
- Debu

Saya perlu melakukan ini karena algoritma KNN bekerja dengan menghitung jarak antar titik data, sedangkan nilai teks tidak bisa langsung dihitung jaraknya. Dengan node One to Many, satu kolom `Tekstur Tanah` saya pecah menjadi beberapa kolom biner, misalnya:

| Tekstur Tanah_Lempung | Tekstur Tanah_Pasir | Tekstur Tanah_Liat | ... |
|-----------------------|---------------------|---------------------|-----|
| 1 | 0 | 0 | ... |
| 0 | 1 | 0 | ... |

![Konfigurasi node One to Many untuk mengubah Tekstur Tanah menjadi kolom biner](Assets/UTS/one-to-many.png)

Dengan begitu, fitur Tekstur Tanah sudah bisa ikut masuk ke perhitungan jarak dalam KNN.

---

### 5. Normalizer

Setelah semua fitur sudah berbentuk numerik, saya tambahkan node **Normalizer** untuk menyeragamkan skala semua fitur menggunakan metode **Min-Max Normalization**.

Rumus yang digunakan:

$$
x' = \frac{x - x_{min}}{x_{max} - x_{min}}
$$

Kenapa saya perlu normalisasi? Karena setiap fitur punya satuan dan rentang nilai yang berbeda. Misalnya, P Tersedia nilainya bisa puluhan (ppm), sementara N Total hanya berkisar antara 0,01 sampai 0,50 (%). Kalau tidak saya samakan skalanya, fitur dengan angka besar akan terlalu mendominasi perhitungan jarak, sementara fitur yang nilainya kecil jadi kurang diperhitungkan.

Setelah normalisasi, semua nilai fitur berada di rentang 0 sampai 1, jadi pengaruh setiap fitur lebih seimbang.

![Konfigurasi node Normalizer dengan metode Min-Max](Assets/UTS/normalizer.png)

---

### 6. Table Partitioner

Sebelum saya masukkan data ke model KNN, saya membagi data terlebih dahulu menggunakan node **Table Partitioner**.

Konfigurasi yang saya pakai:
- First partition type: **Relative (%)**
- Relative size: **80**
- Sampling strategy: **Random**

Dari total 2.000 data, hasil pembagiannya adalah:
- **1.600 data** masuk ke partisi pertama sebagai data latih (training)
- **400 data** masuk ke partisi kedua sebagai data uji (testing)

![Konfigurasi Table Partitioner dengan pembagian 80% training dan 20% testing](Assets/UTS/table-partioner-80%.png)

Saya membagi data seperti ini agar saat saya evaluasi model nanti, data yang diuji adalah data yang belum pernah dilihat model, sehingga hasilnya lebih objektif.

---

### 7. K Nearest Neighbor

Ini adalah node utama yang saya pakai untuk melakukan klasifikasi. Node **K Nearest Neighbor** melatih model menggunakan 1.600 data latih dari partisi pertama, lalu memprediksi kelas pada 400 data uji dari partisi kedua.

Konfigurasi yang saya set:
- Column with class labels: **Label**
- Number of neighbors (k): **5**

Cara kerja KNN yang saya pahami: setiap data uji akan dicari sebanyak k data paling dekat dari data latih. Kelas yang paling banyak muncul di antara k tetangga tersebut akan menjadi hasil prediksi. Di sini saya pakai k = 5, artinya saya lihat 5 tetangga terdekat untuk menentukan apakah tanah tersebut **Subur** atau **Tidak Subur**.

Rumus jarak yang dipakai adalah Euclidean Distance:

$$
d(x, y) = \sqrt{\sum_{i=1}^{n}(x_i - y_i)^2}
$$

![Konfigurasi node K Nearest Neighbor dengan k = 5](Assets/UTS/knn.png)

Setelah node ini selesai, output-nya berupa tabel lengkap dengan tambahan kolom `Class [kNN]` yang berisi hasil prediksi kelas untuk setiap baris data uji.

---

### 8. Scorer

Setelah model selesai berjalan, saya sambungkan ke node **Scorer** untuk mengukur seberapa baik prediksi model dibandingkan dengan label aslinya.

Dari node ini saya mendapatkan:
- Confusion matrix
- Accuracy statistics
- Nilai Precision, Recall, dan F-measure

Perlu saya catat bahwa di KNIME, F1-Score ditampilkan dengan nama **F-measure**, bukan F1-Score seperti yang umum dikenal.

![Output node Scorer menampilkan confusion matrix dan statistik evaluasi](Assets/UTS/scorer.png)

---

### 9. Table View

Node terakhir yang saya tambahkan adalah **Table View**. Saya pakai node ini untuk menampilkan hasil evaluasi akhir dalam bentuk tabel yang lebih mudah saya baca.

Kolom yang ditampilkan:
- Recall
- Precision
- F-measure
- Accuracy

![Tampilan Table View menampilkan hasil metrik evaluasi akhir](Assets/UTS/table-view.png)

---

## Hasil Evaluasi Model

Berdasarkan output yang saya peroleh dari node Scorer dan Table View, hasil evaluasi model KNN pada dataset kesuburan tanah ini adalah:

| Metrik | Nilai |
|--------|-------|
| **Accuracy** | 1.00 (100%) |
| **Precision** | 1.00 (100%) |
| **Recall** | 1.00 (100%) |
| **F1-Score (F-measure)** | 1.00 (100%) |

---

## Confusion Matrix

Dari hasil node Scorer, saya mendapatkan confusion matrix sebagai berikut:

| Aktual / Prediksi | Tidak Subur | Subur |
|-------------------|-------------|-------|
| **Tidak Subur** | 200 | 0 |
| **Subur** | 0 | 200 |

Artinya:
- 200 data yang aslinya Tidak Subur berhasil saya prediksi dengan benar sebagai Tidak Subur
- 200 data yang aslinya Subur berhasil saya prediksi dengan benar sebagai Subur
- Tidak ada satupun data yang salah prediksi

---

## Perhitungan Metrik Evaluasi

Berikut saya hitung masing-masing metrik secara manual. Saya anggap kelas positif adalah **Subur**.

Nilai dari confusion matrix yang saya dapat:
- **TP** (True Positive) = 200, yaitu data yang saya prediksi Subur dan memang aslinya Subur
- **TN** (True Negative) = 200, yaitu data yang saya prediksi Tidak Subur dan memang aslinya Tidak Subur
- **FP** (False Positive) = 0, tidak ada data Tidak Subur yang salah saya prediksi sebagai Subur
- **FN** (False Negative) = 0, tidak ada data Subur yang salah saya prediksi sebagai Tidak Subur

### Accuracy

Rumus:

$$
Accuracy = \frac{TP + TN}{TP + TN + FP + FN}
$$

Saya substitusikan nilainya:

$$
Accuracy = \frac{200 + 200}{200 + 200 + 0 + 0} = \frac{400}{400} = 1.00
$$

Hasil: **Accuracy = 100%**

### Precision

Rumus:

$$
Precision = \frac{TP}{TP + FP}
$$

Saya substitusikan:

$$
Precision = \frac{200}{200 + 0} = 1.00
$$

Hasil: **Precision = 100%**

### Recall

Rumus:

$$
Recall = \frac{TP}{TP + FN}
$$

Saya substitusikan:

$$
Recall = \frac{200}{200 + 0} = 1.00
$$

Hasil: **Recall = 100%**

### F1-Score

Rumus:

$$
F1\text{-}Score = \frac{2 \times Precision \times Recall}{Precision + Recall}
$$

Saya substitusikan:

$$
F1\text{-}Score = \frac{2 \times 1.00 \times 1.00}{1.00 + 1.00} = \frac{2}{2} = 1.00
$$

Hasil: **F1-Score = 100%**

---

## Interpretasi Hasil

Dari hasil evaluasi yang saya peroleh, model KNN yang saya bangun berhasil memprediksi semua 400 data uji dengan benar tanpa ada satupun kesalahan klasifikasi.

Semua metrik, yaitu accuracy, precision, recall, dan F1-score, semuanya mencapai nilai 1.00 atau 100%. Menurut saya hal ini terjadi karena pola perbedaan antara tanah yang Subur dan Tidak Subur pada dataset ini cukup jelas dan terpisah, sehingga KNN dengan k = 5 bisa menangkapnya dengan sangat baik.

---

## Kesimpulan

Dari workflow KNIME yang saya buat, saya berhasil menyelesaikan analisis data kesuburan tanah secara end-to-end mulai dari membaca data sampai mendapatkan hasil evaluasi model. Berikut rangkuman tahapan yang saya kerjakan:

1. **Excel Reader**: saya baca dataset dari file .xlsx langsung ke KNIME
2. **Column Filter**: saya buang kolom ID karena tidak relevan untuk klasifikasi
3. **Missing Value**: saya isi nilai kosong, pakai mean untuk kolom numerik dan most frequent value untuk kolom Tekstur Tanah
4. **One to Many**: saya ubah kolom Tekstur Tanah dari kategorikal menjadi kolom-kolom biner agar bisa diproses KNN
5. **Normalizer**: saya samakan skala semua fitur numerik ke rentang 0 sampai 1 menggunakan Min-Max Normalization
6. **Table Partitioner**: saya bagi data menjadi 80% training (1.600 data) dan 20% testing (400 data)
7. **K Nearest Neighbor**: saya latih model KNN dengan k = 5
8. **Scorer**: saya bandingkan label asli dengan prediksi model untuk mendapatkan metrik evaluasi
9. **Table View**: saya tampilkan hasil evaluasi akhir dalam bentuk tabel

Model KNN dengan k = 5 yang saya bangun menghasilkan nilai Accuracy, Precision, Recall, dan F1-Score masing-masing sebesar **100%**. Hasil ini menunjukkan bahwa model berhasil mengklasifikasikan seluruh 400 data uji dengan benar tanpa ada satupun kesalahan.

---

