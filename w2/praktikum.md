# Dokumentasi Hasil Praktikum: Eksplorasi Protokol HTTP via Wireshark

## 3.2 Analisis Transmisi Data HTTP GET/Response

Pada bagian ini, dilakukan observasi terhadap mekanisme komunikasi antara entitas pengirim (*client*) dan penerima (*server*) saat memproses permintaan dokumen web sederhana.

---

### Tahapan Pengamatan

1. **Penyaringan Paket:** Setelah aplikasi Wireshark dijalankan, protokol HTTP diisolasi menggunakan fitur filter untuk mempermudah pemantauan trafik yang relevan.
2. **Eksekusi Penangkapan Data:** Proses pengumpulan paket dimulai bersamaan dengan pengaksesan laman target pada browser:
   `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file1.html`
3. **Penyelesaian Sesi:** Setelah konten HTML berhasil dimuat sepenuhnya di layar browser, aktivitas perekaman trafik dihentikan.

---

### Temuan Analisis

Berdasarkan data yang berhasil direkam, parameter komunikasi yang terdeteksi adalah:

* **Tipe Instruksi:** HTTP GET
* **Status Pengiriman:** 200 OK (Berhasil)
* **Titik Asal (Client):** Alamat IP lokal perangkat mahasiswa.
* **Titik Tujuan (Server):** 128.119.245.12

**Kesimpulan:**
Eksperimen ini memberikan gambaran nyata mengenai siklus permintaan-jawaban di layer aplikasi, di mana server merespons instruksi spesifik dari client dengan mengirimkan data yang diminta secara akurat.

---

## 3.2.1 Mekanisme Efisiensi Bandwidth (Conditional GET)

Eksperimen kedua bertujuan untuk membedah bagaimana browser mengoptimalkan penggunaan jaringan melalui sistem *caching*.

---

### Prosedur Uji Coba

* **Inisialisasi:** Semua data *cache* pada browser dihapus untuk menjamin pengunduhan data baru dari server pada akses pertama.
* **Interaksi Pertama:** Mengakses URL `HTTP-wireshark-file2.html` untuk menyimpan salinan dokumen di memori lokal.
* **Interaksi Kedua (Refresh):** Melakukan pemuatan ulang halaman guna memicu browser mengirimkan pertanyaan "apakah data sudah berubah?".
* **Monitoring:** Lalu lintas data dipantau untuk melihat perbedaan respon server antara akses pertama dan kedua.

---

### Hasil Observasi

1. **Akses Awal:** Server memberikan status **200 OK** dan mengirimkan seluruh isi file karena browser belum memiliki salinan data tersebut.
2. **Akses Berulang:** Browser menyertakan parameter `If-Modified-Since` pada *header* permintaannya. Karena konten tidak mengalami perubahan, server hanya memberikan respon singkat berupa **304 Not Modified**.

**Kesimpulan:**
Fitur **Conditional GET** terbukti sangat efektif untuk mereduksi beban jaringan. Dengan kode **304**, server tidak perlu mengirim ulang data yang identik, sehingga mempercepat performa pemuatan laman web.

---

## 3.3 Segmentasi Data pada Dokumen Berkapasitas Besar

Bagian ini meneliti perilaku protokol saat menangani file yang ukurannya melampaui batas satu paket transmisi standar.

---

### Proses Eksperimen

Dokumen "US Bill of Rights" diakses untuk memicu fragmentasi paket. Karena ukuran file melebihi nilai **Maximum Segment Size (MSS)**, data tidak dikirimkan dalam satu kesatuan utuh, melainkan dipecah menjadi beberapa bagian oleh protokol TCP di bawahnya.

---

### Analisis Segmentasi

* **Fragmentasi TCP:** Terdeteksi adanya keterangan **"TCP segment of a reassembled PDU"**. Ini menunjukkan bahwa satu balasan HTTP sebenarnya terdiri dari beberapa potongan segmen TCP.
* **Integrasi Data:** Wireshark secara otomatis menyatukan kembali (*reassemble*) potongan-potongan tersebut sehingga di level aplikasi tetap terbaca sebagai satu respon **200 OK**.

**Kesimpulan:**
Praktikum ini menunjukkan ketergantungan HTTP pada TCP untuk pengiriman data yang andal, di mana file besar didistribusikan melalui beberapa segmen demi menjaga stabilitas transmisi di jaringan.

---

## 3.4 Penanganan Objek Majemuk dalam Satu Halaman HTML

Eksperimen ini fokus pada bagaimana satu alamat URL bisa memicu banyak proses pengunduhan sekaligus.

---

### Alur Kerja Browser

Saat mengakses halaman yang berisi gambar (objek eksternal), browser melakukan langkah berikut:
1. Mengambil struktur dasar melalui file HTML utama.
2. Memindai konten HTML dan menemukan referensi objek gambar (`pearson.png` dan `8th_edition_cover.jpg`).
3. Mengirimkan instruksi **GET** tambahan secara otomatis untuk tiap-tiap gambar tersebut.

**Kesimpulan:**
Satu halaman web modern sebenarnya terdiri dari kumpulan banyak permintaan terpisah. Browser berperan sebagai pengelola yang menyatukan seluruh komponen tersebut hingga menjadi tampilan yang utuh bagi pengguna.

---

## 3.5 Protokol Autentikasi dan Keamanan Akses

Bagian akhir mengulas proses verifikasi identitas pengguna pada direktori web yang dikunci.

---

### Tahapan Pertukaran Pesan

1. **Tahap Penolakan:** Pada upaya akses pertama, server mengirimkan kode **401 Unauthorized** sebagai instruksi agar browser memunculkan kolom login.
2. **Tahap Verifikasi:** Setelah username dan password diinput, browser mengirimkan ulang permintaan dengan *header* **Authorization**.
3. **Pemberian Izin:** Server melakukan validasi, dan jika sesuai, barulah kode **200 OK** beserta isi halaman dikirimkan.

**Kesimpulan:**
Metode *Basic Authentication* bekerja dengan cara menolak akses awal untuk memvalidasi identitas. Namun, karena data dikirim menggunakan pengkodean *Base64* tanpa enkripsi tambahan, metode ini sangat rentan terhadap penyadapan jika tidak dijalankan di atas protokol HTTPS.
