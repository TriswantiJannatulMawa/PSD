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

# Pengantar Analisis Polutan Kabupaten Lamongan

## Tentang Proyek

Proyek ini menganalisis **kualitas udara di Kabupaten Lamongan** dengan fokus pada empat polutan udara utama: **NO₂, CO, SO₂, dan CH₄**. Proyek ini disusun sebagai bagian dari mata kuliah **Proyek Sains Data**.

## Latar Belakang

Kabupaten Lamongan memiliki kombinasi aktivitas yang berpotensi memengaruhi kualitas udara: aktivitas logistik migas lepas pantai di **Lamongan Integrated Shorebase (Paciran)**, kawasan industri di jalur **Babat–Jombang**, industri pengolahan hasil laut di wilayah pesisir, sektor pertanian, serta jalur transportasi Pantura yang padat. Kombinasi ini menjadi alasan mengapa kualitas udara di Kabupaten Lamongan perlu dipantau secara berkala menggunakan data penginderaan jauh (satelit).

## Sumber Data

Data kualitas udara diperoleh dari **Copernicus Data Space Ecosystem** (https://dataspace.copernicus.eu/) melalui layanan **openEO**, menggunakan produk **Sentinel-5P L2**, dengan rentang waktu:

- **Mulai:** 24 Agustus 2025
- **Selesai:** 24 Agustus 2026

## Struktur Analisis

Analisis pada buku ini disusun mengikuti alur kerja **CRISP-DM**, terbagi ke dalam tiga bagian utama:

### 1. Business & Data Understanding

Pemahaman awal mengenai indeks kualitas udara (AQI), karakteristik masing-masing polutan (NO₂, CO, SO₂, CH₄) beserta sumber dan dampak kesehatannya, konteks spesifik Kabupaten Lamongan, serta proses pengumpulan dan eksplorasi data satelit — termasuk peta wilayah kajian, pratinjau data, dan identifikasi awal missing values/outlier.

📄 Berkas: `business-understanding.md`, `data-understanding.md`

### 2. Ekstraksi Data

Proses teknis penarikan data satelit Sentinel-5P dari server openEO Copernicus Data Space untuk wilayah Kabupaten Lamongan — mencakup otentikasi, penentuan area kajian, agregasi temporal & spasial per polutan, hingga ekspor hasil ke format NetCDF dan CSV.

📄 Berkas: `1-ekstraksi-data.ipynb`

### 3. Analisis Data (Lamongan)

Pengolahan lanjutan atas data hasil ekstraksi — pemeriksaan dan penanganan missing values, deteksi anomali/outlier menggunakan Isolation Forest, serta visualisasi deret waktu untuk melihat pola konsentrasi tiap polutan di Kabupaten Lamongan.

📄 Berkas: `2-analisis-lamongan.ipynb`

---

## Susunan Daftar Isi (`_toc.yml`)

Contoh struktur `_toc.yml` Jupyter Book v1 yang sesuai dengan urutan di atas:

```yaml
format: jb-book
root: pengantar-analisis-polutan-lamongan
chapters:
  - file: business-understanding
  - file: data-understanding
  - file: 1-ekstraksi-data
  - file: 2-analisis-lamongan
```

> Sesuaikan nama file dengan lokasi sebenarnya di struktur folder proyekmu (mis. jika notebook disimpan dalam subfolder `notebooks/`, tulis `notebooks/1-ekstraksi-data`).
