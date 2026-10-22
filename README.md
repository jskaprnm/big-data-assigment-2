# Big Data Analytics — Assignment II

Prediksi risiko keterlambatan pengiriman pada DataCo Global menggunakan Spark MLlib
dan graph analytics.

**Jesika Purnomo — 2702751025**
Magister Teknik Informatika, BINUS Graduate Program
COMP8035041 — Big Data Analytics

---

## Isi Repositori

```
notebooks/
  Assignment_2_Big_Data_executed.ipynb   notebook yang dijalankan, lengkap dengan output
  Assignment_2_Big_Data_revised.ipynb    versi perbaikan (lihat Catatan Koreksi)
src/
  assignment_2_big_data.py               notebook dalam bentuk skrip Python
output/                                  hasil evaluasi model dan graph (CSV, JSON)
dashboard/                               data siap pakai untuk dashboard Final Project
figures/                                 gambar yang dipakai pada laporan
```

Notebook pada folder `notebooks/` sudah memuat seluruh output eksekusi, jadi hasilnya
bisa dibaca langsung di GitHub tanpa perlu menjalankan ulang. Seluruh angka pada folder
`output/` berasal dari satu kali eksekusi yang sama dengan notebook tersebut, dan
angkanya sama persis dengan yang dilaporkan pada laporan Assignment II.

## Menjalankan Ulang

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USERNAME/REPO/blob/main/notebooks/Assignment_2_Big_Data_executed.ipynb)

1. Buka notebook di Google Colab.
2. Jalankan sel instalasi, lalu pilih **Runtime > Restart session**.
3. Siapkan `kaggle.json` untuk mengunduh dataset DataCo.
4. **Run all**. Waktu eksekusi sekitar 20 sampai 30 menit.

Dataset tidak disertakan pada repositori ini karena ukuran dan ketentuan lisensinya:

- DataCo Smart Supply Chain — https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis
- Amazon Berkeley Objects (ABO) — diunduh otomatis oleh notebook

## Ringkasan Hasil

Model memprediksi `Late_delivery_risk` dari atribut yang sudah tersedia saat order
dibuat. Kolom yang baru terisi setelah pengiriman selesai dikeluarkan untuk mencegah
data leakage. Pembagian data dilakukan secara kronologis 70:10:20, dan ambang keputusan
dipilih hanya memakai validation set.

Hasil pada test set kronologis berisi 36.259 baris, ambang 0,50:

| Model | Framework | Accuracy | Recall kelas 1 | F1 kelas 1 | ROC-AUC |
|---|---|---|---|---|---|
| Logistic Regression | Spark MLlib | 0,7012 | 0,5587 | 0,6716 | 0,7440 |
| Random Forest | Spark MLlib | 0,7182 | 0,5881 | 0,6954 | 0,7596 |
| Gradient-Boosted Trees | Spark MLlib | 0,7153 | 0,5889 | 0,6935 | 0,7714 |
| Decision Tree | Spark MLlib | 0,7158 | 0,5881 | 0,6935 | 0,7662 |
| Baseline lookup Shipping Mode | — | 0,6926 | 0,5384 | 0,6570 | — |

Temuan utamanya, risiko keterlambatan hampir seluruhnya ditentukan oleh Shipping Mode.
Kelompok fitur tersebut menguasai 0,6650 feature importance pada Random Forest,
sedangkan kelompok fitur geografis hanya 0,0005. Model terbaik pun hanya unggul 2,56
poin persentase dari tabel lookup empat baris yang memprediksi semata berdasarkan
Shipping Mode.

Graph analytics memberi kesimpulan yang sejalan. Korelasi antara sentralitas node dan
tingkat keterlambatan berada pada rentang −0,0927 sampai +0,0920, sedangkan modularitas
graph utama hanya 0,0254 dan graph pendukung 0,0000. Keterlambatan tidak terkonsentrasi
pada wilayah maupun kategori produk tertentu.

## Catatan Koreksi

Pada `output/experiment_config.json`, nilai `graph_utama_metrik.modularitas` tercatat
**0,0641**. Nilai tersebut keliru. Variabel penampung modularitas graph utama tertimpa
oleh perhitungan graph pendukung sebelum berkas konfigurasi ditulis.

Nilai yang benar adalah **0,0254**, sesuai output sel deteksi komunitas pada notebook.
Jumlah komunitas sebanyak tiga tidak terpengaruh. Perbaikannya ada pada
`notebooks/Assignment_2_Big_Data_revised.ipynb`, berupa pemisahan nama variabel menjadi
`mod_utama` untuk graph utama dan `mod_unw` untuk graph pendukung.

## Catatan Reproduksibilitas

Batas pembagian data ditentukan memakai `approxQuantile` dengan toleransi galat 0,001,
sehingga batasnya bisa bergeser beberapa baris antareksekusi. Pada eksekusi lain jumlah
baris test set dapat menjadi 36.256 dan sebagian nilai TN bergeser sekitar tiga baris,
tanpa mengubah kesimpulan apa pun. Waktu pelatihan juga berbeda-beda karena bergantung
pada mesin yang dialokasikan Colab.

## Lingkungan

Python 3.13.16 · PySpark 3.5.1 · GraphFrames 0.8.3 · scikit-learn 1.6.1 ·
pandas 2.2.3 · NetworkX 3.7 · random seed 42

PySpark 3.5.1 dipakai, bukan versi 4.x, karena GraphFrames belum tersedia secara stabil
untuk Spark 4 sedangkan satu runtime hanya bisa menjalankan satu versi Spark.
