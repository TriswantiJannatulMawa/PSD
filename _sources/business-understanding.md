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

# Business Understanding

Business Understanding adalah tahap awal dalam kerangka kerja **CRISP-DM** yang bertujuan memahami **latar belakang**, **tujuan**, dan **konteks masalah** sebelum masuk ke tahap pengolahan data. Pada proyek ini, kita membahas kualitas udara di **Kabupaten Lamongan, Jawa Timur**, dengan fokus pada empat polutan utama: **NO₂, CO, SO₂, dan CH₄**.

---

## 1. Latar Belakang Masalah

Kualitas udara adalah salah satu indikator lingkungan yang berdampak langsung pada kesehatan masyarakat. Meskipun Kabupaten Lamongan tidak sebesar Gresik atau Surabaya dari sisi kepadatan industri berat, wilayah ini tetap memiliki beberapa **sumber emisi udara yang nyata** dan terus berkembang, sehingga pemantauan kualitas udara berbasis data tetap relevan untuk dilakukan.

### Konteks Kabupaten Lamongan

Berikut adalah beberapa aktivitas di Kabupaten Lamongan yang berpotensi menjadi sumber emisi udara:

- **Lamongan Integrated Shorebase (LIS) — Kecamatan Paciran**
  Fasilitas logistik dan *shorebase* migas lepas pantai (*offshore oil & gas*) yang melayani eksplorasi dan produksi minyak-gas di perairan Jawa Timur. Aktivitas bongkar-muat, lalu lintas kapal, dan operasional alat berat di kawasan pelabuhan ini berpotensi menyumbang emisi NO₂ dan CO dari pembakaran bahan bakar kapal dan kendaraan operasional.
- **Kawasan industri di jalur Babat–Jombang**
  Terdapat pabrik pengolahan kayu (kayu lapis) di sekitar Desa Kalen dan Dradahblumbang yang pernah menjadi sorotan warga karena cerobong asapnya mengeluarkan asap hitam pekat, memicu keluhan gangguan pernapasan dan iritasi mata di sekitar permukiman.
- **Industri pengolahan hasil laut**
  Dengan garis pantai sepanjang ±47 km yang mencakup 17 desa pesisir (Paciran, Brondong, dan sekitarnya), Lamongan memiliki sejumlah industri pengolahan ikan (surimi, tepung ikan) yang sebagian masih menggunakan bahan bakar solar dan batu bara untuk boiler dan genset — sumber emisi SO₂ dan CO yang khas dari pembakaran bahan bakar fosil konvensional.
- **Sektor pertanian**
  Lamongan dikenal sebagai salah satu lumbung padi Jawa Timur. Aktivitas pertanian (sawah, pengelolaan limbah organik) merupakan salah satu sumber emisi CH₄ (metana) yang signifikan secara global maupun lokal.
- **Jalur transportasi Pantura (Jalan Daendels) dan koridor Surabaya–Babat–Jombang**
  Sebagai jalur penghubung utama pantai utara Jawa dan koridor ke Surabaya, kepadatan lalu lintas kendaraan bermotor turut berkontribusi terhadap emisi NO₂ dan CO.

> Kombinasi aktivitas pelabuhan/logistik migas, industri manufaktur skala menengah, pengolahan hasil laut, pertanian, dan transportasi inilah yang menjadi alasan mengapa kualitas udara di Kabupaten Lamongan perlu dipantau secara berkala dan berbasis data.

---

## 2. Tujuan Analisis

Proyek ini disusun sebagai bagian dari mata kuliah **Proyek Sains Data**, dengan tujuan sebagai berikut:

1. **Mengukur dan memantau** konsentrasi empat polutan udara utama (NO₂, CO, SO₂, CH₄) di wilayah Kabupaten Lamongan menggunakan data satelit Sentinel-5P, tanpa memerlukan stasiun pemantauan darat.
2. **Mengidentifikasi pola dan tren** konsentrasi polutan dari waktu ke waktu (24 Agustus 2025 – 24 Agustus 2026) untuk melihat apakah ada musim, periode, atau kejadian tertentu dengan tingkat polusi yang menonjol.
3. **Mendeteksi anomali/outlier** pada data deret waktu polutan, yang dapat mengindikasikan kejadian pencemaran tidak biasa (mis. kebakaran, lonjakan aktivitas industri, atau justru artefak/gangguan pada data satelit itu sendiri).
4. **Menyediakan visualisasi yang mudah dipahami** (peta interaktif dan grafik deret waktu) sebagai dasar edukasi publik maupun bahan pertimbangan awal untuk pemangku kepentingan terkait kualitas udara di Lamongan.
5. **Menerapkan alur kerja Data Science yang terstruktur** (CRISP-DM: Business Understanding → Data Understanding → Data Preparation → dst.) sebagai bagian dari pembelajaran mata kuliah.

> **Catatan:** Proyek ini bersifat akademis/edukatif dan menggunakan data penginderaan jauh (satelit), bukan pengukuran langsung di permukaan tanah (*ground station*). Hasil analisis sebaiknya tidak dijadikan satu-satunya dasar pengambilan keputusan resmi terkait kebijakan lingkungan.

---

## 3. Mengamati Kualitas Udara

### Apa itu Indeks Kualitas Udara (AQI)?

