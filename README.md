# Segmentasi Polip pada Citra Kolonoskopi dengan U-Net

<!--
  TEMPLATE LAPORAN — Tugas P10 Kecerdasan Buatan (BM25-41204), Teknik Biomedis ITERA.
  Petunjuk lengkap: https://mctm.web.id/ai-bm/tugas

  Cara memakai:
  - Ganti semua teks yang diawali "✍️" dengan tulisan kelompokmu, lalu hapus tanda ✍️-nya.
  - Blok komentar seperti ini adalah petunjuk; tidak tampil di GitHub. Boleh dihapus.
  - Gambar diambil dari folder hasil/ yang dibuat otomatis oleh notebook.
  - Jangan hapus judul-judul bagian (##).
-->

**Tugas P10 — Kecerdasan Buatan (BM25-41204), Teknik Biomedis ITERA**

## Anggota kelompok

| Nama | NIM | Username GitHub | Kontribusi utama |
|---|---|---|---|
| Rizka Putri Handayani | 123430017 | rizkaputrihandayanj-ui |  menjalankan eksperimen,menjalankan di collab, menulis bagian 2, menulis bagian 3, menulis bagian 4 |
| Aliyah Wardani | 12343003 | aliyahwardani237 | menulis bagian 5, menulis bagian 6, menulis bagian 7, menulis bagian 8 |

## 1. Pertanyaan

Augmentasi apa yang membantu U-Net menemukan polip pada citra kolonoskopi, dan augmentasi apa yang tidak? Kenapa?

## 2. Data & pengaturan

<!-- Isi dari Bagian 2 notebook. Pastikan CEPAT = False untuk hasil akhir. -->

- Dataset: Kvasir-SEG (1000 citra kolonoskopi + mask polip), diperkecil ke 128 × 128 piksel.
- Data uji: 200 citra (tetap). Data latih: 800 citra per ulangan.
- Model: U-Net mini (±0,5 juta parameter), 30 epoch, 3 ulangan per skenario.
- Dijalankan di: Google Colab T4.

![Contoh data](hasil/contoh_data.png)

## 3. Skenario augmentasi & hipotesis

<!-- Tulis bagian ini SEBELUM menjalankan eksperimen. Commit dulu, baru jalankan.
     Alasan klinis = kenapa variasi ini mungkin (atau tidak mungkin) terjadi pada kolonoskopi sungguhan. -->

| Skenario | Augmentasi (kode) | Alasan klinis | Hipotesis: Dice naik/turun/sama? |
|---|---|---|---|
| Tanpa augmentasi | — | Pembanding | — |
| Flip horizontal | `v2.RandomHorizontalFlip(p=0.5)` | Polip dapat terlihat dari berbagai orientasi akibat perubahan posisi dan sudut pengambilan gambar kamera kolonoskopi. | Naik |
| Rotasi | `v2.RandomRotation(degrees=15)` | Posisi dan orientasi citra dapat berubah selama pemeriksaan akibat pergerakan kamera kolonoskopi. | Naik |
| Kecerahan & kontras | `v2.ColorJitter(brightness=0.2, contrast=0.2)` | Intensitas pencahayaan dan kontras citra dapat berbeda akibat kondisi pencahayaan serta jarak kamera terhadap jaringan.| Naik |
| Kombinasi | `Flip horizontal + Rotasi + Kecerahan & kontras` | Menggabungkan variasi orientasi dan pencahayaan untuk meningkatkan kemampuan model dalam mengenali polip pada kondisi citra yang beragam.| Naik |

https://colab.research.google.com/drive/1SA3Hcq02cK_OzUxZQH8k8mBtKLuKPYcD#scrollTo=6389f853&fullscreenOutput=true
<!-- Ganti nama file sesuai skenario kalian (lihat isi folder hasil/). -->

## 4. Hasil
| Skenario | Dice (rata-rata ± sb) | IoU (rata-rata ± sb) | Dice tiap ulangan |
|---|---|---|---|
| Tanpa augmentasi | 0.535 ± 0.078 | 0.429 ± 0.063 | 0.552, 0.621, 0.431 |
| Flip horizontal | 0.574 ± 0.045 | 0.463 ± 0.036 | 0.593, 0.617, 0.511 |
| Rotasi | 0.562 ± 0.059 | 0.455 ± 0.053 | 0.485, 0.572, 0.628 |
| Kecerahan & kontras | 0.438 ± 0.102 | 0.345 ± 0.088 | 0.399, 0.577, 0.337 |
| Kombinasi | 0.543 ± 0.041 | 0.432 ± 0.036 | 0.562, 0.486, 0.582 | 

(https://colab.research.google.com/drive/1SA3Hcq02cK_OzUxZQH8k8mBtKLuKPYcD#scrollTo=14dd4f45&fullscreenOutput=true)

## 5. Analisis peta kesalahan

Berdasarkan visualisasi peta kesalahan segmentasi, terlihat bahwa performa model berbeda pada setiap jenis augmentasi. Pengujian tanpa augmentasi menghasilkan rata-rata Dice sebesar 0,478, sedangkan flip horizontal memperoleh nilai tertinggi sebesar 0,705. Augmentasi rotasi menghasilkan rata-rata Dice sebesar 0,558, kombinasi augmentasi sebesar 0,510, dan perubahan kecerahan serta kontras memperoleh nilai terendah sebesar 0,095. Hasil ini menunjukkan bahwa flip horizontal memberikan kesesuaian segmentasi terbaik dibandingkan teknik augmentasi lainnya pada empat citra yang diuji.

Perbedaan nilai Dice menunjukkan bahwa model masih memiliki tingkat kesalahan segmentasi yang bervariasi, terutama dalam mengenali area dan batas objek sesuai anotasi dokter. Pada beberapa citra, hasil prediksi memiliki kesesuaian yang tinggi, sedangkan citra lainnya menunjukkan perbedaan area yang cukup besar. Perubahan kecerahan dan kontras diduga memengaruhi kemampuan model dalam mengenali karakteristik objek, sehingga menghasilkan segmentasi yang kurang optimal. Secara keseluruhan, flip horizontal menjadi teknik augmentasi dengan hasil terbaik pada pengujian ini, tetapi pengujian menggunakan lebih banyak data tetap diperlukan untuk memastikan konsistensi performa model.

https://colab.research.google.com/drive/1SA3Hcq02cK_OzUxZQH8k8mBtKLuKPYcD#scrollTo=0b460587&fullscreenOutput=true

## 6. Pembahasan & kesimpulan

Berdasarkan hasil eksperimen, hipotesis bahwa seluruh jenis augmentasi dapat meningkatkan nilai Dice tidak sepenuhnya terbukti. Augmentasi flip horizontal menghasilkan nilai Dice tertinggi sebesar 0,574 ± 0,045, diikuti rotasi sebesar 0,562 ± 0,059. Kedua augmentasi tersebut menunjukkan peningkatan dibandingkan kondisi tanpa augmentasi yang memperoleh Dice sebesar 0,535 ± 0,078. Hal ini menunjukkan bahwa perubahan orientasi citra dapat membantu U-Net mengenali polip dengan lebih baik karena posisi dan sudut pandang polip pada pemeriksaan kolonoskopi dapat bervariasi. Sementara itu, kombinasi augmentasi menghasilkan Dice sebesar 0,543 ± 0,041, yang hanya sedikit lebih tinggi dibandingkan tanpa augmentasi. Augmentasi kecerahan dan kontras justru menghasilkan Dice terendah sebesar 0,438 ± 0,102. Penurunan ini kemungkinan terjadi karena perubahan intensitas warna dan kontras memengaruhi karakteristik visual polip, sehingga model kesulitan membedakan polip dengan jaringan di sekitarnya. Selain itu, perbedaan hasil antarulangan menunjukkan bahwa performa model masih dipengaruhi oleh variasi proses pelatihan.

Dengan demikian, augmentasi yang paling membantu dalam eksperimen ini yaitu flip horizontal, sedangkan augmentasi kecerahan dan kontras belum memberikan hasil yang baik. Rotasi juga menunjukkan peningkatan performa, meskipun hasilnya sedikit lebih rendah dibandingkan flip horizontal. Penggunaan kombinasi augmentasi belum memberikan peningkatan yang berarti karena kemungkinan variasi yang diberikan secara bersamaan justru membuat karakteristik citra menjadi lebih sulit dikenali. Berdasarkan hasil tersebut, augmentasi geometris lebih sesuai untuk membantu segmentasi polip pada dataset yang diuji dibandingkan augmentasi perubahan intensitas pencahayaan. Namun, hasil ini masih terbatas pada dataset, ukuran citra, dan konfigurasi model yang digunakan.


## 7. Pembagian Kerja

| Nama | NIM | Username GitHub | Kontribusi utama |
|---|---|---|---|
| Rizka Putri Handayani | 123430017 | rizkaputrihandayanj-ui | Mengerjakan bagian 1–4, meliputi pertanyaan penelitian, pengaturan dataset dan model, penyusunan skenario augmentasi dan hipotesis, menjalankan eksperimen, serta menyusun hasil pengujian. |
| Aliyah Wardani | 123430031 | aliyahwardani237 | Mengerjakan bagian 5–8, meliputi analisis peta kesalahan segmentasi, pembahasan dan kesimpulan, penyusunan pembagian kerja, serta dokumentasi penggunaan AI. |


## 8. Penggunaan AI

ChatGPT digunakan sebagai alat bantu dalam menyusun dan merapikan penulisan laporan, khususnya pada bagian analisis peta kesalahan segmentasi serta pembahasan dan kesimpulan hasil eksperimen. AI juga membantu menjelaskan perbandingan nilai Dice pada setiap skenario augmentasi agar hasil pengujian lebih mudah dipahami. Seluruh data hasil eksperimen, pemilihan skenario augmentasi, dan evaluasi performa model tetap mengacu pada hasil pengujian yang diperoleh melalui Google Colab.

## Cara menjalankan ulang

```bash
uv sync
uv run jupyter lab segmentasi_polip.ipynb
```

Atau buka notebook di Google Colab (Runtime → Change runtime type → T4 GPU).

## Referensi

- Jha, D., Smedsrud, P. H., Riegler, M. A., Halvorsen, P., de Lange, T., Johansen, D., & Johansen, H. D. (2020). Kvasir-SEG: A Segmented Polyp Dataset. MultiMedia Modeling (MMM 2020), 451–462. https://doi.org/10.1007/978-3-030-37734-2_37. 
datasets.simula.no+1
- Ronneberger, O., Fischer, P., & Brox, T. (2015). U-Net: Convolutional Networks for Biomedical Image Segmentation. Medical Image Computing and Computer-Assisted Intervention (MICCAI 2015), 234–241. https://doi.org/10.1007/978-3-319-24574-4_28.
- Adjei, P. E., Lonseko, Z. M., Du, W., Zhang, H., & Rao, N. (2022). Examining the effect of synthetic data augmentation in polyp detection and segmentation. International Journal of Computer Assisted Radiology and Surgery, 17(7), 1289–1302. https://doi.org/10.1007/s11548-022-02651-x. 
PubMed
- Liu, X., Isa, N. A. M., Chen, C., & Lv, F. (2025). Colorectal Polyp Segmentation Based on Deep Learning Methods: A Systematic Review. Journal of Imaging, 11(9), 293. https://doi.org/10.3390/jimaging11090293. 
PubMed+1
- Mei, J., Zhou, T., Huang, K., Zhang, Y., Zhou, Y., Wu, Y., et al. (2025). A survey on deep learning for polyp segmentation: techniques, challenges and future trends. Visual Intelligence, 3, Article 1. https://doi.org/10.1007/s44267-024-00071-w. 
DOI
- Bernal, J., Sánchez, J., Fernández-Esparrach, G., Gil, D., Rodríguez, C., & Vilariño, F. (2015). WM-DOVA maps for accurate polyp highlighting in colonoscopy: Validation vs. saliency maps from physicians. Computerized Medical Imaging and Graphics, 43, 99–111. https://doi.org/10.1016/j.compmedimag.2015.02.007.
