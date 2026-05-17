# Random Forest

## Contents

- [Dataset](#dataset)
- [Implementasi Pada KNIME](#implementasi-pada-knime)
  - [Workflow](#workflow)
  - [Partisi](#partisi)
  - [Column Filter](#column-filter)
  - [Training: Python Script Random Forest](#training-python-script-random-forest)
  - [Testing: Python Script Prediksi](#testing-python-script-prediksi)
  - [Scorer](#scorer)
- [Kesimpulan](#kesimpulan)

---

## Dataset

Dataset yang digunakan adalah **Dataset Prediksi Kelulusan Mahasiswa**. Dataset ini dibuat untuk melakukan klasifikasi status kelulusan mahasiswa berdasarkan beberapa atribut akademik dan non-akademik.

Tujuan dari dataset ini adalah memprediksi apakah mahasiswa termasuk dalam kategori:

- **Tepat Waktu**
- **Tidak Tepat Waktu**

Dataset ini memiliki:

- **500 baris data**
- **7 fitur utama**
- **1 label/target** bernama `Status_Kelulusan`

Berikut fitur yang digunakan pada dataset:

| No | Nama Fitur | Tipe Data | Deskripsi | Contoh Nilai |
| --- | --- | --- | --- | --- |
| 1 | ID_Mahasiswa | String | Kode unik mahasiswa | MHS001 |
| 2 | IPK | Numerik | Indeks Prestasi Kumulatif mahasiswa | 3.45 |
| 3 | Kehadiran | Numerik | Persentase kehadiran mahasiswa | 85 |
| 4 | Nilai_Tugas | Numerik | Nilai tugas mahasiswa | 88 |
| 5 | Nilai_UTS | Numerik | Nilai Ujian Tengah Semester | 80 |
| 6 | Nilai_UAS | Numerik | Nilai Ujian Akhir Semester | 84 |
| 7 | Organisasi | Kategorikal | Status keaktifan organisasi | Aktif / Tidak Aktif |
| 8 | Status_Kelulusan | Kategorikal | Label target prediksi | Tepat Waktu / Tidak Tepat Waktu |

Label yang digunakan:

| Label | Keterangan |
| --- | --- |
| Tepat Waktu | Mahasiswa diprediksi lulus tepat waktu |
| Tidak Tepat Waktu | Mahasiswa diprediksi tidak lulus tepat waktu |

---

## Implementasi Pada KNIME

Workflow ini dirancang menggunakan KNIME untuk membangun model klasifikasi **Random Forest** berbasis Python menggunakan library `scikit-learn`.

Model dibuat menggunakan node **Python Script**, sehingga proses training dan testing dilakukan melalui script Python di dalam KNIME.

Tujuan dari workflow ini adalah memprediksi `Status_Kelulusan` mahasiswa berdasarkan IPK, kehadiran, nilai tugas, nilai UTS, nilai UAS, dan status organisasi.

---

### Workflow

```{figure} ../img/random-forest-kelulusan/workflow random forest.png
:name: workflow-rf-kelulusan
:align: center

Workflow KNIME untuk klasifikasi Random Forest prediksi kelulusan mahasiswa

Workflow terdiri dari node-node berikut:

1. **CSV Reader** digunakan untuk membaca dataset.
2. **Column Filter** digunakan untuk membuang kolom yang tidak diperlukan.
3. **Table Partitioner** digunakan untuk membagi data training dan testing.
4. **Python Script** pertama digunakan untuk melatih model Random Forest.
5. **Python Script** kedua digunakan untuk melakukan prediksi.
6. **Scorer** digunakan untuk mengevaluasi hasil prediksi.

Alur workflow:

```text
CSV Reader
    ↓
Column Filter
    ↓
Table Partitioner
   ├── Python Script Training
   └── Python Script Testing / Prediction
                ↓
              Scorer
```

---

### CSV Reader

```{figure} ../img/random-forest-kelulusan/csv reader forest.png
:name: csv-reader-rf-kelulusan
:align: center

Node CSV Reader untuk membaca dataset
```

Node **CSV Reader** digunakan untuk membaca file dataset:

```text
dataset_kelulusan_mahasiswa.csv
```

Langkah konfigurasi:

1. Tambahkan node **CSV Reader**.
2. Pilih file `dataset_kelulusan_mahasiswa.csv`.
3. Pastikan delimiter menggunakan koma `,`.
4. Klik **Apply**.
5. Klik **OK**.
6. Jalankan node dengan memilih **Execute**.

Setelah node dijalankan, data dari CSV akan masuk ke dalam workflow KNIME.

---

### Column Filter

```{figure} ../img/random-forest-kelulusan/collum filter forest.png
:name: column-filter-rf-kelulusan
:align: center
Konfigurasi Column Filter

```

Node **Column Filter** digunakan untuk membuang kolom yang tidak diperlukan pada proses klasifikasi.

Pada dataset ini, kolom `ID_Mahasiswa` tidak digunakan sebagai fitur prediksi karena hanya berfungsi sebagai identitas mahasiswa.

Kolom yang dibuang:

| Kolom | Alasan |
| --- | --- |
| ID_Mahasiswa | Hanya sebagai identitas, bukan atribut prediksi |

Kolom yang tetap digunakan:

| No | Kolom |
| --- | --- |
| 1 | IPK |
| 2 | Kehadiran |
| 3 | Nilai_Tugas |
| 4 | Nilai_UTS |
| 5 | Nilai_UAS |
| 6 | Organisasi |
| 7 | Status_Kelulusan |

---

### Partisi

```{figure} ../img/random-forest-kelulusan/partisi forest.png
:name: partisi-rf-kelulusan
:align: center

Konfigurasi Table Partitioner
```

Node **Table Partitioner** digunakan untuk membagi dataset menjadi data training dan data testing.

Data dibagi dengan proporsi:

- **80%** → data training
- **20%** → data testing

Konfigurasi:

| Parameter | Nilai |
| --- | --- |
| Partitioning Method | Relative |
| Training Data | 80% |
| Testing Data | 20% |
| Sampling | Stratified Sampling |
| Stratification Column | Status_Kelulusan |

Keterangan:

- Output atas digunakan sebagai **data training**.
- Output bawah digunakan sebagai **data testing**.

---

### Training: Python Script Random Forest

```{figure} ../img/random-forest-kelulusan/Training Python Script.png
:name: py-training-rf-kelulusan
:align: center

Konfigurasi Python Script untuk Training
```

Node **Python Script** pertama digunakan untuk melakukan training model Random Forest.

Node ini menerima data training dari output atas **Table Partitioner**.

Script yang digunakan:

```python
import knime.scripting.io as knio
import pandas as pd
import pickle
import os

from sklearn.ensemble import RandomForestClassifier
from sklearn.preprocessing import LabelEncoder

# Ambil data training dari KNIME
train_df = knio.input_tables[0].to_pandas()

# Pisahkan fitur dan label
X_train = train_df.drop(columns=["Status_Kelulusan"])
y_train = train_df["Status_Kelulusan"]

# Encoding fitur kategorikal
feature_encoders = {}

for col in X_train.select_dtypes(include=["object"]).columns:
    le = LabelEncoder()
    X_train[col] = le.fit_transform(X_train[col])
    feature_encoders[col] = le

# Encoding target
target_encoder = LabelEncoder()
y_train_encoded = target_encoder.fit_transform(y_train)

# Membuat model Random Forest
model = RandomForestClassifier(
    n_estimators=100,
    max_depth=10,
    random_state=42
)

# Training model
model.fit(X_train, y_train_encoded)

# Simpan model dan encoder
pickle_filename = "random_forest_kelulusan_model.pkl"

with open(pickle_filename, "wb") as file:
    pickle.dump((model, feature_encoders, target_encoder), file)

print(f"Model disimpan di: {os.path.abspath(pickle_filename)}")

# Output data training ke KNIME
knio.output_tables[0] = knio.Table.from_pandas(train_df)
```

Parameter yang digunakan:

| Parameter | Nilai | Keterangan |
| --- | --- | --- |
| `n_estimators` | 100 | Jumlah pohon keputusan |
| `max_depth` | 10 | Kedalaman maksimum pohon |
| `random_state` | 42 | Seed agar hasil konsisten |

Pada script ini, kolom `Organisasi` diubah menjadi angka menggunakan `LabelEncoder`, sedangkan target `Status_Kelulusan` juga diubah menjadi bentuk numerik agar dapat diproses oleh model.

---

### Testing: Python Script Prediksi

```{figure} ../img/random-forest-kelulusan/Testing Python Script Prediksi.png
:name: py-testing-rf-kelulusan
:align: center

Konfigurasi Python Script untuk Testing
```

Node **Python Script** kedua digunakan untuk melakukan prediksi pada data testing.

Node ini menerima data testing dari output bawah **Table Partitioner**.

Script yang digunakan:

```python
import knime.scripting.io as knio
import pandas as pd
import pickle

# Ambil data testing dari KNIME
test_df = knio.input_tables[0].to_pandas()

# Load model dan encoder
pickle_filename = "random_forest_kelulusan_model.pkl"

with open(pickle_filename, "rb") as file:
    model, feature_encoders, target_encoder = pickle.load(file)

print("Model berhasil dimuat.")

# Pisahkan fitur dan label
X_test = test_df.drop(columns=["Status_Kelulusan"], errors="ignore")

# Encoding fitur kategorikal menggunakan encoder dari data training
for col, le in feature_encoders.items():
    X_test[col] = le.transform(X_test[col])

# Prediksi
predictions_encoded = model.predict(X_test)
probabilities = model.predict_proba(X_test)

# Ubah hasil prediksi menjadi label asli
predictions_label = target_encoder.inverse_transform(predictions_encoded)

# Tambahkan hasil prediksi ke dataframe
test_df["Prediction_Status_Kelulusan"] = predictions_label
test_df["confidence"] = probabilities.max(axis=1).round(4)

# Output ke KNIME
knio.output_tables[0] = knio.Table.from_pandas(test_df)
```

Kolom hasil yang ditambahkan:

| Kolom | Keterangan |
| --- | --- |
| Prediction_Status_Kelulusan | Hasil prediksi model |
| confidence | Tingkat keyakinan model terhadap prediksi |

---

### Scorer

```{figure} ../img/random-forest-kelulusan/scorer forest.png
:name: scorer-rf-kelulusan
:align: center

Hasil Scorer
```

Node **Scorer** digunakan untuk mengevaluasi performa model.

Node ini membandingkan label asli dengan hasil prediksi.

Konfigurasi Scorer:

| Parameter | Kolom |
| --- | --- |
| Actual Column | Status_Kelulusan |
| Predicted Column | Prediction_Status_Kelulusan |

Output dari Scorer berupa:

- Confusion Matrix
- Accuracy
- Precision
- Recall
- F1-Score

---

#### Confusion Matrix

```{figure} ../img/random-forest-kelulusan/confusion matrik forest.png
:name: confusion-rf-kelulusan
:align: centerr

Confusion Matrix
```

Confusion Matrix digunakan untuk melihat jumlah prediksi benar dan salah.

| Aktual / Prediksi | Tepat Waktu | Tidak Tepat Waktu |
| --- | ---: | ---: |
| Tepat Waktu | Benar | Salah |
| Tidak Tepat Waktu | Salah | Benar |

---

#### Akurasi

```{figure} ../img/random-forest-kelulusan/accuracy forest.png
:name: akurasi-rf-kelulusan
:align: center

Hasil Akurasi Model
```

Akurasi menunjukkan persentase prediksi yang benar dibandingkan dengan seluruh data testing.

Rumus akurasi:

```text
Accuracy = Jumlah Prediksi Benar / Total Data Testing
```

Contoh interpretasi:

Apabila akurasi model sebesar 92%, maka model Random Forest berhasil memprediksi status kelulusan mahasiswa dengan tingkat ketepatan sebesar 92%.

---

## Kesimpulan

Berdasarkan implementasi yang dilakukan, algoritma Random Forest dapat digunakan untuk memprediksi status kelulusan mahasiswa.

Model menggunakan beberapa atribut, yaitu:

- IPK
- Kehadiran
- Nilai_Tugas
- Nilai_UTS
- Nilai_UAS
- Organisasi

Hasil prediksi kemudian dievaluasi menggunakan node Scorer di KNIME. Semakin tinggi nilai akurasi, precision, recall, dan F1-Score, maka semakin baik performa model dalam melakukan klasifikasi status kelulusan mahasiswa.

Random Forest cocok digunakan pada kasus ini karena mampu menangani data numerik dan kategorikal serta memiliki performa yang stabil untuk tugas klasifikasi.