**Indeks Kualitas Udara (Air Quality Index / AQI)** adalah standar pengukuran yang digunakan untuk melaporkan seberapa bersih atau tercemarnya udara di suatu wilayah. AQI mengubah konsentrasi polutan udara utama menjadi angka pada skala 0–500, di mana angka yang lebih rendah menunjukkan kualitas udara yang lebih baik dan angka yang lebih tinggi menunjukkan tingkat polusi yang lebih berbahaya.

| Rentang AQI | Kategori | Dampak Kesehatan |
|:-----------:|:--------:|:-----------------|
| 0 – 50 | Baik | Kualitas udara memuaskan, risiko polusi rendah |
| 51 – 100 | Sedang | Kualitas udara dapat diterima |
| 101 – 150 | Tidak Sehat bagi Kelompok Sensitif | Kelompok sensitif (anak, lansia, penderita asma) mulai terdampak |
| 151 – 200 | Tidak Sehat | Seluruh populasi mulai mengalami efek kesehatan |
| 201 – 300 | Sangat Tidak Sehat | Peringatan kesehatan untuk seluruh populasi |
| 301 – 500 | Berbahaya | Kondisi darurat kesehatan masyarakat |

> Catatan: Sentinel-5P mengukur **kolom konsentrasi gas di atmosfer** (mol/m² atau serupa), bukan langsung nilai AQI 0–500. Tabel AQI di atas disertakan sebagai kerangka konseptual untuk memahami tingkat bahaya polutan secara umum, bukan hasil konversi langsung dari data satelit pada proyek ini.

---

## 4. Unsur Polutan yang Dianalisis

### 4.1 NO₂ (Nitrogen Dioksida)

Gas berwarna coklat kemerahan dengan bau menyengat. Sumber utama:
- **Emisi kendaraan bermotor** — pembakaran bahan bakar fosil pada mesin kendaraan, relevan terutama di jalur Pantura dan koridor Surabaya–Babat–Jombang.
- **Aktivitas pelabuhan/logistik** — lalu lintas kapal dan alat berat di kawasan Lamongan Integrated Shorebase (Paciran).
- **Aktivitas industri** — pabrik yang menggunakan proses pembakaran, termasuk pabrik pengolahan kayu di jalur Babat–Jombang.

**Dampak kesehatan:** iritasi saluran pernapasan, peningkatan risiko infeksi pernapasan, dan memperburuk kondisi asma.

### 4.2 CO (Karbon Monoksida)

Gas tidak berwarna dan tidak berbau yang sangat berbahaya. Sumber utama:
- **Pembakaran tidak sempurna** bahan bakar kendaraan bermotor dan kapal.
- **Asap pabrik** — termasuk cerobong industri kayu lapis yang pernah dikeluhkan warga sekitar Babat–Jombang.
- **Boiler dan genset industri** — banyak digunakan pada industri pengolahan hasil laut di kawasan pesisir Lamongan.

**Dampak kesehatan:** mengikat hemoglobin dalam darah lebih kuat dari oksigen, menyebabkan kekurangan oksigen pada organ tubuh, pusing, mual, dan pada paparan tinggi dapat menyebabkan kematian.

### 4.3 SO₂ (Sulfur Dioksida)

Gas tak berwarna dengan bau tajam dan mengganggu. Sumber utama:
- **Pembakaran batu bara** — dipakai sebagian industri pengolahan ikan (boiler) di kawasan pesisir Lamongan.
- **Proses industri** — peleburan logam dan proses manufaktur skala menengah.
- **Emisi kapal/kendaraan** yang menggunakan bahan bakar mengandung sulfur, relevan di kawasan pelabuhan Paciran.

**Dampak kesehatan:** iritasi mata dan tenggorokan, memperburuk penyakit asma dan bronkitis, serta berkontribusi terhadap terbentuknya **asam sulfat** yang menyebabkan hujan asam.

### 4.4 CH₄ (Metana)

Gas rumah kaca yang sangat efektif dalam memerangkap panas. Sumber utama:
- **Aktivitas pertanian** — sawah dan pencernaan hewan ternak, sangat relevan mengingat Lamongan adalah salah satu daerah lumbung padi Jawa Timur.
- **Limbah organik** — pembusukan sampah di Tempat Pembuangan Akhir (TPA) dan limbah hasil pengolahan ikan.
- **Emisi industri** — kebocoran gas alam dan proses industri, termasuk potensi dari aktivitas migas di sekitar kawasan shorebase.

**Dampak lingkungan:** CH₄ merupakan gas rumah kaca yang **80 kali lebih kuat** dari CO₂ dalam jangka pendek, berkontribusi signifikan terhadap perubahan iklim dan pemanasan global.

---

## 5. Ringkasan

| Polutan | Sumber Dominan di Lamongan | Dampak Utama |
|:-------:|:---------------------------|:--------------|
| NO₂ | Transportasi Pantura, aktivitas pelabuhan Paciran | Gangguan pernapasan |
| CO | Asap industri kayu, boiler industri pengolahan ikan | Keracunan oksigen darah |
| SO₂ | Boiler batu bara industri perikanan, emisi kapal | Iritasi & hujan asam |
| CH₄ | Pertanian (sawah), limbah organik, aktivitas migas | Pemanasan global |

Pemahaman di atas menjadi dasar untuk tahap berikutnya, yaitu **Data Understanding**, di mana kita akan mengumpulkan dan mengeksplorasi data satelit Sentinel-5P untuk keempat polutan tersebut di wilayah Kabupaten Lamongan.
