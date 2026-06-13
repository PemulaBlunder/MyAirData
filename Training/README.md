Kalau template ini dipakai untuk semua polutan (CO, PM2.5, PM10, O₃, SO₂, NO₂, dll.), README-nya lebih baik dibuat generik seperti berikut:

# README - Training Model Time Series Menggunakan LSTM + Genetic Algorithm

## Deskripsi

Project ini digunakan untuk melakukan prediksi parameter kualitas udara menggunakan model **Long Short-Term Memory (LSTM)**.

Pemilihan fitur lag dilakukan berdasarkan analisis **Partial Autocorrelation Function (PACF)**, sedangkan optimasi hyperparameter dilakukan menggunakan **Genetic Algorithm (GA)** menggunakan library DEAP.

Model dapat digunakan untuk berbagai parameter kualitas udara seperti:

* NO₂
* PM2.5
* PM10
* CO
* O₃
* SO₂
* Parameter lainnya yang tersedia pada dataset

---

## Alur Proses

### 1. Load Dataset

Dataset dibaca dari file Excel:

```python
df_final = pd.read_excel('aqicn_Combin.xlsx')
```

Pilih parameter yang akan diprediksi:

```python
data_ts = df_final[target_column]
```

---

### 2. Analisis PACF

PACF digunakan untuk mengidentifikasi lag yang memiliki pengaruh signifikan terhadap nilai saat ini.

```python
lags, threshold, pacf_vals = suggest_time_steps(data_ts, max_lag=40)
```

Output:

* Batas signifikansi PACF
* Daftar lag signifikan
* Rekomendasi jumlah time step

---

### 3. Pembuatan Dataset Supervised

Lag signifikan digunakan sebagai fitur input.

Contoh:

| y  | lag_1 | lag_2 | lag_3 |
| -- | ----- | ----- | ----- |
| xₜ | xₜ₋₁  | xₜ₋₂  | xₜ₋₃  |

Kode:

```python
lag_sig = lags[:3]

for i in lag_sig:
    df_supervised[f'lag_{i}'] = data_ts.shift(i)
```

---

### 4. Split Data

Dataset dibagi menjadi:

* 80% Training
* 20% Testing

```python
train_size = int(len(df_supervised) * 0.8)
```

---

### 5. Normalisasi Data

Menggunakan MinMaxScaler:

```python
scaler = MinMaxScaler()
```

Rentang hasil transformasi:

```text
0 - 1
```

---

### 6. Reshape Data untuk LSTM

Input LSTM memiliki format:

```python
(samples, timesteps, features)
```

Contoh:

```python
X_train.shape = (1000, 3, 1)
```

---

### 7. Arsitektur Model

Model terdiri dari:

* LSTM Layer
* Dropout Layer
* Dense Output Layer

```python
LSTM(units)
Dropout(dropout)
Dense(1)
```

Loss Function:

```python
Mean Squared Error (MSE)
```

Optimizer:

```python
Adam
```

---

### 8. Optimasi Hyperparameter dengan Genetic Algorithm

Parameter yang dioptimasi:

| Parameter     | Range         |
| ------------- | ------------- |
| Units         | 50 – 128      |
| Dropout       | 0.2 – 0.5     |
| Learning Rate | 0.001 – 0.005 |
| Epochs        | 30 – 60       |

Konfigurasi Genetic Algorithm:

```python
Population = 15
Generation = 10
Crossover Probability = 0.5
Mutation Probability = 0.2
```

Objective:

```python
Meminimalkan nilai MSE pada data testing
```

---

### 9. Training Model Final

Hyperparameter terbaik hasil GA digunakan untuk melatih model final.

```python
final_model.fit(...)
```

---

### 10. Evaluasi Model

Metrik evaluasi yang digunakan:

#### MAE

```text
Mean Absolute Error
```

#### RMSE

```text
Root Mean Squared Error
```

#### R² Score

```text
Coefficient of Determination
```

---

### 11. Visualisasi Hasil Prediksi

Menampilkan perbandingan:

* Nilai aktual
* Nilai prediksi

```python
plt.plot(y_test)
plt.plot(y_pred)
```

---

### 12. Penyimpanan Model

#### Hyperparameter Terbaik

```python
best_params.json
```

Contoh:

```json
{
    "units": 96,
    "dropout": 0.31,
    "learning_rate": 0.0025,
    "epochs": 45,
    "batch_size": 32
}
```

#### Model Hasil Training

```python
final_model.keras
```

---

## Struktur Output

| File              | Deskripsi                             |
| ----------------- | ------------------------------------- |
| aqicn_Combin.xlsx | Dataset sumber                        |
| best_params.json  | Hyperparameter terbaik hasil optimasi |
| final_model.keras | Model hasil training                  |
| Grafik Evaluasi   | Visualisasi aktual vs prediksi        |

---

## Dependency

```bash
pip install pandas numpy matplotlib scikit-learn statsmodels tensorflow deap
```

---

## Catatan

* Model menggunakan pendekatan **univariate time series forecasting**.
* Fitur input berasal dari nilai historis parameter yang sama (lag features).
* Lag dipilih secara otomatis berdasarkan hasil PACF.
* Hyperparameter model dioptimasi menggunakan Genetic Algorithm.
* Template ini dapat digunakan untuk berbagai parameter kualitas udara dengan mengganti kolom target yang digunakan pada proses training.
