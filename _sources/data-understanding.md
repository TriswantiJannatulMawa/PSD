---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Data Understanding

Data Understanding adalah tahap untuk **mengumpulkan**, **mengeksplorasi**, dan **menilai kualitas** data yang akan digunakan dalam analisis kualitas udara di Kabupaten Lamongan.

---

## Library Python yang Diperlukan

| Library | Kegunaan |
| ------- | -------- |
| `openeo` | Menghubungkan dan memproses data satelit dari server openEO (Copernicus Data Space). |
| `xarray` / `netCDF4` | Membaca file hasil batch job openEO berformat NetCDF (`.nc`). |
| `pandas` | Membaca dan mengolah data tabular (CSV), serta manipulasi deret waktu. |
| `numpy` | Komputasi numerik, misalnya untuk perhitungan rata-rata dan statistik. |
| `folium` | Membuat visualisasi peta interaktif lokasi pengamatan. |
| `matplotlib` / `seaborn` | Visualisasi grafik deret waktu dan deteksi outlier. |
| `scikit-learn` | Algoritma `IsolationForest` untuk deteksi anomali/outlier. |

Instalasi (lihat juga `requirements.txt` pada proyek ini):

```bash
pip install -r requirements.txt
```

---

## 1. Collecting (Mengumpulkan Data)

### 1.1 Sumber Data

| Item | Keterangan |
| ---- | ---------- |
| **Sumber** | Copernicus Data Space Ecosystem |
| **Layanan** | openEO |
| **Server** | `openeo.dataspace.copernicus.eu` |
| **Produk / Koleksi** | Sentinel-5P L2 |
| **Polutan** | NO₂, CO, SO₂, CH₄ |
| **Periode** | 24 Agustus 2025 – 24 Agustus 2026 |
| **Lokasi** | Kabupaten Lamongan, Jawa Timur |

### 1.2 Koneksi dan Otentikasi

Langkah pertama adalah menghubungkan ke server openEO dan melakukan otentikasi menggunakan akun **Copernicus Data Space** dengan metode **device code flow**.

```python
import openeo

connection = openeo.connect("openeo.dataspace.copernicus.eu")
connection.authenticate_oidc()
```

Saat dijalankan, akan muncul tautan untuk login di browser. Setelah berhasil login, akan muncul konfirmasi seperti:

```
Visit (link autentikasi) 📋 to authenticate.
✅ Authorized successfully
Authenticated using device code flow.
```

> **Catatan:** Proses ekstraksi lengkap (loop 4 polutan, agregasi, ekspor NetCDF & CSV) ada pada notebook `1-ekstraksi-data.ipynb` di proyek ini.

### 1.3 Penentuan Area of Interest (AOI)

Lokasi pengamatan dibatasi pada wilayah **Kabupaten Lamongan**. Setiap titik pada polygon memiliki format `[longitude (bujur), latitude (lintang)]`.

| Atribut | Nilai | Keterangan |
| ------- | ----- | ---------- |
| `west` | 112.00 | Longitude terkecil (batas kiri) |
| `east` | 112.60 | Longitude terbesar (batas kanan) |
| `south` | -7.35 | Latitude terkecil (batas bawah) |
| `north` | -6.85 | Latitude terbesar (batas atas) |

