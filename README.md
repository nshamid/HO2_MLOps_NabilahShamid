# HO2_MLOps_NabilahShamid - Content-Based Recommendation System

## Pendahuluan

Hands-On kedua ini bertujuan untuk membangun sistem rekomendasi sederhana berbasis kemiripan teks. Sistem ini dirancang untuk menyarankan lowongan pekerjaan yang paling relevan berdasarkan preferensi atau kebutuhan pengguna. Pendekatan utama yang digunakan dalam pengerjaan ini adalah **Content-Based Filtering**, di mana rekomendasi diberikan berdasarkan kemiripan antara item yang pernah disukai dengan item lainnya, dihitung dari fitur atau atribut item seperti deskripsi teks, kategori, atau tag. Data yang digunakan dalam Hands-On ini merupakan hasil *preprocessing* dari Hands-On 1.

## Data yang Digunakan

Dataset yang menjadi *input* untuk Hands-On 2 ini adalah *dataset* lowongan pekerjaan yang telah dibersihkan pada Hands-On 1. Dataset ini berasal dari Kaggle, berjudul "[Data Scientist Job Dataset Collections of data scientist and network engineer related job from jobstreet](https://www.kaggle.com/datasets/azraimahadan/data-scientist-jobstreet-scraped)" oleh Azrai Mahadan.

Pada Hands-On 1, saya berupaya melakukan *web scraping* mandiri dari platform Glints menggunakan Selenium. [cite_start]Namun, proses ini menghadapi kendala teknis signifikan (halaman tidak dimuat sempurna atau tidak merender konten lowongan sama sekali), mengindikasikan adanya mekanisme anti-*scraping* yang kuat[cite: 254]. [cite_start]Oleh karena itu, sesuai dengan kelonggaran pada tugas Hands-On 1 dan 2[cite: 155, 256], saya memutuskan untuk menggunakan *dataset* publik yang sudah tersedia sebagai alternatif. Ini memungkinkan saya untuk tetap fokus pada inti pembelajaran *data processing* dan *rekomendasi* meskipun tidak melakukan *scraping* mandiri.

[cite_start]Dataset yang digunakan memiliki kolom-kolom teks utama seperti `job_title` dan `descriptions` yang telah melewati proses *text preprocessing* menyeluruh (lowercasing [cite: 204][cite_start], penghapusan *noise* seperti HTML tags, URLs, hashtag, angka, tanda baca [cite: 209, 212][cite_start], penghapusan *stopwords* [cite: 216][cite_start], dan *stemming* [cite: 219]).

## Proses Text Vectorization

[cite_start]Sebelum teks dapat digunakan dalam sistem rekomendasi atau model *machine learning* lainnya, teks tersebut perlu diubah menjadi representasi numerik (vektor)[cite: 43]. Tahapan ini disebut **Text Vectorization**.

Pada Hands-On ini, saya menggunakan metode **TF-IDF (Term Frequency-Inverse Document Frequency)**. [cite_start]TF-IDF adalah metode *vektorisasi* teks yang digunakan untuk menilai seberapa penting sebuah kata dalam suatu dokumen relatif terhadap seluruh korpus[cite: 73]. [cite_start]Metode ini membantu mengurangi bobot kata-kata umum yang terlalu sering muncul dan tidak memberikan informasi spesifik (seperti "dan" atau "yang")[cite: 74]. [cite_start]Saya memilih TF-IDF karena cocok untuk data berbasis kata kunci dan efektif dalam menyoroti kata-kata yang benar-benar khas dan relevan terhadap konteks dokumen tertentu[cite: 81].

Untuk memperkaya representasi teks, saya menggabungkan konten dari kolom `job_title` dan `descriptions` yang sudah bersih menjadi satu kolom baru bernama `combined_text`. Kolom `combined_text` inilah yang kemudian menjadi input untuk proses *vektorisasi* TF-IDF.

## Proses Similarity Mapping

Setelah teks diubah menjadi vektor numerik, langkah selanjutnya adalah mengukur seberapa mirip dua buah item. [cite_start]Proses ini menggunakan **Similarity Metrics**[cite: 96].

[cite_start]Metrik kemiripan yang digunakan adalah **Cosine Similarity**[cite: 121]. *Cosine similarity* mengukur sudut antara dua vektor, di mana semakin kecil sudutnya, semakin besar kemiripannya. [cite_start]Nilainya berkisar antara -1 (berlawanan arah) hingga 1 (sama persis)[cite: 124]. [cite_start]Metrik ini sangat cocok digunakan dengan vektor TF-IDF dan *embeddings* karena tidak dipengaruhi oleh panjang dokumen[cite: 125]. [cite_start]Matriks kemiripan hasil perhitungan ini akan menjadi dasar penentuan rekomendasi[cite: 17].

## Proses Recommendation Modelling & Hasil

