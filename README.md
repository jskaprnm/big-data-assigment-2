# Big Data Analytics — Assignment II

Prediksi risiko keterlambatan pengiriman pada DataCo Global menggunakan Spark MLlib
dan graph analytics.

**Jesika Purnomo — 2702751025**
COMP8035041 — Big Data Analytics

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

## Environment
Python 3.13.16 · PySpark 3.5.1 · GraphFrames 0.8.3 · scikit-learn 1.6.1 ·
pandas 2.2.3 · NetworkX 3.7 · random seed 42

PySpark 3.5.1 dipakai, bukan versi 4.x, karena GraphFrames belum tersedia secara stabil
untuk Spark 4 sedangkan satu runtime hanya bisa menjalankan satu versi Spark.
