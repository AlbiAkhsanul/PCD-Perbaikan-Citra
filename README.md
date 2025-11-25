# 🌙 Peningkatan Citra dengan Multi-Scale Retinex dan Guided Filtering  
### *Perbaikan citra cerah, remang, dan gelap menggunakan metode Retinex klasik dengan evaluasi NIQE & BRISQUE tanpa ground truth*

---

## 📌 Gambaran Umum

Repositori ini berisi implementasi dan evaluasi:

- **Multi-Scale Retinex (MSR)**  
- **MSR yang dikembangkan dengan Gaussian multi-skala dan Guided Filtering (MSR-GF)**  
- **Metrik evaluasi no-reference: NIQE & BRISQUE**  

Proyek ini berfokus pada analisis kemampuan metode peningkatan citra berbasis Retinex terhadap **berbagai kondisi pencahayaan**, yaitu:

- Citra cerah  
- Citra remang (pencahayaan sedang)  
- Citra gelap (low-light)  

Pendekatan ini sangat relevan ketika **tidak tersedia citra referensi (ground truth)**, sehingga dibutuhkan metrik kualitas citra *tanpa referensi*.

---

## 📖 Latar Belakang

### 🟦 1. Multi-Scale Retinex (MSR)

Teori Retinex memodelkan citra sebagai kombinasi antara reflektansi dan iluminasi:

\[
I(x,y) = R(x,y) \cdot L(x,y)
\]

dengan:
- \( I(x,y) \) → citra pengamatan  
- \( R(x,y) \) → reflektansi (informasi objek)  
- \( L(x,y) \) → iluminasi (pencahayaan)

Tujuan Retinex adalah memperkirakan \( R(x,y) \) dengan cara mengurangi pengaruh \( L(x,y) \).

Multi-Scale Retinex (MSR) menghitung hasil peningkatan citra dengan:

\[
MSR(x,y) = \sum_{i=1}^{N} w_i \left[ \log(I(x,y)) - \log(I(x,y) * G_{\sigma_i}) \right]
\]

di mana:
- \( G_{\sigma_i} \) adalah Gaussian blur dengan skala (σ) berbeda  
- \( N \) adalah jumlah skala  
- \( w_i \) adalah bobot tiap skala  

**Kelebihan MSR:**
- Meningkatkan kontras lokal dan global  
- Memperjelas detail pada daerah gelap dan terang  

**Kekurangan MSR:**
- Rentan meningkatkan noise  
- Dapat menimbulkan artefak halo di sekitar tepi objek  
- Warna dapat terlihat tidak natural akibat operasi logaritmik

---

### 🟩 2. MSR dengan Guided Filtering (MSR-GF)

**MSR-GF** adalah pengembangan MSR yang menggunakan **estimasi iluminasi yang lebih halus dan edge-aware** melalui kombinasi:

1. **Gaussian filtering multi-skala**  
   - Menghasilkan *rough illumination* pada beberapa tingkat smoothing.

2. **Guided filtering**  
   - Menghasilkan *edge-preserving illumination* yang halus namun tetap mempertahankan struktur tepi.

Alur besarnya:

1. Konversi citra ke grayscale untuk memodelkan iluminasi.  
2. Terapkan Gaussian blur pada beberapa skala (σ kecil, sedang, besar).  
3. Haluskan estimasi iluminasi dengan guided filter menggunakan citra asli sebagai *guide*.  
4. Gabungkan seluruh skala dengan bobot adaptif (misalnya berbasis kekuatan gradien/tepi).  
5. Terapkan Retinex menggunakan iluminasi hasil kombinasi tersebut.  
6. Normalisasi dan lakukan *color balance* sederhana.

**Keuntungan MSR-GF dibanding MSR klasik:**
- Iluminasi lebih halus dan tidak terlalu agresif  
- Noise lebih terkontrol  
- Artefak halo berkurang  
- Detail tetap terjaga, terutama pada citra remang dan gelap  

MSR-GF menjadi kompromi yang baik antara peningkatan pencahayaan dan preservasi struktur lokal citra.

---

## 🎚 3. Metrik Kualitas Citra No-Reference

Karena dataset yang digunakan merupakan **dataset primer tanpa ground truth**, metrik seperti PSNR dan SSIM **tidak dapat digunakan**.  
Sebagai gantinya, digunakan dua metrik *no-reference* yang populer:

