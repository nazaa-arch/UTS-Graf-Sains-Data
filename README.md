# Ujian Tengah Semester (UTS) - Graf Sains Data
Program Studi S1 Sains Data - Fakultas Matematika dan Ilmu Pengetahuan Alam  
Universitas Negeri Surabaya  
Tahun Akademik: Gasal 2026/2027  

---

### Identitas Mahasiswa
* Nama: Muhammad Naza Firmansyah
* NIM: 24031554164
* Mata Kuliah: Graf Sains Data (3 SKS)
* Dosen Pengampu: TIM Dosen Graf Sains Data UNESA

---

## 1. Ringkasan Tugas dan Hasil Eksperimen

Repositori ini memuat implementasi komprehensif analisis graf, Vector Space Modelling (VSM), dan sistem rekomendasi berbasis Collaborative Filtering (KNN) menggunakan basis data graf Neo4j serta pustaka Python (networkx, scipy, scikit-learn, pandas).

### Tugas Pendahuluan
* Persiapan Lingkungan: Menyiapkan virtual environment Python 3.10 via uv / venv.
* Penggunaan Driver Neo4J: Menguji koneksi, menambahkan simpul dan relasi pertemanan Arthur, Guinevere, Lancelot, dan Merlin.
* Jawaban Checkpoint:
  1. Database yang digunakan: neo4j (database graf default).
  2. Akses database lain dalam satu instance: Klausul Cypher USE <nama_db>, parameter sesi driver driver.session(database="<nama_db>"), serta perintah DDL Cypher di database system (SHOW DATABASES;, CREATE DATABASE, STOP/START DATABASE, DROP DATABASE).
  3. Jenis Graf Pertemanan: Homogeneous Directed Graph (Graf Berarah Homogen) dengan topologi Ego-Network (Star Graph) berpusat pada simpul Arthur.
* Impor Data RetailRocket: Pembuatan mekanisme batch insert ke Neo4j memanfaatkan relasi [:VIEWED], [:ADDED_TO_CART], dan [:BOUGHT].

---

### Soal 2.1 - Wrangling dan EDA
1. Penentuan Dua Periode Waktu Unik:
   * Mengikuti metode holdout time-series split standar pada rentang penuh dataset RetailRocket (3 Mei 2015 hingga 18 September 2015), titik potong (cut-off date) ditentukan secara unik berdasarkan digit belakang NIM 164 (tanggal 16 Juli 2015):
     - Fold Training (P1): 3 Mei 2015 s.d. 16 Juli 2015
       * UNIX Timestamp Mulai: 1430622004384 (3 Mei 2015, 03:00:04 UTC)
       * UNIX Timestamp Selesai: 1437091183647 (16 Juli 2015, 23:59:43 UTC)
     - Fold Testing (P2): 17 Juli 2015 s.d. 18 September 2015
       * UNIX Timestamp Mulai: 1437091199857 (17 Juli 2015, 00:00:00 UTC)
       * UNIX Timestamp Selesai: 1442545187788 (18 September 2015, 02:59:47 UTC)
   * Rasionalisasi: Posisi tanggal 16 Juli unik di antara rekan sekelas (misal: Umi di 14 Juli dan Nadine di 21 Juli).
   * Subgraph Representatif: Sesuai instruksi soal No. 3 ("subgraph yang dibuat"), dari partisi waktu P1 (1.556.454 baris) dan P2 (1.199.647 baris) yang masif, diekstraksi subgraph interaksi di sekitar cut-off 16 Juli (16-17 Juli untuk pelatihan P1 dan 18-19 Juli untuk pengujian P2) dan disimpan ke data/events_filtered.csv. Hal ini menjaga efisiensi komputasi VSM, mencegah Out-of-Memory, dan menghasilkan visualisasi NetworkX yang jelas.
2. Dataset Filter: Data subgraph hasil pemfilteran disimpan pada berkas data/events_filtered.csv (41.818 baris pada P1 dan 33.293 baris pada P2).
3. Komposisi Subgraph di Neo4j:
   * Simpul :Visitor: 25.321 nodes
   * Simpul :Item: 22.788 nodes
   * Total Relasi: 41.818 edges (VIEWED: 40.376, ADDED_TO_CART: 1.064, BOUGHT: 378).
