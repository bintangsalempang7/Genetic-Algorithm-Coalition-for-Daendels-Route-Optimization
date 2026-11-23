# Optimasi Rute Jalan Daendels Anyer–Panarukan dengan Algoritma Genetika Metode Koalisi

Proyek ini menerapkan **Algoritma Genetika (Genetic Algorithm/GA) metode koalisi** untuk mencari urutan perjalanan dengan total jarak terpendek pada koridor Anyer–Panarukan. Permasalahan dimodelkan sebagai **Open Asymmetric Traveling Salesman Problem (Open ATSP)**: perjalanan dimulai dari Anyer, berakhir di Panarukan, dan setiap lokasi dikunjungi satu kali tanpa kembali ke titik awal.

Proyek dibuat menggunakan Python dan Jupyter Notebook sebagai tugas mata kuliah **Teknik Optimalisasi**, Universitas Kristen Satya Wacana (2026).

## Isi repositori

| File | Keterangan |
|---|---|
| `Teknik Optimalisasi_Daendels_GA_Koalisi.ipynb` | Notebook berisi data lokasi, pembentukan matriks jarak, implementasi GA Koalisi, eksperimen, visualisasi, validasi, dan ekspor hasil. |
| `Laporan_Proyek_Daendels_GA_Koalisi_BAB_I-V.pdf` | Laporan proyek yang menjelaskan latar belakang, teori, metodologi, hasil, pembahasan, kesimpulan, dan saran. |
| `README.md` | Panduan ringkas mengenai proyek dan cara menjalankan notebook. |

## Tujuan proyek

1. Menerapkan GA metode koalisi untuk mengoptimalkan rute Anyer–Panarukan.
2. Menentukan urutan kunjungan untuk 52 titik lokasi dengan total jarak yang lebih rendah.
3. Membandingkan hasil GA Koalisi dengan rute referensi dan metode Nearest Neighbor.
4. Menguji konsistensi hasil melalui 10 percobaan dan membandingkan GA Koalisi dengan GA tanpa koalisi.

## Data dan pemodelan

- **Jumlah lokasi:** 52 titik, terdiri atas Anyer sebagai titik awal, 50 titik perantara, dan Panarukan sebagai titik akhir.
- **Cakupan wilayah:** Banten, DKI Jakarta, Jawa Barat, Jawa Tengah, dan Jawa Timur.
- **Data lokasi:** ID, nama lokasi, kabupaten/kota, provinsi, jenis wilayah, latitude, dan longitude. Nama serta hierarki wilayah dicocokkan dengan kode wilayah administrasi; koordinat merujuk pada data Kemendagri/BIG melalui paket geografis versi 1.3.2.
- **Representasi jarak:** matriks jarak berarah berukuran `52 × 52`, dalam kilometer.
- **Fungsi tujuan:** meminimalkan jumlah jarak pada 51 ruas perjalanan yang menghubungkan seluruh lokasi secara berurutan.

### Sumber matriks jarak

Berdasarkan metadata hasil yang digunakan dalam laporan, matriks jarak berasal dari **OSRM public server** melalui Table Service dengan profil **`DRIVING`**. Jarak mengikuti jaringan jalan dan bersifat berarah. Karena itu, jarak dari lokasi A ke B tidak harus sama dengan jarak dari B ke A.

Notebook juga memuat kode integrasi Google Routes API dengan mode `TWO_WHEELER`. Namun, konfigurasi bawaan `API_MODE="reuse"` tidak mengambil matriks baru dari Google Routes API. Matriks yang dipakai dalam hasil laporan dibentuk melalui OSRM dengan profil `DRIVING`. Beberapa keterangan atau judul visualisasi di notebook masih menyebut Google Routes API atau `TWO_WHEELER`; untuk membaca hasil eksperimen pada laporan, gunakan keterangan sumber pada `matrix_meta.json`, yaitu OSRM `DRIVING_OSRM`.

## Metode Algoritma Genetika Koalisi

Metode koalisi membentuk pasangan orang tua dari dua kelompok:

- **Bapak:** individu elit, yaitu kandidat dengan jarak terbaik dari populasi saat ini.
- **Ibu:** individu pendatang baru, yang digunakan untuk menambah keragaman populasi. Kandidat dapat berasal dari urutan acak, Nearest Neighbor teracak, atau individu elit yang diacak.
- **Anak:** solusi baru yang dibentuk dari proses crossover, mutasi, perbaikan kromosom, dan kemungkinan local search.

Tahapan utama algoritma:

1. Membentuk populasi awal dari kandidat acak dan kandidat heuristik.
2. Mengevaluasi total jarak setiap kromosom.
3. Memilih Bapak dari kelompok elit dan Ibu dari kelompok pendatang baru.
4. Membentuk Anak menggunakan Order Crossover (OX).
5. Melakukan mutasi dengan metode swap atau inversion.
6. Memperbaiki kromosom agar setiap lokasi perantara muncul tepat satu kali dan titik awal serta akhir tetap.
7. Menerapkan local search sesuai probabilitas.
8. Menggabungkan populasi lama, Anak, dan pendatang baru, kemudian memilih populasi generasi berikutnya dengan elitisme.
9. Menghentikan pencarian ketika jumlah generasi maksimum tercapai atau tidak ada perbaikan selama 75 generasi.