Sistem rekomendasi dibangun dengan fungsi `recommend_jobs` yang menerima judul pekerjaan target dan jumlah rekomendasi (`top_n`) yang diinginkan. Fungsi ini bekerja dengan langkah-langkah berikut:
1.  Mengambil indeks dari pekerjaan target berdasarkan judul yang bersih.
2.  Mendapatkan `job_id` dari pekerjaan target.
3.  Menghitung skor kemiripan antara pekerjaan target dengan semua pekerjaan lain dalam *dataset* menggunakan *cosine similarity matrix* yang telah dihitung.
4.  Mengurutkan pekerjaan berdasarkan skor kemiripan dari tertinggi ke terendah.
5.  Melakukan filtering untuk mengecualikan pekerjaan yang memiliki `job_id` yang sama dengan pekerjaan target. Hal ini untuk memastikan bahwa rekomendasi yang diberikan adalah lowongan pekerjaan yang *berbeda* dan bukan sekadar entri duplikat dari lowongan yang sama di *dataset*.
6.  Mengembalikan `top_n` lowongan pekerjaan yang paling mirip setelah filtering.

**Contoh Hasil Rekomendasi:**

Sebagai contoh, ketika mencari rekomendasi untuk posisi `Data Analyst`:

```

Rekomendasi untuk 'Data Analyst':

1.  Data Analyst — Skor Kemiripan: 0.7029

<!-- end list -->

```
*Catatan: Contoh output di atas adalah hasil dari pengujian fungsionalitas model. Jumlah rekomendasi yang tampil mungkin bervariasi tergantung pada data yang tersedia dan tingkat kemiripan.*

## Refleksi Singkat (Kendala dan Solusi)

Selama pengerjaan Hands-On ini, saya menghadapi beberapa kendala, baik dari modul sebelumnya maupun modul saat ini:

* [cite_start]**Kendala Web Scraping (Hands-On 1)**: Upaya awal untuk melakukan *web scraping* data lowongan pekerjaan dari Glints menggunakan Selenium tidak berhasil karena adanya mekanisme anti-*scraping* yang kuat[cite: 254], yang diduga mendeteksi aktivitas sebagai *bot*.
    * [cite_start]**Solusi**: Sebagai alternatif, saya menggunakan *dataset* publik dari Kaggle[cite: 155]. Ini memungkinkan saya untuk tetap fokus pada inti pembelajaran *data processing* dan *rekomendasi* meskipun tidak melakukan *scraping* mandiri.
* **Kendala Penyesuaian Kolom (Hands-On 2)**: Setelah proses *preprocessing* di Hands-On 1, nama kolom teks utama diubah (`cleaned_job_title` menjadi `job_title`, `cleaned_descriptions` menjadi `descriptions`, dll.). Ini memerlukan penyesuaian pada kode Hands-On 2 agar menggunakan nama kolom yang baru tersebut.
    * **Solusi**: Saya melakukan inspeksi ulang nama kolom pada *DataFrame* yang telah dibersihkan dan memperbarui semua referensi kolom dalam kode *vektorisasi* dan fungsi rekomendasi agar sesuai dengan nama kolom yang baru (`job_title` dan `descriptions`).
* **Kendala Jumlah Rekomendasi yang Minim (Hands-On 2)**: Meskipun `top_n` diatur ke 5, fungsi rekomendasi seringkali hanya mengembalikan satu lowongan pekerjaan.
    * **Analisis**: Setelah *debugging*, teridentifikasi bahwa *dataset* memiliki banyak entri duplikat untuk lowongan pekerjaan yang sama (dengan `job_id` yang identik), dan juga kemungkinan rendahnya kemiripan yang signifikan antar lowongan pekerjaan yang *berbeda secara unik*. Fungsi *cosine similarity* dengan TF-IDF cenderung menghasilkan skor yang sangat rendah jika tidak ada tumpang tindih kata kunci yang kuat antar dokumen yang berbeda.
    * **Solusi**: Saya menerapkan filter `if df.iloc[i]['job_id'] != target_job_id:` dalam fungsi `recommend_jobs`. Filter ini secara efektif mengecualikan semua entri yang berbagi `job_id` yang sama dengan lowongan pekerjaan target. Hasilnya adalah rekomendasi yang lebih akurat dalam konteks "lowongan lain yang berbeda". Meskipun jumlah rekomendasi mungkin tetap sedikit, ini mencerminkan karakteristik data dan bukan kesalahan dalam logika kode. Kendala ini menjadi bagian dari pembelajaran tentang batasan model dan kualitas data.

## Deliverables

Repositori GitHub ini berisi *deliverables* berikut:
* `HO2_MLOps_[FullName].ipynb` (Jupyter Notebook berisi kode Hands-On 2).
* `cleaned_job_data.csv` (Dataset hasil *preprocessing* dari Hands-On 1).
* `processed_job_data.csv` (Dataset yang sudah memiliki kolom `combined_text` sebagai hasil Hands-On 2).
* `processed_job_data.json` (Versi JSON dari `processed_job_data.csv`).
* `tfidf_matrix.csv` (Hasil *vektorisasi* TF-IDF dalam format CSV).
* `tfidf_matrix.npy` (Hasil *vektorisasi* TF-IDF dalam format NumPy binary).
* `cosine_sim_matrix.csv` (Matriks *cosine similarity* dalam format CSV).
* `cosine_sim_matrix.npy` (Matriks *cosine similarity* dalam format NumPy binary).
* `README.md` (Laporan singkat ini).
```
