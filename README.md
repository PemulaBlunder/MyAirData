# My Air Data - Air Quality Forecasting Pipeline

## Overview

My Air Data merupakan aplikasi berbasis web yang digunakan untuk memantau kualitas udara di wilayah Tandes, Surabaya. Sistem ini menyediakan informasi konsentrasi polutan udara serta prediksi kualitas udara beberapa hari ke depan menggunakan model Deep Learning berbasis Long Short-Term Memory (LSTM).

Polutan yang dipantau meliputi:

* PM2.5
* PM10
* CO
* NO₂
* O₃
* SO₂

Selain menampilkan nilai konsentrasi polutan, sistem juga mengonversi hasil prediksi menjadi Air Quality Index (AQI) sehingga lebih mudah dipahami oleh masyarakat.

---

# Pipeline Overview

## 1. Data Collection

Data historis kualitas udara diperoleh melalui API AQICN untuk setiap polutan yang tersedia.

Data yang diambil meliputi:

* PM2.5
* PM10
* CO
* NO₂
* O₃
* SO₂

Nilai yang digunakan adalah median harian dari masing-masing polutan.

---

## 2. Data Preprocessing

Tahapan preprocessing meliputi:

* Menggabungkan seluruh data polutan berdasarkan tanggal.
* Melakukan interpolasi linear untuk mengisi data yang hilang.
* Melakukan forward fill dan backward fill untuk sisa nilai kosong.
* Menghapus data hari berjalan agar hanya menggunakan data yang sudah lengkap.
* Menyusun ulang kolom sesuai format dataset yang digunakan model.

Output tahap ini berupa dataset kualitas udara yang telah bersih dan siap digunakan untuk retraining maupun forecasting.

---

## 3. Model Retraining

Setiap polutan memiliki model LSTM tersendiri yang telah dilatih sebelumnya.

Pada tahap retraining:

1. Model terbaik yang tersimpan dimuat kembali.
2. Parameter terbaik hasil optimasi dimuat dari file JSON.
3. Scaler yang digunakan saat training dimuat kembali.
4. Dataset terbaru digunakan untuk memperbarui model.

Tujuan retraining adalah menjaga performa model agar tetap mengikuti pola kualitas udara terkini.

---

## 4. Forecasting

Forecasting dilakukan menggunakan pendekatan Sequential Forecasting.

Karakteristik metode:

* Prediksi dilakukan secara bertahap untuk beberapa hari ke depan.
* Hasil prediksi hari sebelumnya digunakan sebagai input pada prediksi berikutnya.
* Model diperbarui kembali setelah setiap iterasi prediksi.

Pendekatan ini memungkinkan model menghasilkan prediksi multi-step yang lebih adaptif terhadap perubahan pola data.

---

## 5. AQI Calculation

Hasil prediksi konsentrasi polutan dikonversi menjadi Air Quality Index (AQI) menggunakan standar US EPA.

Kategori AQI yang digunakan:

| AQI       | Kategori           |
| --------- | ------------------ |
| 0 - 50    | Baik               |
| 51 - 100  | Sedang             |
| 101 - 200 | Tidak Sehat        |
| 201 - 300 | Sangat Tidak Sehat |
| > 300     | Berbahaya          |

AQI digunakan untuk memberikan interpretasi yang lebih mudah dipahami oleh pengguna aplikasi.

---

## Output

Pipeline menghasilkan:

### Dataset Bersih

```json
df_final.json
```

Berisi data historis kualitas udara yang telah melalui proses preprocessing.

### Hasil Prediksi

```json
forecasts.json
```

Berisi:

```json
{
  "date": "2025-10-08",
  "value": 35.42,
  "aqi": 98,
  "category": "Sedang"
}
```

---

## Model Architecture

Model yang digunakan:

* LSTM Layer
* Dropout Layer
* Dense Output Layer

Loss Function:

```text
Mean Squared Error (MSE)
```

Optimizer:

```text
Adam
```

---

## Features

* Monitoring kualitas udara harian.
* Prediksi kualitas udara hingga beberapa hari ke depan.
* Retraining otomatis menggunakan data terbaru.
* Perhitungan AQI berdasarkan standar US EPA.
* Dukungan untuk enam polutan utama.
* Output dalam format JSON yang siap digunakan oleh aplikasi web.

---

## Use Case

My Air Data dapat digunakan untuk:

* Pemantauan kualitas udara harian.
* Edukasi masyarakat mengenai kondisi lingkungan.
* Pendukung pengambilan keputusan terkait kesehatan dan lingkungan.
* Sistem peringatan dini terhadap potensi peningkatan tingkat polusi udara.
