# HO2_MLOps_NabilahShamid - Content-Based Recommendation System

## Pendahuluan

Hands-On kedua ini bertujuan untuk membangun sistem rekomendasi sederhana berbasis kemiripan teks. Sistem ini dirancang untuk menyarankan lowongan pekerjaan yang paling relevan berdasarkan preferensi atau kebutuhan pengguna. Pendekatan utama yang digunakan dalam pengerjaan ini adalah **Content-Based Filtering**, di mana rekomendasi diberikan berdasarkan kemiripan antara item yang pernah disukai dengan item lainnya, dihitung dari fitur atau atribut item seperti deskripsi teks, kategori, atau tag. Data yang digunakan dalam Hands-On ini merupakan hasil *preprocessing* dari Hands-On 1.

## Data yang Digunakan

Dataset yang menjadi *input* untuk Hands-On 2 ini adalah *dataset* lowongan pekerjaan yang telah dibersihkan pada Hands-On 1. Dataset ini berasal dari Kaggle, berjudul "[Data Scientist Job Dataset Collections of data scientist and network engineer related job from jobstreet](https://www.kaggle.com/datasets/azraimahadan/data-scientist-jobstreet-scraped)" oleh Azrai Mahadan.

Pada Hands-On 1, saya berupaya melakukan *web scraping* mandiri dari platform Glints menggunakan Selenium. Namun, proses ini menghadapi kendala teknis signifikan (halaman tidak dimuat sempurna atau tidak merender konten lowongan sama sekali), mengindikasikan adanya mekanisme anti-*scraping* yang kuat. Oleh karena itu, sesuai dengan kelonggaran pada tugas Hands-On 1 dan 2, saya memutuskan untuk menggunakan *dataset* publik yang sudah tersedia sebagai alternatif. Ini memungkinkan saya untuk tetap fokus pada inti pembelajaran *data processing* dan *rekomendasi* meskipun tidak melakukan *scraping* mandiri. Akan tetapi saya akan tetap mencoba belajar kembali melakukan scrapping data secara mandiri dari paltform sederhana lainnya.

Dataset yang digunakan memiliki kolom-kolom teks utama seperti `job_title` dan `descriptions` yang telah melewati proses *text preprocessing* menyeluruh (lowercasing, penghapusan *noise* seperti HTML tags, URLs, hashtag, angka, tanda baca, penghapusan *stopwords* , dan *stemming*).

## Proses Text Vectorization

Sebelum teks dapat digunakan dalam sistem rekomendasi atau model *machine learning* lainnya, teks tersebut perlu diubah menjadi representasi numerik (vektor). Tahapan yang disebut **Text Vectorization**.

Pada Hands-On ini, saya menggunakan metode **TF-IDF (Term Frequency-Inverse Document Frequency)**. TF-IDF adalah metode *vektorisasi* teks yang digunakan untuk menilai seberapa penting sebuah kata dalam suatu dokumen relatif terhadap seluruh korpus. Metode ini membantu mengurangi bobot kata-kata umum yang terlalu sering muncul dan tidak memberikan informasi spesifik (seperti "dan" atau "yang"). Saya memilih TF-IDF karena cocok untuk data berbasis kata kunci dan efektif dalam menyoroti kata-kata yang benar-benar khas dan relevan terhadap konteks dokumen tertentu.

Untuk memperkaya representasi teks, saya menggabungkan konten dari kolom `job_title` dan `descriptions` yang sudah bersih menjadi satu kolom baru bernama `combined_text`. Kolom `combined_text` inilah yang kemudian menjadi input untuk proses *vektorisasi* TF-IDF.

## Proses Similarity Mapping

Setelah teks diubah menjadi vektor numerik, langkah selanjutnya adalah mengukur seberapa mirip dua buah item. Proses ini menggunakan **Similarity Metrics**.

Metrik kemiripan yang digunakan adalah **Cosine Similarity**. *Cosine similarity* mengukur sudut antara dua vektor, di mana semakin kecil sudutnya, semakin besar kemiripannya. Nilainya berkisar antara -1 (berlawanan arah) hingga 1 (sama persis). Metrik ini sangat cocok digunakan dengan vektor TF-IDF dan *embeddings* karena tidak dipengaruhi oleh panjang dokumen. Matriks kemiripan hasil perhitungan ini akan menjadi dasar penentuan rekomendasi[cite: 17].

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
*Catatan: Contoh output di atas adalah hasil dari pengujian fungsionalitas model. Jumlah rekomendasi yang tampil mungkin bervariasi tergantung pada data dan tingkat kemiripan.*

## Refleksi Singkat (Kendala dan Solusi)

Selama pengerjaan Hands-On ini, saya menghadapi beberapa kendala, baik dari modul sebelumnya maupun modul saat ini:

* **Kendala Web Scraping (Hands-On 1)**: Upaya awal untuk melakukan *web scraping* data lowongan pekerjaan dari Glints menggunakan Selenium tidak berhasil karena adanya mekanisme anti-*scraping* yang kuat, yang diduga mendeteksi aktivitas sebagai *bot*.
    * **Solusi**: Sebagai alternatif, saya menggunakan *dataset* publik dari Kaggle. Ini memungkinkan saya untuk tetap fokus pada inti pembelajaran *data processing* dan *rekomendasi* meskipun tidak melakukan *scraping* mandiri.
* **Kendala Jumlah Rekomendasi yang Minim (Hands-On 2)**: Meskipun `top_n` diatur ke 5, fungsi rekomendasi seringkali hanya mengembalikan satu lowongan pekerjaan.
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