---

### ⭐ BRISQUE (Blind/Referenceless Image Spatial Quality Evaluator)

**BRISQUE** mengevaluasi kualitas citra berdasarkan:

- statistik spasial lokal,  
- deteksi distorsi seperti blur, noise, kompresi, dan gangguan tekstur.

BRISQUE menggunakan fitur **Natural Scene Statistics (NSS)** dan memetakan fitur tersebut ke skor kualitas menggunakan model yang telah dilatih sebelumnya.

**Interpretasi BRISQUE:**
- Nilai **lebih rendah** → kualitas citra **lebih baik**  
- Nilai **mendekati 0** → kualitas sangat baik  
- Rentang umum: **0–100** (meski nilai bisa negatif tergantung implementasi)

BRISQUE sensitif terhadap perubahan tekstur dan noise, sehingga sangat cocok untuk mengevaluasi efek samping dari metode peningkatan citra seperti MSR.

---

### ⭐ NIQE (Natural Image Quality Evaluator)

**NIQE** mengukur seberapa jauh sebuah citra menyimpang dari distribusi statistik citra alami berkualitas tinggi.

Karakteristik NIQE:
- Tidak memerlukan pelatihan khusus pada dataset distorsi tertentu (*training-free*).  
- Menggunakan model statistik dari citra alami sebagai referensi.

**Interpretasi NIQE:**
- Nilai **lebih rendah** → citra lebih natural / lebih berkualitas  
- Nilai **lebih tinggi** → citra semakin jauh dari citra alami

Perbedaan utama dengan BRISQUE:
- **NIQE** → menilai kualitas **global** (naturalness, distribusi intensitas, kontras, struktur besar)  
- **BRISQUE** → menilai kualitas **lokal** (noise, tekstur, artefak)

---

## 🧪 Eksperimen

### Kategori Pencahayaan

Eksperimen dilakukan pada tiga kategori citra:

- **Citra cerah**  
- **Citra remang (pencahayaan sedang / low-light ringan)**  
- **Citra gelap (low-light berat)**  

Untuk setiap citra dalam tiap kategori, dilakukan:

- Penerapan **MSR klasik**  
- Penerapan **MSR-GF (metode usulan)**  
- Perhitungan:
  - NIQE citra asli, MSR, dan MSR-GF  
  - BRISQUE citra asli, MSR, dan MSR-GF  
  - Waktu pemrosesan untuk masing-masing metode  

---

## 📈 Ringkasan Temuan Utama

### 🔵 Citra Cerah
- MSR dan MSR-GF umumnya **menurunkan NIQE**, artinya meningkatkan kualitas global dibanding citra asli.
- Pada beberapa citra yang sudah cukup baik, peningkatan justru terbatas dan kadang BRISQUE tidak banyak membaik.
- MSR-GF sering memberikan BRISQUE yang **lebih baik** dibanding MSR, artinya **distorsi lokal lebih rendah**.

### 🟡 Citra Remang
- Keduanya **meningkatkan NIQE** (lebih natural dibanding citra asli).  
- MSR klasik:
  - cenderung memberikan NIQE yang sedikit lebih baik,  
  - tetapi sering meningkatkan BRISQUE cukup tinggi (lebih banyak noise/artefak).  
- MSR-GF:
  - NIQE sedikit lebih tinggi (sedikit kurang baik dari MSR),  
  - namun BRISQUE **lebih rendah** → distorsi lokal lebih terkendali.

### 🔴 Citra Gelap
- Perbaikan paling signifikan terjadi pada kategori ini.  
- MSR dan MSR-GF sama-sama secara drastis **menurunkan NIQE dan BRISQUE**.
- MSR:
  - sering memberikan NIQE terbaik.  
- MSR-GF:
  - memberikan BRISQUE **terendah** → noise dan artefak paling sedikit.  

Secara umum, **MSR-GF dapat dianggap sebagai metode yang paling seimbang** untuk citra gelap: pencahayaan membaik, noise berkurang, dan detail tetap terjaga.

### ⚡ Waktu Pemrosesan

Dalam implementasi ini:

- **MSR-GF** secara konsisten memiliki **waktu pemrosesan lebih cepat** dibanding MSR klasik,  
- sehingga lebih praktis untuk pemrosesan batch pada banyak citra.

---