4. VSM Simpul User (Visitor-Item Matrix):
   * Dimensi: 25.321 x 22.788
   * Sparsity: 99,99402%
   * Visualisasi proyeksi graf User-User menggunakan NetworkX.
5. VSM Simpul Item (Item-Visitor Matrix):
   * Dimensi: 22.788 x 25.321
   * Sparsity: 99,99402%
   * Visualisasi proyeksi graf Item-Item menggunakan NetworkX.
6. Analisis Komparasi Proyeksi:
   * Proyeksi Item-Item terbukti lebih unggul (efektif dan efisien) karena jumlah simpul lebih sedikit, stabilitas temporal profil produk konsisten, dan komputasi kemiripan dapat dilakukan secara precomputed/offline.
7. Perhitungan Similarity pada Graf Terproyeksi:
   * Menggunakan Cosine Similarity pada Python.
   * Item Acuan: Produk 119736 (produk populer dengan 578 co-occurrences).
   * Nilai kemiripan dituliskan kembali ke Neo4j sebagai relasi (Item)-[:SIMILAR {score}]->(Item).
8. Kondisi Graf Sebelum dan Sesudah:
   * Sebelum: Graf bipartit interaksi Pengunjung ke Item.
   * Sesudah: Graf memiliki relasi horizontal kemiripan langsung antar Item.

---

### Soal 2.2 - Analytics
1. Pemilihan Pengguna Acak (Reproducible):
   * Menggunakan seed = 135 pada pool pengguna aktif P1 dan P2.
   * Pengguna Terpilih: Visitor 988009
   * Riwayat Pelatihan P1: [308353, 45059, 288359]
   * Data Uji Ground Truth P2: [44894, 375011, 181102]
   * Algoritma Rekomendasi: Item-based Collaborative Filtering KNN (k = 5).
   * Tiga Item yang Diiklankan:
     1. Item 226940 (Skor: 0.7071)
     2. Item 197863 (Skor: 0.7071)
     3. Item 181102 (Skor: 0.7071) - tercatat dikunjungi pengguna pada periode pengujian P2.
2. Evaluasi Metrik:
   * Hit@5: 1 (Item 181102 berhasil masuk pada peringkat ke-3 rekomendasi Top-5).
   * NDCG@5: 0.5000
     - DCG@5 = (2^1 - 1) / log2(3 + 1) = 1 / log2(4) = 0.5000
     - IDCG@5 = (2^1 - 1) / log2(1 + 1) = 1 / log2(2) = 1.0000
     - NDCG@5 = DCG@5 / IDCG@5 = 0.5000

---

## 2. Struktur Berkas

```
UTS/
|-- 2601_UTS-GSD.pdf                                   # Naskah soal UTS resmi
|-- UTS_Graf_Sains_Data_Muhammad_Naza_Firmansyah.ipynb # Notebook utama pengerjaan UTS
|-- requirements.txt                                   # Daftar dependensi pustaka
|-- README.md                                          # Dokumentasi laporan UTS
`-- data/
    `-- events_filtered.csv                            # Dataset interaksi hasil filter (P1 dan P2)
```

---

## 3. Petunjuk Menjalankan

1. Clone repositori:
   ```bash
   git clone https://github.com/nazaa-arch/UTS-Graf-Sains-Data.git
   cd UTS-Graf-Sains-Data
   ```
2. Buat virtual environment dan install dependensi:
   ```bash
   python -m venv myvenv
   source myvenv/bin/activate  # atau myvenv\Scripts\activate pada Windows
   pip install -r requirements.txt
   ```
3. Pastikan service Neo4j aktif pada port 7687 dengan user `neo4j` dan password `password`.
4. Buka notebook pengerjaan:
   ```bash
   jupyter notebook UTS_Graf_Sains_Data_Muhammad_Naza_Firmansyah.ipynb
   ```