> **Penting:** Bounding box di atas adalah **perkiraan sederhana** yang mengikuti bentuk umum Kabupaten Lamongan. Untuk hasil yang presisi secara administratif, sebaiknya diganti dengan geometri resmi (mis. dari [geojson.io](https://geojson.io), **GADM**, atau **Batas Administrasi BIG**) dibaca via `geopandas`.

`spatial_extent` pada `load_collection` menggunakan *bounding box* (`west`, `south`, `east`, `north`), sedangkan geometri polygon digunakan pada tahap `aggregate_spatial` untuk menghitung rata-rata di dalam area tersebut.

### 1.4 Memuat Data (Load Collection)

Data dimuat **per polutan**, karena band Sentinel-5P diproses satu per satu. Berikut contoh untuk NO₂ — CO, SO₂, dan CH₄ menggunakan struktur kode yang identik, hanya berbeda pada parameter `bands`.

```python
LAMONGAN_BBOX = {
    "west": 112.00,
    "south": -7.35,
    "east": 112.60,
    "north": -6.85,
}

s5 = connection.load_collection(
    "SENTINEL_5P_L2",
    temporal_extent=["2025-08-24", "2026-08-24"],
    spatial_extent=LAMONGAN_BBOX,
    bands=["NO2"],
)
```

### 1.5 Definisi AOI (Polygon)

```python
LAMONGAN_POLYGON = {
    "type": "Polygon",
    "coordinates": [[
        [112.00, -7.00],
        [112.10, -7.30],
        [112.25, -7.35],
        [112.40, -7.25],
        [112.55, -7.15],
        [112.60, -7.00],
        [112.50, -6.88],
        [112.30, -6.85],
        [112.10, -6.90],
        [112.00, -7.00],
    ]]
}
```

### 1.6 Agregasi Data

Setiap datacube diagregasi menjadi **rata-rata harian**, lalu dihitung **rata-rata spasial** di dalam polygon AOI untuk menghasilkan deret waktu (time series).

```python
s5 = s5.aggregate_temporal_period(reducer="mean", period="day")
s5 = s5.aggregate_spatial(reducer="mean", geometries=LAMONGAN_POLYGON)
```

### 1.7 Menjalankan Batch Job & Ekspor

Hasil diunduh sebagai NetCDF, lalu dikonversi menjadi CSV:

| Polutan | File NetCDF | File CSV |
| ------- | ----------- | -------- |
| NO₂ | `data/nc/no2_lamongan.nc` | `data/csv/no2_lamongan.csv` |
| CO | `data/nc/co_lamongan.nc` | `data/csv/co_lamongan.csv` |
| SO₂ | `data/nc/so2_lamongan.nc` | `data/csv/so2_lamongan.csv` |
| CH₄ | `data/nc/ch4_lamongan.nc` | `data/csv/ch4_lamongan.csv` |

> Detail lengkap proses ini (fungsi `extract_pollutant()`, loop 4 polutan, konversi NetCDF→CSV) ada di notebook `1-ekstraksi-data.ipynb`.

---

## 2. Visualisasi Peta Wilayah Kajian (Folium)

Peta interaktif berikut menampilkan **batas wilayah kajian (AOI)** Kabupaten Lamongan beserta **titik-titik potensi sumber emisi** yang telah dibahas pada bagian Business Understanding — Lamongan Integrated Shorebase (Paciran), kawasan industri kayu di jalur Babat–Jombang, dan pusat kota Lamongan sebagai titik acuan.

```{code-cell}
:tags: [hide-input]
import folium

# Titik pusat AOI (rata-rata bounding box)
LAMONGAN_BBOX = {"west": 112.00, "south": -7.35, "east": 112.60, "north": -6.85}
lat_c = (LAMONGAN_BBOX["south"] + LAMONGAN_BBOX["north"]) / 2
lon_c = (LAMONGAN_BBOX["west"] + LAMONGAN_BBOX["east"]) / 2

# Peta dasar dengan pilihan layer (jalan & citra satelit)
m = folium.Map(location=[lat_c, lon_c], zoom_start=10, tiles="OpenStreetMap")
folium.TileLayer(
    tiles="https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png",
    attr="OpenTopoMap",
    name="Topografi",
).add_to(m)

# Polygon AOI Kabupaten Lamongan (perkiraan)
lamongan_polygon_latlon = [
    [-7.00, 112.00], [-7.30, 112.10], [-7.35, 112.25], [-7.25, 112.40],
    [-7.15, 112.55], [-7.00, 112.60], [-6.88, 112.50], [-6.85, 112.30],
    [-6.90, 112.10], [-7.00, 112.00],
]
folium.Polygon(
    locations=lamongan_polygon_latlon,
    color="blue",
    weight=2,
    fill=True,
    fill_color="blue",
    fill_opacity=0.12,
    tooltip="Area of Interest (AOI) — Kabupaten Lamongan",
).add_to(m)

# Kotak bounding box (persegi pembatas yang dipakai load_collection)
folium.Rectangle(
    bounds=[[LAMONGAN_BBOX["south"], LAMONGAN_BBOX["west"]],
            [LAMONGAN_BBOX["north"], LAMONGAN_BBOX["east"]]],
    color="gray",
    weight=1,
    dash_array="5,5",
    fill=False,
    tooltip="Bounding Box (spatial_extent load_collection)",
).add_to(m)

# Titik pusat kota Lamongan
folium.Marker(
    location=[-7.1201, 112.4147],
    popup="Pusat Kota Lamongan",
    tooltip="Pusat Kota Lamongan",
    icon=folium.Icon(color="blue", icon="home"),
).add_to(m)

# Titik potensi sumber emisi: Lamongan Integrated Shorebase (Paciran)
folium.Marker(
    location=[-6.8767, 112.3735],
    popup="Lamongan Integrated Shorebase (LIS) — Kec. Paciran<br>Aktivitas logistik & shorebase migas lepas pantai",
    tooltip="Lamongan Integrated Shorebase (Paciran)",
    icon=folium.Icon(color="red", icon="ship", prefix="fa"),
).add_to(m)

# Titik potensi sumber emisi: kawasan industri kayu Babat-Jombang
folium.Marker(
    location=[-7.1000, 112.1833],
    popup="Kawasan Industri Kayu — Jalur Babat–Jombang<br>Pabrik kayu lapis, dilaporkan mengeluarkan asap cerobong",
    tooltip="Kawasan Industri Babat",
    icon=folium.Icon(color="orange", icon="industry", prefix="fa"),
).add_to(m)

folium.LayerControl().add_to(m)
m
```

> Koordinat titik marker (Shorebase Paciran, kawasan industri Babat) merupakan **perkiraan lokasi kecamatan**, bukan koordinat presisi fasilitas. Gunakan untuk konteks visual, bukan rujukan teknis presisi tinggi.

---

## 3. Menampilkan Hasil Data CSV

Setelah data dikonversi menjadi CSV, kita tampilkan 5 baris teratas dari masing-masing polutan.

### 3.1 NO₂

```{code-cell}
:tags: [hide-input]
import pandas as pd

df_no2 = pd.read_csv("../data/csv/no2_lamongan.csv")
df_no2.head()
```

### 3.2 CO

```{code-cell}
:tags: [hide-input]
df_co = pd.read_csv("../data/csv/co_lamongan.csv")
df_co.head()
```

### 3.3 SO₂

```{code-cell}
:tags: [hide-input]
df_so2 = pd.read_csv("../data/csv/so2_lamongan.csv")
df_so2.head()
```

### 3.4 CH₄

```{code-cell}
:tags: [hide-input]
df_ch4 = pd.read_csv("../data/csv/ch4_lamongan.csv")
df_ch4.head()
```

### 3.5 Kenapa Ada Nilai NaN?

Beberapa baris pada tabel di atas menunjukkan nilai **NaN** (Not a Number) — artinya pada tanggal tersebut tidak ada nilai polutan yang tercatat. Penyebabnya antara lain:

1. **Tidak ada lintasan satelit** — Sentinel-5P memiliki *revisit time* (jadwal orbit) tertentu, sehingga tidak setiap hari melewati Kabupaten Lamongan.
2. **Tutupan awan (cloud cover)** — sensor Sentinel-5P dipengaruhi kondisi atmosfer; area yang tertutup awan tebal menghasilkan data tidak valid.
3. **Validasi kualitas (quality flag)** — data dengan kualitas buruk (noise instrumen) dibuang oleh proses validasi, menyisakan celah (*gap*) pada deret waktu.

Dari total hari dalam rentang waktu ekstraksi, tidak semua tanggal memiliki nilai polutan. Hari-hari kosong ini dibahas lebih lanjut pada bagian **identifikasi missing values** berikut.

---

## 4. Identifikasi Kualitas Data

Sesuai prinsip CRISP-DM, tahap Data Understanding hanya **menemukan dan mencatat** masalah kualitas data (missing values, outliers) — penanganan dilakukan pada tahap Data Preparation berikutnya.

### 4.1 Missing Values

**NO₂**

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../data/csv/no2_lamongan.csv")
no2 = df.iloc[:, -1]
print(f"Jumlah missing value pada data NO2 : {no2.isna().sum()}")
print(f"Jumlah data terisi (valid) pada data NO2 : {no2.notna().sum()}")
```

**CO**

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../data/csv/co_lamongan.csv")
co = df.iloc[:, -1]
print(f"Jumlah missing value pada data CO : {co.isna().sum()}")
print(f"Jumlah data terisi (valid) pada data CO : {co.notna().sum()}")
```

**SO₂**

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../data/csv/so2_lamongan.csv")
so2 = df.iloc[:, -1]
print(f"Jumlah missing value pada data SO2 : {so2.isna().sum()}")
print(f"Jumlah data terisi (valid) pada data SO2 : {so2.notna().sum()}")
```

**CH₄**

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../data/csv/ch4_lamongan.csv")
ch4 = df.iloc[:, -1]
print(f"Jumlah missing value pada data CH4 : {ch4.isna().sum()}")
print(f"Jumlah data terisi (valid) pada data CH4 : {ch4.notna().sum()}")
```

### 4.2 Outliers

Deteksi outlier dilakukan menggunakan algoritma **Isolation Forest** dengan `contamination=0.05` (5%). Sebelum deteksi, baris dengan nilai kosong (`NaN`) dibuang terlebih dahulu (`dropna()`) agar populasi perhitungan pencilan selaras.

**NO₂**

```{code-cell}
:tags: [hide-input]
import matplotlib.pyplot as plt
from sklearn.ensemble import IsolationForest

df = pd.read_csv("../data/csv/no2_lamongan.csv")
value_col = df.columns[-1]
df["date"] = pd.to_datetime(df.iloc[:, 0])
df_clean = df.dropna(subset=[value_col]).copy()

model = IsolationForest(contamination=0.05, random_state=42)
df_clean["outlier"] = model.fit_predict(df_clean[[value_col]])

normal = df_clean[df_clean["outlier"] == 1]
outliers = df_clean[df_clean["outlier"] == -1]

print(f"Jumlah outlier pada data NO2 : {len(outliers)}")
print(f"Jumlah tidak outlier (normal) pada data NO2 : {len(normal)}")

plt.figure(figsize=(10, 4))
plt.scatter(normal["date"], normal[value_col], color="blue", label="Normal", s=30)
plt.scatter(outliers["date"], outliers[value_col], color="red", label="Outlier", s=50)
plt.title("Deteksi Outlier NO2 — Kabupaten Lamongan (Merah = Outlier, Biru = Normal)")
plt.xlabel("Tanggal")
plt.ylabel("Konsentrasi NO2")
plt.legend()
plt.grid(True)
plt.show()
```

**CO**

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../data/csv/co_lamongan.csv")
value_col = df.columns[-1]
df["date"] = pd.to_datetime(df.iloc[:, 0])
df_clean = df.dropna(subset=[value_col]).copy()

model = IsolationForest(contamination=0.05, random_state=42)
df_clean["outlier"] = model.fit_predict(df_clean[[value_col]])

normal = df_clean[df_clean["outlier"] == 1]
outliers = df_clean[df_clean["outlier"] == -1]

print(f"Jumlah outlier pada data CO : {len(outliers)}")
print(f"Jumlah tidak outlier (normal) pada data CO : {len(normal)}")

plt.figure(figsize=(10, 4))
plt.scatter(normal["date"], normal[value_col], color="blue", label="Normal", s=30)
plt.scatter(outliers["date"], outliers[value_col], color="red", label="Outlier", s=50)
plt.title("Deteksi Outlier CO — Kabupaten Lamongan (Merah = Outlier, Biru = Normal)")
plt.xlabel("Tanggal")
plt.ylabel("Konsentrasi CO")
plt.legend()
plt.grid(True)
plt.show()
```

**SO₂**

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../data/csv/so2_lamongan.csv")
value_col = df.columns[-1]
df["date"] = pd.to_datetime(df.iloc[:, 0])
df_clean = df.dropna(subset=[value_col]).copy()

model = IsolationForest(contamination=0.05, random_state=42)
df_clean["outlier"] = model.fit_predict(df_clean[[value_col]])

normal = df_clean[df_clean["outlier"] == 1]
outliers = df_clean[df_clean["outlier"] == -1]

print(f"Jumlah outlier pada data SO2 : {len(outliers)}")
print(f"Jumlah tidak outlier (normal) pada data SO2 : {len(normal)}")

plt.figure(figsize=(10, 4))
plt.scatter(normal["date"], normal[value_col], color="blue", label="Normal", s=30)
plt.scatter(outliers["date"], outliers[value_col], color="red", label="Outlier", s=50)
plt.title("Deteksi Outlier SO2 — Kabupaten Lamongan (Merah = Outlier, Biru = Normal)")
plt.xlabel("Tanggal")
plt.ylabel("Konsentrasi SO2")
plt.legend()
plt.grid(True)
plt.show()
```

**CH₄**

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../data/csv/ch4_lamongan.csv")
value_col = df.columns[-1]
df["date"] = pd.to_datetime(df.iloc[:, 0])
df_clean = df.dropna(subset=[value_col]).copy()

model = IsolationForest(contamination=0.05, random_state=42)
df_clean["outlier"] = model.fit_predict(df_clean[[value_col]])

normal = df_clean[df_clean["outlier"] == 1]
outliers = df_clean[df_clean["outlier"] == -1]

print(f"Jumlah outlier pada data CH4 : {len(outliers)}")
print(f"Jumlah tidak outlier (normal) pada data CH4 : {len(normal)}")

plt.figure(figsize=(10, 4))
plt.scatter(normal["date"], normal[value_col], color="blue", label="Normal", s=30)
plt.scatter(outliers["date"], outliers[value_col], color="red", label="Outlier", s=50)
plt.title("Deteksi Outlier CH4 — Kabupaten Lamongan (Merah = Outlier, Biru = Normal)")
plt.xlabel("Tanggal")
plt.ylabel("Konsentrasi CH4")
plt.legend()
plt.grid(True)
plt.show()
```

### 4.3 Noise

*(Akan dibahas pada iterasi berikutnya — belum dianalisis pada versi ini.)*

---

## 5. Ringkasan Data Understanding

- Data bersumber dari **Sentinel-5P L2** via openEO Copernicus Data Space, mencakup 4 polutan (NO₂, CO, SO₂, CH₄) di Kabupaten Lamongan sepanjang 24 Agustus 2025 – 24 Agustus 2026.
- Data memiliki **missing values** akibat *revisit time* satelit, tutupan awan, dan validasi kualitas — proporsi tiap polutan diringkas pada Bagian 4.1.
- Deteksi awal **outlier** dengan Isolation Forest (`contamination=0.05`) menunjukkan sebagian kecil (~5%) titik data yang menyimpang signifikan dari pola umum, perlu ditelaah lebih lanjut apakah mencerminkan kejadian nyata atau gangguan data.
- Temuan-temuan ini menjadi dasar untuk tahap **Data Preparation** (imputasi missing values, penanganan outlier) sebelum data digunakan pada tahap analisis/pemodelan lanjutan.