### Parameter eksperimen

| Parameter | Nilai |
|---|---:|
| Ukuran populasi | 100 individu |
| Maksimum generasi | 500 |
| Crossover rate | 0.90 |
| Mutation rate | 0.15 |
| Jumlah elit | 5 individu |
| Newcomer rate | 0.10 |
| Jumlah anak per generasi | 100 |
| Peluang local search | 0.50 |
| Percobaan local search | 20 |
| Patience | 75 generasi |
| Crossover | Order Crossover (OX) |
| Mutasi | Campuran swap dan inversion |
| Diversity keep | 0.20 |
| Jumlah percobaan | 10 (seed 42–51) |

Fitness dihitung menggunakan rumus `1 / (1 + Z)`, dengan `Z` sebagai total jarak rute. Pemilihan solusi terbaik tetap didasarkan pada total jarak terkecil.

## Hasil utama

Solusi terbaik yang ditemukan dalam eksperimen memiliki total jarak **1,286.906 km**, diperoleh pada seed `42` dan pertama kali ditemukan pada generasi ke-`60`.

### Perbandingan metode

| Metode | Total jarak |
|---|---:|
| Rute referensi | 1,309.068 km |
| Nearest Neighbor | 1,293.473 km |
| **GA Koalisi** | **1,286.906 km** |

GA Koalisi menghasilkan jarak sekitar **1.693% lebih rendah** daripada rute referensi dan **0.508% lebih rendah** daripada Nearest Neighbor.

### Eksperimen 10 kali

- Seluruh 10 percobaan GA Koalisi memperoleh jarak terbaik yang sama, yaitu `1,286.906 km`.
- Rata-rata generasi saat solusi terbaik ditemukan adalah `61.6` generasi.
- Rata-rata generasi yang dijalankan adalah `136.6` generasi.
- Dalam pengujian yang dilaporkan, GA Koalisi memiliki rata-rata waktu komputasi `7.845` detik, sedangkan GA tanpa koalisi `8.641` detik.
- Rata-rata generasi terbaik GA tanpa koalisi adalah `88.3` generasi.

Angka tersebut merupakan hasil eksperimen pada data, parameter, dan lingkungan komputasi yang digunakan. Hasil ini tidak membuktikan bahwa solusi yang ditemukan merupakan optimum global.

## Cara menjalankan notebook

Notebook dapat dijalankan melalui **Google Colab** atau Jupyter Notebook. Google Colab merupakan pilihan yang paling praktis karena konfigurasi awal notebook sudah mendukung Google Drive.

### Opsi 1 — Google Colab

1. Unggah file `Teknik Optimalisasi_Daendels_GA_Koalisi.ipynb` ke Google Drive atau buka notebook melalui Google Colab.
2. Jalankan sel notebook secara berurutan dari atas ke bawah.
3. Sel instalasi akan memasang pustaka yang diperlukan.
4. Pada konfigurasi bawaan, `USE_DRIVE=True` sehingga notebook akan meminta izin untuk mengakses Google Drive dan menyimpan hasil pada folder `MyDrive/daendels_ga`.
5. Notebook membuat dataset 52 lokasi dan menyimpannya sebagai `cities.csv`.
6. Jalankan bagian pembentukan matriks jarak OSRM sebelum menjalankan algoritma genetika.
7. Lanjutkan eksperimen 10 kali, perbandingan metode, visualisasi, validasi, dan ekspor hasil.

### Opsi 2 — Jupyter Notebook lokal

1. Pastikan Python dan Jupyter Notebook sudah terpasang.
2. Buka notebook di Jupyter.
3. Pada bagian konfigurasi, ubah pengaturan penyimpanan berikut agar menggunakan folder lokal:

   Pada dictionary `CONFIG` di notebook, ubah dua nilai berikut:

   ```python
   USE_DRIVE=False,
   WORK_DIR="./daendels_ga",
   ```

   Pertahankan parameter `CONFIG` lainnya.
4. Jalankan sel dari atas ke bawah dan pastikan komputer terhubung ke internet ketika matriks jarak OSRM perlu diambil.

### Pustaka yang digunakan

Notebook memasang pustaka berikut pada sel instalasi:

```bash
pip install folium openpyxl seaborn requests pandas numpy matplotlib
```

Pustaka utama yang digunakan:

- `numpy` dan `pandas` untuk pengolahan data serta perhitungan matriks.
- `matplotlib` dan `seaborn` untuk grafik dan heatmap.
- `requests` untuk mengambil data jarak dari layanan routing.
- `folium` untuk membuat peta interaktif.
- `openpyxl` untuk ekspor hasil ke Excel.

### Apakah perlu Google Maps API Key?

