# Pemanfaatan Regular Expression untuk Seleksi dan Ekspor Band Sentinel-2 Otomatis pada Google Earth Engine

**Tutorial Teknis Berbasis Literatur**

![Google Earth Engine](https://img.shields.io/badge/Google_Earth_Engine-4285F4?style=for-the-badge&logo=googleearthengine&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Sentinel-2](https://img.shields.io/badge/Sentinel--2_L2A-0B3D91?style=for-the-badge)
![Cloud Score+](https://img.shields.io/badge/Cloud_Score%2B-34A853?style=for-the-badge)
![Regex](https://img.shields.io/badge/Regex-C0392B?style=for-the-badge)
![Google Drive](https://img.shields.io/badge/Google_Drive-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)
![LaTeX](https://img.shields.io/badge/LaTeX-008080?style=for-the-badge&logo=latex&logoColor=white)
![Markdown](https://img.shields.io/badge/Markdown-000000?style=for-the-badge&logo=markdown&logoColor=white)

![Bahasa](https://img.shields.io/badge/Bahasa-Indonesia-red?style=flat-square)
![Jenis](https://img.shields.io/badge/Jenis-Tutorial_Teknis-orange?style=flat-square)
![Diperbarui](https://img.shields.io/badge/Diperbarui-2026--11--22-blue?style=flat-square)

**Penulis:** Defani Arman Alfitriansyah

---

## Abstrak

Dokumen ini membahas pemanfaatan *regular expression* (regex) untuk memilih band citra Sentinel-2 secara otomatis pada Google Earth Engine (GEE), sehingga daftar nama band tidak perlu ditulis satu per satu. Regex diuraikan secara formal sebagai himpunan pola atas suatu alfabet, kemudian diterapkan pada himpunan nama band untuk menghasilkan himpunan bagian band terpilih. Hasil seleksi dipakai untuk mengekspor citra komposit median, satu band satu berkas GeoTIFF, melalui iterasi otomatis. Sebagai penyaring kualitas piksel pada komposit, digunakan produk **Cloud Score+ (CS+)**; landasan ilmiahnya dijelaskan tersendiri pada **Bab 6** berdasarkan Pasquarella *et al.* (2023).

**Kata kunci:** regular expression, Sentinel-2, Google Earth Engine, seleksi band, komposit median, Cloud Score+.

---

## Tech Stack

| Komponen | Peran dalam tutorial |
|---|---|
| ![GEE](https://img.shields.io/badge/Google_Earth_Engine-4285F4?style=flat-square&logo=googleearthengine&logoColor=white) | Platform pengolahan citra satelit berbasis awan (Code Editor) |
| ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) | Bahasa skrip GEE sisi klien (`evaluate`, `forEach`) |
| ![S2](https://img.shields.io/badge/COPERNICUS%2FS2__SR__HARMONIZED-0B3D91?style=flat-square) | Citra Sentinel-2 level 2A (*surface reflectance*) |
| ![CS+](https://img.shields.io/badge/GOOGLE%2FCLOUD__SCORE__PLUS-34A853?style=flat-square) | Band `cs_cdf` untuk masker kualitas piksel |
| ![Regex](https://img.shields.io/badge/Regex-C0392B?style=flat-square) | Seleksi band otomatis, misalnya `B.*` |
| ![Drive](https://img.shields.io/badge/Google_Drive-4285F4?style=flat-square&logo=googledrive&logoColor=white) | Tujuan ekspor GeoTIFF per band (10 m, EPSG:4326) |
| ![LaTeX](https://img.shields.io/badge/LaTeX-008080?style=flat-square&logo=latex&logoColor=white) | Penulisan rumus (sintaks `$$ ... $$`) |
| ![Markdown](https://img.shields.io/badge/Markdown-000000?style=flat-square&logo=markdown&logoColor=white) | Format dokumen |

---

## Daftar Isi

1. [Pendahuluan](#bab-1-pendahuluan)
2. [Landasan Teori Regular Expression](#bab-2-landasan-teori-regular-expression)
3. [Penerapan Regex pada Band Sentinel-2](#bab-3-penerapan-regex-pada-band-sentinel-2)
4. [Ekspor Otomatis per Band](#bab-4-ekspor-otomatis-per-band)
5. [Implementasi pada Google Earth Engine](#bab-5-implementasi-pada-google-earth-engine)
6. [Landasan Ilmiah Cloud Score+ (Bab CS+)](#bab-6-landasan-ilmiah-cloud-score-bab-cs)
7. [Daftar Notasi](#bab-7-daftar-notasi)
8. [Daftar Pustaka](#bab-8-daftar-pustaka)

> **Konvensi penomoran persamaan.** Persamaan Bab 2 sampai 4 diberi label **R-n** (regex), Bab 5 diberi label **P-n** (pengolahan), dan Bab 6 diberi label **C-n** (Cloud Score+). Rujukan ke persamaan pada makalah sumber ditulis eksplisit sebagai "Pers. n pada makalah".

---

## Bab 1. Pendahuluan

### 1.1 Latar belakang

Citra Sentinel-2 level 2A (*surface reflectance*) pada GEE terdiri atas belasan band: 12 band spektral (B1 sampai B9, B8A, B11, B12) dan beberapa band tambahan khusus L2. Dalam banyak alur kerja, band spektral perlu diproses atau diekspor satu per satu. Pendekatan konvensional menuliskan daftar nama band secara manual:

```javascript
var bands = ['B1','B2','B3','B4','B5','B6','B7','B8','B8A','B9','B11','B12'];
```

Cara ini panjang, mudah salah ketik, dan harus disunting setiap kali band yang dibutuhkan berubah. Regex menyelesaikan masalah ini dengan mengganti daftar eksplisit menjadi satu **aturan** pencocokan:

```javascript
image.select('B.*')
```

### 1.2 Tujuan

1. Menjelaskan dasar formal regex dan operator yang dipakai pada GEE.
2. Menunjukkan cara menurunkan himpunan band terpilih dari suatu pola regex.
3. Menerapkannya pada alur komposit median dan ekspor otomatis per band.
4. Menyajikan landasan ilmiah penyaring kualitas piksel Cloud Score+ pada bab tersendiri (Bab 6).

### 1.3 Ruang lingkup

Materi utama adalah regex dan penerapannya (Bab 2 sampai 5). Bab 6 bersifat pendukung dan dapat dibaca terpisah.

---

## Bab 2. Landasan Teori Regular Expression

### 2.1 Definisi formal

Misalkan $\Sigma$ adalah alfabet (himpunan karakter yang diizinkan) dan $\Sigma^\ast$ adalah himpunan semua string hingga atas $\Sigma$, termasuk string kosong $\varepsilon$. Sebuah *regular expression* $\pi$ mendefinisikan suatu himpunan string, yang disebut bahasa $L(\pi) \subseteq \Sigma^\ast$. String $s$ dikatakan **cocok** dengan $\pi$ apabila $s \in L(\pi)$:

$$
m_{\pi}(s) =
\begin{cases}
1, & s \in L(\pi) \\
0, & \text{lainnya}
\end{cases}
\qquad \text{(R-1)}
$$

Konsep regex berakar pada teori himpunan reguler yang diperkenalkan Kleene (1956) (Pustaka 1), dan algoritma pencarian berbasis automata hingga untuk regex dipublikasikan oleh Thompson (1968) (Pustaka 2).

### 2.2 Operator dasar

Bahasa suatu regex dibangun secara rekursif dari tiga operasi himpunan: gabungan (*union*), penyambungan (*concatenation*), dan bintang Kleene (*Kleene star*):

$$
\begin{aligned}
L(a) &= \lbrace a \rbrace, \quad a \in \Sigma \\
L(\pi_1 \mid \pi_2) &= L(\pi_1) \cup L(\pi_2) \\
L(\pi_1 \pi_2) &= L(\pi_1) \cdot L(\pi_2) = \lbrace uv : u \in L(\pi_1),\ v \in L(\pi_2) \rbrace \\
L(\pi^{\ast}) &= \bigcup_{k=0}^{\infty} L(\pi)^{k}
\end{aligned}
\qquad \text{(R-2)}
$$

dengan $L(\pi)^{0} = \lbrace \varepsilon \rbrace$. Operator tambahan yang praktis dapat diturunkan dari ketiganya:

$$
\begin{aligned}
L(\mathtt{.}) &= \Sigma \\
L(\pi\mathtt{?}) &= L(\pi) \cup \lbrace \varepsilon \rbrace \\
L([x\text{-}y]) &= \lbrace c \in \Sigma : x \le c \le y \rbrace
\end{aligned}
\qquad \text{(R-3)}
$$

### 2.3 Operator yang dipakai pada GEE

Dokumentasi `ee.Image.select` menyatakan bahwa band dapat dipilih berdasarkan nama, indeks numerik, atau regex (Pustaka 7). Contoh pola pada dokumentasi tersebut sejalan dengan operator berikut:

| Operator | Arti | Contoh | Himpunan string cocok |
|---|---|---|---|
| literal | karakter itu sendiri | `B` | hanya string `B` |
| `.` | satu karakter sembarang | `B.` | `B1`, `B2`, ... (dua karakter berawalan B) |
| `*` | nol atau lebih pengulangan | `B.*` | semua string berawalan `B` |
| `?` | nol atau satu kali | `B..?` | string berawalan `B` dengan dua atau tiga karakter |
| `[a-b]` | satu karakter dalam rentang | `B[6-8]` | `B6`, `B7`, `B8` |
| `\|` | pilihan (gabungan) | `B.*\|QA.*` | berawalan `B` atau berawalan `QA` |

### 2.4 Aturan pencocokan penuh

Tutorial ini mengasumsikan **pencocokan penuh**: seluruh nama band harus termasuk dalam $L(\pi)$, bukan hanya sebagian. Dokumentasi GEE tidak menyatakan hal ini secara eksplisit pada halaman yang dikaji, sehingga perilakunya perlu diverifikasi langsung dengan mencetak hasil seleksi:

```javascript
print(image.select('B8.?').bandNames());
```

### 2.5 Regex berbeda dari wildcard sederhana

Pengguna yang terbiasa dengan pola nama berkas (misalnya `B*` untuk semua berkas berawalan B) perlu memperhatikan bahwa pada regex, `*` tidak berdiri sendiri. Ia memodifikasi elemen sebelumnya. Padanan wildcard `B*` pada regex adalah `B.*` (huruf `B`, lalu karakter sembarang berulang).

| Konteks | Pola | Arti |
|---|---|---|
| Wildcard nama berkas | `B*` | berawalan B, lanjutannya apa saja |
| Regex | `B*` | nol atau lebih huruf B (bukan yang dimaksud) |
| Regex | `B.*` | huruf B, lalu karakter apa saja berulang |

---

## Bab 3. Penerapan Regex pada Band Sentinel-2

### 3.1 Himpunan band dan himpunan terpilih

Misalkan $\mathcal{B}$ adalah himpunan nama band pada citra. Seleksi dengan pola $\pi$ menghasilkan himpunan bagian:

$$
\mathcal{B}_{\pi} = \lbrace b \in \mathcal{B} : m_{\pi}(b) = 1 \rbrace
\qquad \text{(R-4)}
$$

Untuk pola `B.*`, bahasanya adalah penyambungan huruf `B` dengan string apa pun:

$$
L(\mathtt{B.\ast}) = \lbrace \mathtt{B} \rbrace \cdot \Sigma^{\ast}
\qquad \text{(R-5)}
$$

Pada citra Sentinel-2 L2A, himpunan terpilih adalah band spektral:

$$
\mathcal{B}_{\mathtt{B.\ast}} = \lbrace \mathtt{B1}, \mathtt{B2}, \mathtt{B3}, \mathtt{B4}, \mathtt{B5}, \mathtt{B6}, \mathtt{B7}, \mathtt{B8}, \mathtt{B8A}, \mathtt{B9}, \mathtt{B11}, \mathtt{B12} \rbrace, \qquad \left| \mathcal{B}_{\mathtt{B.\ast}} \right| = 12
\qquad \text{(R-6)}
$$

Band lain seperti `AOT`, `WVP`, `SCL`, dan `TCI_R` tidak termasuk karena nama-nama tersebut tidak diawali huruf `B` (dokumentasi `S2_SR_HARMONIZED` memuat 12 band spektral dan beberapa band khusus L2; Pustaka 5).

### 3.2 Contoh pola dan hasilnya

Pola berikut disusun dengan operator pada Tabel Bab 2.3. Hasil diturunkan berdasarkan asumsi pencocokan penuh (Bagian 2.4).

| Pola | Himpunan band terpilih | Jumlah |
|---|---|---|
| `B.*` | B1 sampai B9, B8A, B11, B12 | 12 |
| `B[2-4]` | B2, B3, B4 | 3 |
| `B8.?` | B8, B8A | 2 |
| `B1.*` | B1, B11, B12 | 3 |
| `B[1-9][1-2]` | B11, B12 | 2 |
| `B(2\|3\|4)` | B2, B3, B4 | 3 |

Pola `B[1-9][1-2]` dan `B.*|QA.*` termasuk contoh yang dipakai pada dokumentasi resmi (Pustaka 7, 8).

### 3.3 Verifikasi hasil seleksi

Selalu cetak nama band hasil seleksi untuk memastikan pola bekerja sesuai harapan:

```javascript
print('Band terpilih:', image.select('B.*').bandNames());
```

Cara ini juga membantu menangkap kesalahan pola, karena regex yang salah tidak selalu menimbulkan pesan yang jelas.

---

## Bab 4. Ekspor Otomatis per Band

### 4.1 Daftar band sebagai objek sisi-server

`bandNames()` mengembalikan objek `ee.List` yang berada di sisi server GEE. Agar dapat diiterasi dengan `forEach` JavaScript pada sisi klien, daftar tersebut harus diambil dahulu melalui `.evaluate()`.

### 4.2 Himpunan tugas ekspor

Setiap band terpilih memetakan ke satu tugas ekspor $E_b$. Himpunan tugas yang dibentuk adalah:

$$
\mathcal{E} = \lbrace E_b : b \in \mathcal{B}_{\pi} \rbrace, \qquad |\mathcal{E}| = |\mathcal{B}_{\pi}|
\qquad \text{(R-7)}
$$

Untuk pola `B.*` pada L2A, terbentuk 12 tugas. Mengubah pola otomatis mengubah jumlah tugas tanpa menyunting kode lain.

### 4.3 Alur

| Tahap | Perintah | Fungsi |
|---|---|---|
| 1 | `s2.bandNames()` | Mengambil daftar nama band |
| 2 | `.evaluate(function(names) {...})` | Memindahkan daftar ke sisi klien |
| 3 | `names.forEach(function(b) {...})` | Mengulang untuk tiap band, `b` = B1, B2, ... |
| 4 | `Export.image.toDrive({...})` | Membuat satu tugas ekspor per band |

---

## Bab 5. Implementasi pada Google Earth Engine

**Prasyarat:** variabel geometri area studi bernama `batas_wilayah` telah dibuat pada panel *Imports*.

### 5.1 Formulasi tahapan pengolahan

Misalkan $i$ menandai piksel, $t$ tanggal akuisisi pada himpunan $T$ (hasil filter wilayah, tanggal, dan persentase awan), dan $b$ band.

**Threshold masking** dengan threshold $\tau$ pada band `cs_cdf` (penjelasan ilmiahnya pada Bab 6):

$$
M_t(i) =
\begin{cases}
1, & \mathrm{CS}^{+}_{\mathrm{cdf},t}(i) \ge \tau \\
0, & \text{lainnya}
\end{cases}
\qquad \text{(P-1)}
$$

Pada skrip, $\tau = 0{,}60$ (operasi `.gte(0.60)`).

**Konversi ke reflektansi.** Nilai digital L2A berskala 10.000 (Pustaka 5), sehingga:

$$
\rho_{b,t}(i) = 0{,}0001 \times \mathrm{DN}_{b,t}(i)
\qquad \text{(P-2)}
$$

**Komposit median** dari piksel yang lolos masker, untuk tiap band $b \in \mathcal{B}_{\pi}$:

$$
\tilde{\rho}_b(i) = \underset{t \in T}{\operatorname{median}} \lbrace \rho_{b,t}(i) : M_t(i) = 1 \rbrace
\qquad \text{(P-3)}
$$

Median bersifat tahan terhadap nilai pencilan, sehingga sisa awan tipis dan bayangan yang lolos masker cenderung tertekan.

### 5.2 Kode lengkap

```javascript
// CLOUD SCORE+
var csPlus = ee.ImageCollection('GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED');

// SENTINEL-2 L2A: FILTER, MASKING, KOMPOSIT
var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
  .filterBounds(batas_wilayah)
  .filterDate('2026-01-01', '2026-11-20')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
  .linkCollection(csPlus, ['cs_cdf'])
  .map(function(image) {
    var mask = image.select('cs_cdf').gte(0.60);   // MASK, THRESHOLD 0,60 (P-1)
    return image.select('B.*')                      // SELEKSI BAND REGEX (R-4, R-6)
      .multiply(0.0001)                             // SKALA REFLEKTANSI (P-2)
      .updateMask(mask);
  })
  .median()                                         // KOMPOSIT MEDIAN (P-3)
  .clip(batas_wilayah);

// TAMPILKAN DI PETA
Map.centerObject(batas_wilayah, 15);
Map.addLayer(s2, {bands: ['B11', 'B8', 'B4'], min: 0, max: 0.3}, 'Komposit Citra');

// EKSPOR PER BAND (R-7)
s2.bandNames().evaluate(function(names) {
  names.forEach(function(b) {
    Export.image.toDrive({
      image: s2.select(b).toFloat(),
      description: 'S2A_' + b,
      folder: 'GEE_Sentinel2_2026',
      fileNamePrefix: 'S2A_' + b,
      region: batas_wilayah,
      scale: 10,
      crs: 'EPSG:4326',
      maxPixels: 1e13
    });
  });
});
```

### 5.3 Catatan implementasi

- **Dua kali seleksi regex berbeda tujuan.** `select('B.*')` pada `.map()` menyeleksi band dari citra sumber. Setelah komposit terbentuk, `s2.bandNames()` sudah hanya berisi band terpilih, sehingga loop ekspor tidak perlu menyeleksi lagi.
- **Resolusi.** Band 20 m dan 60 m di-*resample* ke 10 m pada ekspor dengan `scale: 10`.
- **Galat nama tidak selalu memunculkan pesan.** Salah ketik pada nama properti (misalnya `CLOUDY_PIXEL_PERCENTAGE`) atau nama band (`cs_cdf`) dapat menghasilkan koleksi kosong tanpa galat yang jelas. Periksa jumlah citra dengan `print('Jumlah citra:', koleksi.size());`.
- **Pola yang tidak cocok.** Jika regex tidak cocok dengan band mana pun, hasil seleksi kosong atau GEE melaporkan galat. Gunakan `print(...bandNames())` untuk memeriksa (Bagian 3.3).

### 5.4 Alternatif: satu berkas multiband

Jika tidak harus dipisah per band, loop tidak diperlukan:

```javascript
Export.image.toDrive({
  image: s2.toFloat(),
  description: 'S2A_all_bands',
  folder: 'GEE_Sentinel2_2026',
  fileNamePrefix: 'S2A_all_bands',
  region: batas_wilayah,
  scale: 10,
  crs: 'EPSG:4326',
  maxPixels: 1e13
});
```

Hanya satu tugas, tetapi band 20 m dan 60 m ikut di-*resample* ke 10 m dan ukuran berkas besar untuk area luas.

---

## Bab 6. Landasan Ilmiah Cloud Score+ (Bab CS+)

Bab ini menjelaskan dasar ilmiah produk Cloud Score+ yang dipakai pada masker (P-1). Seluruh uraian teoretis mengacu pada Pasquarella *et al.* (2023) (Pustaka 3), kecuali dinyatakan lain.

### 6.1 Motivasi

Pendekatan operasional seperti QA60 Sentinel-2, SCL Sen2Cor, dan Fmask mengandalkan uji spektral dan heuristik. Pendekatan ini kaku dan rentan gagal pada permukaan berpantulan tinggi (salju, bangunan, pantai), target gelap yang mirip bayangan awan, wilayah pegunungan, dan awan sirus tipis. Penulis mengusulkan agar kuantifikasi gangguan atmosfer diperlakukan sebagai **masalah regresi** karena atmosfer bersifat kontinu, bukan klasifikasi diskret.

### 6.2 Model kualitas piksel (QA)

Skor kualitas didefinisikan pada rentang $[0, 1]$. Piksel terekam $p_m$ dimodelkan sebagai kombinasi linear antara reflektansi sebenarnya $p_t$ dan permukaan yang terganggu (awan, bayangan, atau atmosfer) $c$. Skor kualitas $q$ adalah koefisien pada suku reflektansi sebenarnya (Pers. 1 pada makalah):

$$
\mathrm{QA}(p_m) = q, \qquad p_m = p_t\,q + c\,(1 - q)
\qquad \text{(C-1)}
$$

"Reflektansi sebenarnya" adalah reflektansi puncak atmosfer pada kondisi kedalaman optik minimum. Karena modelnya linear, kombinasi linear dua piksel yang skornya diketahui menghasilkan piksel dengan skor berupa kombinasi linear skor keduanya. Sifat ini dimanfaatkan pada tahap *fine-tuning* melalui teknik *mixup*.

### 6.3 Kemiripan atmosferik (ASIM)

Mengestimasi kemiripan atmosferik antara dua citra lebih mudah daripada mengestimasi kualitas satu citra secara langsung. ASIM untuk piksel ke-$i$ pada citra $x$ dan $y$ didefinisikan sebagai QA minimum di antara keduanya (Pers. 2 pada makalah):

$$
\mathrm{ASIM}_i(x_i, y_i) := \min_{p \in \lbrace x_i,\, y_i \rbrace} \mathrm{QA}(p)
\qquad \text{(C-2)}
$$

Jika $r$ adalah citra referensi bebas gangguan, maka $\mathrm{QA}(r) = 1$ sehingga:

$$
\mathrm{ASIM}(x, r) = \mathrm{QA}(x)
\qquad \text{(C-3)}
$$

Dengan citra referensi yang bersih, skor kualitas citra $x$ dapat diperoleh langsung dari ASIM. Prinsip ini dipakai untuk membangun data latih secara *bootstrapping*: model ASIM menemukan pasangan citra bersih pada deret waktu, lalu skor QA tiap citra dihitung terhadap referensi terdekat. Pasangan dianggap referensi jernih bila $\mu - \sigma > 0{,}75$. Proses ini menghasilkan 2,2 juta rangkaian video untuk tahun 2018 sampai 2021.

### 6.4 Normalisasi masukan model

Seluruh masukan model ditransformasi logaritmik untuk menekan ekor panjang reflektansi dari permukaan berpantulan tinggi (Pers. 3 pada makalah):

$$
x' = \frac{\log(x + 1)}{10}
\qquad \text{(C-4)}
$$

Transformasi ini menormalkan masukan ke rentang sekitar $[0,\ 1{,}1]$.

### 6.5 Fungsi kerugian kemungkinan maksimum

Model memprediksi rerata dan log-varians $(\mu, \log\sigma^2)$ dari skor QA. Parameter dioptimalkan dengan meminimalkan log-likelihood teregularisasi (Pers. 4 sampai 6 pada makalah). Pertama, log-varians dibatasi (di-*clip*) pada rentang $[a, b]$:

$$
z = \min\left( \max\left( \log\sigma^2,\ a \right),\ b \right)
\qquad \text{(C-5)}
$$

Kemudian suku regularisasi dihitung:

$$
r = \left( z - \log\sigma^2 \right)^2
\qquad \text{(C-6)}
$$

Kerugian akhirnya:

$$
l = \frac{1}{2}\left( \log 2\pi + z + \frac{(\mu' - \mu)^2}{e^{z}} \right) + r
\qquad \text{(C-7)}
$$

dengan $\mu'$ skor QA acuan, domain $z$ dibatasi pada $[a, b]$ dengan $a = -16$ dan $b = 5$ demi stabilitas numerik, dan $r$ ditambahkan untuk mencegah dataran gradien.

### 6.6 Skor persentil sebagai luaran produk

Model QA bersifat multi-tugas. Yang dipublikasikan sebagai produk CS+ adalah estimasi **persentil** skor QA, dilatih dengan meminimalkan (Pers. 7 pada makalah):

$$
\mathcal{L}_p = \left( p(x, \theta) - \mathbf{1}\left[ \mathrm{QA}_\mu(x) \ge \mathrm{QA}_\mu(y) \right] \right)^2
\qquad \text{(C-8)}
$$

Karena $x$ dan $y$ diambil dari sebaran yang sama pada lokasi tertentu, ekspektasi indikator tersebut adalah **fungsi sebaran kumulatif** (CDF) QA untuk citra $x$:

$$
\mathbb{E}\left[ \mathbf{1}\left[ \mathrm{QA}_\mu(x) \ge \mathrm{QA}_\mu(y) \right] \right] = F_{\mathrm{QA}}\left( \mathrm{QA}_\mu(x) \right)
\qquad \text{(C-9)}
$$

Secara praktis, nilai CS+ menyatakan posisi relatif kejernihan piksel dibandingkan pengamatan lain pada lokasi yang sama.

### 6.7 Koreksi sambungan antar-*tile* saat inferensi

Pada inferensi, citra dibagi menjadi sub-*tile*, dan tiap sub-*tile* memperoleh himpunan citra pendukung (maksimum 16) yang dapat berbeda sehingga menimbulkan sambungan (*seam*) visual. Untuk meredamnya, dilakukan optimasi global terhadap *gain* dan *bias* $(\gamma, \beta)$ pada empat titik kontrol per sub-*tile* dengan interpolasi bilinear, dengan $\mu' = \gamma\mu + \beta$ dan $\sigma' = \gamma\sigma$ (Pers. 8 pada makalah):

$$
\min_{\gamma,\beta}\ \lambda \sum_{T} \left[ (\gamma_T - 1)^2 + \beta_T^2 \right] + \sum_{i} \left( \gamma_{Q_i} Q_i + \beta_{Q_i} - \gamma_{P_i} P_i - \beta_{P_i} \right)^2
\qquad \text{(C-10)}
$$

dengan $\lambda = 1$, serta $P$ dan $Q$ adalah skor rerata dua sub-*tile* bersebelahan sepanjang sambungan. Menurut penulis, koreksi ini tidak berpengaruh signifikan pada validasi tetapi mengurangi artefak visual secara nyata.

### 6.8 Band `cs` dan `cs_cdf` pada GEE

Dataset `GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED` memuat dua band QA, `cs` dan `cs_cdf`, yang keduanya menilai kegunaan piksel terhadap visibilitas permukaan pada skala kontinu 0 sampai 1 (0 berarti tidak jernih atau terhalang, 1 berarti jernih), pada resolusi 10 m (Pustaka 4). Menurut dokumentasi, `cs` dihitung berdasarkan jarak spektral terhadap referensi jernih teoretis. Makalah sumber menyebut bahwa luaran yang dipublikasikan sebagai CS+ adalah estimasi persentil/CDF (Bagian 6.6); penamaan band mengikuti dokumentasi dataset, bukan makalah.

Citra pada koleksi CS+ memiliki `system:index` yang sama dengan citra Sentinel-2 L1C sumbernya sehingga dapat ditautkan dengan `linkCollection`. CS+ dibuat dari data L1C, tetapi dapat diterapkan pada koleksi L1C maupun L2A (Pustaka 4). Registrasi L1C dan L2A identik (Pasquarella *et al.*, 2023).

### 6.9 Pemilihan threshold $\tau$

Makalah sumber menguji threshold 0 sampai 1 pada interval 0,01, lalu memilih nilai dengan galat omisi dan komisi paling seimbang:

| Dataset acuan | Threshold yang direkomendasikan |
|---|---|
| Tarrio | 0,64 |
| CMIX-I PixBox (semua titik) | 0,50 |
| CMIX-I PixBox (tanpa bayangan) | 0,48 |

Perbedaan threshold ditafsirkan penulis sebagai akibat label CMIX-I yang lebih terpisahkan serta cakupan kondisi geografis dan atmosferik yang lebih beragam. Threshold hanyalah rekomendasi; pengguna dapat memilih nilai sesuai kebutuhan, atau memakai skor kontinu langsung sebagai bobot pengamatan atau untuk komposit "piksel terjernih".

Threshold $\tau = 0{,}60$ pada skrip berada di antara nilai-nilai tersebut. Threshold yang lebih tinggi lebih ketat (lebih banyak piksel dibuang), sehingga pada wilayah yang sering berawan perlu dipertimbangkan penurunan threshold atau pelonggaran filter `CLOUDY_PIXEL_PERCENTAGE` agar komposit tidak berlubang.

### 6.10 Hasil evaluasi pada literatur

Nilai berikut dikutip dari Pasquarella *et al.* (2023); metrik dihitung setelah luaran dibinerkan menjadi kelas *clear* dan *not clear*.

**Tabel 6.1. Dataset Tarrio (2.681 titik interpretasi)**

| Metode | Threshold | F1 | Akurasi keseluruhan | Omisi | Komisi |
|---|---|---|---|---|---|
| **CS+** | 0,64 | **0,8466** | **0,8096** | 0,1889 | 0,1933 |
| Fmask v2 | - | 0,8133 | 0,7742 | 0,2411 | 0,1976 |
| s2cloudless | 0,12 | 0,7589 | 0,7111 | 0,2985 | 0,2711 |
| Sen2Cor | - | 0,5190 | 0,5675 | 0,6399 | 0,0508 |
| QA60 | - | 0,4579 | 0,5401 | 0,7003 | 0,0173 |

**Tabel 6.2. Dataset CMIX-I PixBox (17.351 piksel)**

| Metode | Acuan | Threshold | F1 | Akurasi keseluruhan |
|---|---|---|---|---|
| **CS+** | semua titik | 0,50 | **0,8816** | 0,8768 |
| **CS+** | tanpa bayangan | 0,48 | 0,8800 | **0,8841** |
| s2cloudless | tanpa bayangan | 0,24 | 0,8419 | 0,8471 |
| s2cloudless | semua titik | 0,18 | 0,8162 | 0,8094 |

Pada Tabel 6.1, CS+ memiliki F1 dan akurasi keseluruhan tertinggi, sedangkan produk yang tersedia luas seperti Sen2Cor dan QA60 menunjukkan kinerja terendah. Pada Tabel 6.2, CS+ konsisten mengungguli s2cloudless pada seluruh skema pelabelan. Selisih terbesar muncul pada acuan "semua titik", wajar karena s2cloudless tidak mengidentifikasi bayangan awan sebagai kualitas buruk.

### 6.11 Keterbatasan

Menurut Pasquarella *et al.* (2023), kelemahan CS+ yang diketahui meliputi deteksi bayangan awan di atas permukaan air, kapal bergerak yang terkarakterisasi keliru sebagai kualitas buruk, dan ketidakpastian tinggi di atas bongkahan es aktif. Karena model mengestimasi rerata sekaligus varians, varians dapat dipakai untuk menyaring hasil lebih lanjut. Validasi memakai titik interpretasi pada dua dataset acuan, sehingga performa pada wilayah dan kondisi lain (misalnya tropis berawan sepanjang tahun) perlu diperiksa secara visual. Untuk komposit median, jumlah citra yang sedikit membuat median kurang stabil, dan piksel tanpa observasi bersih akan kosong.

---

## Bab 7. Daftar Notasi

| Simbol | Keterangan |
|---|---|
| $\Sigma$, $\Sigma^\ast$ | Alfabet karakter dan himpunan semua string atasnya |
| $\varepsilon$ | String kosong |
| $\pi$ | Pola regex |
| $L(\pi)$ | Bahasa (himpunan string cocok) dari pola $\pi$ |
| $m_\pi(s)$ | Fungsi indikator kecocokan string $s$ dengan pola $\pi$ |
| $\mathcal{B}$ | Himpunan nama band pada citra |
| $\mathcal{B}_\pi$ | Himpunan band hasil seleksi pola $\pi$ |
| $E_b$, $\mathcal{E}$ | Tugas ekspor band $b$ dan himpunan seluruh tugas |
| $M_t(i)$ | Masker biner piksel $i$ pada tanggal $t$ |
| $\tau$ | Threshold skor CS+ untuk masking |
| $\rho_{b,t}(i)$ | Reflektansi band $b$, tanggal $t$, piksel $i$ |
| $\tilde{\rho}_b(i)$ | Reflektansi komposit median |
| $p_m$, $p_t$, $c$ | Piksel terekam, reflektansi sebenarnya, permukaan terganggu |
| $q$, $\mathrm{QA}$ | Skor kualitas piksel pada rentang $[0, 1]$ |
| $\mathrm{ASIM}$ | Kemiripan atmosferik antara dua citra |
| $(\mu, \log\sigma^2)$ | Rerata dan log-varians estimasi QA |
| $\mu'$ | Skor QA acuan (*ground truth*) |
| $F_{\mathrm{QA}}$ | Fungsi sebaran kumulatif QA |
| $(\gamma, \beta)$ | *Gain* dan *bias* koreksi sambungan antar-*tile* |

---

## Bab 8. Daftar Pustaka

**Teori regular expression**

[1] Kleene, S. C. (1956). Representation of events in nerve nets and finite automata. Dalam C. E. Shannon & J. McCarthy (Eds.), *Automata Studies* (hlm. 3–41). Princeton University Press.

[2] Thompson, K. (1968). Programming techniques: Regular expression search algorithm. *Communications of the ACM*, 11(6), 419–422.

**Cloud Score+**

[3] Pasquarella, V. J., Brown, C. F., Czerwinski, W., & Rucklidge, W. J. (2023). *Comprehensive quality assessment of optical satellite imagery using weakly supervised video learning*. Dalam Prosiding IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW), hlm. 2125–2135. DOI: [10.1109/CVPRW59228.2023.00206](https://doi.org/10.1109/CVPRW59228.2023.00206)

**Dokumentasi Google Earth Engine** (diakses 22 November 2026)

[4] Google Earth Engine. *Cloud Score+ S2_HARMONIZED V1* (`GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED`). Lisensi CC-BY-4.0.
https://developers.google.com/earth-engine/datasets/catalog/GOOGLE_CLOUD_SCORE_PLUS_V1_S2_HARMONIZED

[5] Uni Eropa/ESA/Copernicus. *Harmonized Sentinel-2 MSI: MultiSpectral Instrument, Level-2A (SR)* (`COPERNICUS/S2_SR_HARMONIZED`). Penggunaan data tunduk pada Copernicus Sentinel Data Terms and Conditions.
https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED

[6] Uni Eropa/ESA/Copernicus. *Harmonized Sentinel-2 MSI: MultiSpectral Instrument, Level-1C* (`COPERNICUS/S2_HARMONIZED`). Koleksi sumber pembuatan Cloud Score+.
https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_HARMONIZED

[7] Google Earth Engine. *ee.Image.select*. Dokumentasi API.
https://developers.google.com/earth-engine/apidocs/ee-image-select

[8] Google Earth Engine. *ee.ImageCollection.select*. Dokumentasi API.
https://developers.google.com/earth-engine/apidocs/ee-imagecollection-select

---