**Tidak untuk menjalankan eksperimen utama dengan konfigurasi bawaan.**

- `API_MODE="reuse"` berarti bagian Google Routes API hanya menggunakan cache yang tersedia dan tidak melakukan panggilan API baru.
- Bagian pembentukan matriks OSRM mengambil data dari server publik OSRM. Bagian ini memerlukan koneksi internet, tetapi tidak memerlukan Google Maps API Key.
- Jika `API_MODE` sengaja diubah menjadi `"first"` atau `"refresh"`, bagian Google Routes API memerlukan API Key yang valid dan dapat menimbulkan biaya sesuai ketentuan Google Maps Platform.
- Jangan menuliskan API Key ke notebook atau mengunggahnya ke repositori publik. Simpan sebagai environment variable atau Colab Secret dengan nama `GOOGLE_MAPS_API_KEY`.

## File keluaran

Notebook menyimpan hasil pada folder kerja `daendels_ga` (secara bawaan di Google Drive jika dijalankan di Colab). Beberapa keluaran yang dibuat:

| Keluaran | Isi |
|---|---|
| `cities.csv` | Data 52 lokasi. |
| `distance_matrix.csv` | Matriks jarak jalan dalam kilometer. |
| `duration_matrix_minutes.csv` | Matriks estimasi durasi dalam menit. |
| `matrix_meta.json` | Metadata sumber dan konfigurasi matriks jarak. |
| `distance_matrix.xlsx` | Matriks jarak dan durasi dalam format Excel. |
| `best_route.csv` | Urutan rute terbaik beserta jarak tiap ruas dan jarak kumulatif. |
| `comparison.csv` | Perbandingan rute referensi, Nearest Neighbor, dan GA Koalisi. |
| `experiments_10runs.csv` | Hasil 10 percobaan GA Koalisi. |
| `validation.csv` | Hasil pemeriksaan 10 kriteria validasi. |
| `results.xlsx` | Ringkasan hasil dalam beberapa sheet Excel. |
| `results.json` | Ringkasan hasil eksperimen dan metadata. |
| `results_summary.html` | Ringkasan hasil yang dapat dibuka melalui browser. |
| `convergence.png` | Grafik konvergensi GA. |
| `heatmap_distance.png` | Heatmap matriks jarak berarah. |
| `comparison.png` | Grafik perbandingan total jarak. |
| `route_map_folium.html` | Peta rute interaktif berbasis Folium. |
| `hasil_ga_koalisi.zip` | Arsip ZIP berisi file keluaran pada folder kerja. |

File rute alternatif dan file peta dapat bergantung pada keberhasilan pengambilan data rute pada sel terkait.

## Validasi dan keterbatasan

Notebook memeriksa 10 kriteria validasi, termasuk jumlah lokasi, kelengkapan kunjungan, duplikasi, titik awal dan akhir, ketersediaan jarak setiap ruas, kesesuaian total jarak, verifikasi nama/koordinat, dan penggunaan jarak berbasis jalan. Laporan menyatakan seluruh 10 kriteria lulus.

Hal-hal yang perlu diperhatikan:

1. Matriks jarak yang digunakan berasal dari profil OSRM `DRIVING`, bukan estimasi khusus sepeda motor (`TWO_WHEELER`).
2. Koordinat merupakan titik representatif wilayah. Metadata mencatat snapping maksimum sekitar `852.72 m`, sehingga ketepatan titik terhadap akses jalan masih perlu diperiksa.
3. Rute referensi merupakan urutan ID dataset dari barat ke timur, bukan hasil verifikasi historis jalur Jalan Daendels secara persis.
4. Algoritma Genetika merupakan metode heuristik. Solusi terbaik dari eksperimen tidak menjamin tercapainya optimum global.
5. Jarak hasil optimasi tidak dimaksudkan sebagai petunjuk navigasi langsung. Periksa kembali rute melalui layanan navigasi aktual sebelum melakukan perjalanan.

Hasil dapat berbeda ketika matriks jarak diambil ulang, misalnya karena pembaruan data jaringan jalan atau perubahan respons layanan OSRM.

## Referensi

- Project OSRM. (n.d.). *OSRM HTTP server API documentation*. https://project-osrm.org/docs/v26.4.0/http
- Tsai, C.-W., Tseng, S.-P., Chiang, M.-C., Yang, C.-S., & Hong, T.-P. (2014). A high-performance genetic algorithm: Using traveling salesman problem as a case. *The Scientific World Journal, 2014*, Article 178621. https://doi.org/10.1155/2014/178621
- Xu, J., Pei, L., & Zhu, R.-Z. (2018). Application of a genetic algorithm with random crossover and dynamic mutation on the travelling salesman problem. *Procedia Computer Science, 131*, 937–945. https://doi.org/10.1016/j.procs.2018.04.230

---

**Penyusun:** Anugerah Bintang Matoto Salempang  
**Mata kuliah:** Teknik Optimalisasi  
**Universitas Kristen Satya Wacana — 2026**
