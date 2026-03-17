# Laporan Praktikum: Analisis Protokol HTTP dengan Wireshark

## 3.2 Interaksi Dasar HTTP GET/Response

Bagian ini menganalisis komunikasi fundamental antara entitas *client* (browser) dan *web server* melalui protokol HTTP.

---

### Prosedur Eksperimen

#### 1. Konfigurasi Filter Wireshark
Langkah pertama adalah menjalankan aplikasi Wireshark dan mengaktifkan filter `http`. Hal ini bertujuan untuk mengisolasi lalu lintas data sehingga hanya paket yang menggunakan protokol HTTP yang ditampilkan pada jendela observasi.

#### 2. Inisiasi Packet Capture dan Akses URL
Setelah filter dikonfigurasi, proses pengambilan paket (*capture*) dimulai. Selanjutnya, akses alamat URL berikut melalui browser:
`http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file1.html`

#### 3. Verifikasi Pemuatan Dokumen
Keberhasilan pengunduhan file HTML ditandai dengan munculnya satu baris teks pada halaman web. Segera setelah konten tampil, proses *capture* dihentikan untuk memulai tahap analisis.

#### 4. Observasi Paket GET dan Response
Melalui jendela Wireshark, dilakukan identifikasi terhadap paket **HTTP GET** yang diinisiasi oleh browser dan paket respons **HTTP OK (200)** yang dikirimkan oleh server sebagai jawaban.

---

### Ringkasan Parameter Analisis

Data yang diperoleh dari hasil *capture* menunjukkan parameter sebagai berikut:

* **Metode Request:** HTTP GET
* **Kode Status Response:** 200 OK
* **Source IP:** Alamat IP perangkat lokal (*Client*).
* **Destination IP:** 128.119.245.12 (*Server gaia.cs.umass.edu*).

---

**Kesimpulan:**
Eksperimen ini membuktikan terjadinya proses interaksi sederhana pada *layer* aplikasi, di mana *client* melakukan permintaan sumber daya dan server memberikan respon balik berupa konten file HTML yang diminta.

---

## 3.2.1 Interaksi HTTP CONDITIONAL GET/Response

Bagian ini mengkaji mekanisme *caching* pada browser melalui metode *Conditional GET* untuk efisiensi penggunaan lebar pita (*bandwidth*).

---

### Prosedur Eksperimen

1. **Eliminasi Cache:** Membersihkan riwayat dan *cache* browser guna memastikan browser meminta salinan data baru dari server.
2. **Aktivasi Capture:** Menjalankan Wireshark untuk merekam seluruh aktivitas pertukaran data.
3. **Akses Awal URL:** Mengunjungi URL:
   `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file2.html`
   Browser akan menampilkan dokumen HTML sederhana yang terdiri dari lima baris teks.
4. **Pemicuan Conditional GET (Refresh):** Melakukan penyegaran halaman (*refresh*) atau mengakses kembali URL yang sama secara cepat untuk mengaktifkan mekanisme pengecekan *cache*.
5. **Terminasi dan Filtrasi:** Menghentikan proses rekaman dan menerapkan filter `http` untuk meninjau urutan paket.

---

### Analisis Hasil Capture

Berdasarkan data yang diperoleh, terdapat perbedaan signifikan antara permintaan pertama dan kedua:

#### 1. Initial HTTP GET
Pada permintaan pertama, server memberikan respon **HTTP/1.1 200 OK**. Server mengirimkan paket data secara utuh karena browser belum memiliki salinan lokal di dalam *cache*.

#### 2. Conditional HTTP GET
Pada permintaan kedua (pascafitur *refresh*), browser menyertakan *header* `If-Modified-Since`. Protokol ini berfungsi menanyakan kepada server apakah ada perubahan pada file sejak waktu akses terakhir.

#### 3. Respon Server: 304 Not Modified
Dikarenakan tidak ada perubahan data pada sisi server, server tidak mengirimkan ulang isi file tersebut. Sebagai gantinya, dikirimkan kode status **HTTP/1.1 304 Not Modified**, yang menginstruksikan browser untuk memuat dokumen dari *cache* lokal.

---

### Kesimpulan
Implementasi **Conditional GET** sangat krusial dalam optimasi sumber daya jaringan. Dengan memanfaatkan status **304 Not Modified**, redundansi transmisi data dapat dihindari, sehingga mempercepat waktu pemuatan halaman dan menghemat kuota data.

---

## 3.3 Penanganan Dokumen Berukuran Besar (Long Documents)

Bagian ini menganalisis kapabilitas protokol HTTP dan TCP dalam mengelola transfer file HTML yang melebihi kapasitas standar satu segmen data (umumnya di atas 1460 byte).

---

### Prosedur Eksperimen

1. **Pembersihan Cache:** Memastikan dokumen diunduh sepenuhnya dari server tanpa ketergantungan pada data lokal.
2. **Inisiasi Capture:** Menjalankan Wireshark tanpa filter awal untuk memantau seluruh proses *handshake* dan segmentasi.
3. **Akses Konten Panjang:** Mengunjungi URL:
   `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file3.html`
   Browser akan memuat dokumen "US Bill of Rights" yang memiliki volume data cukup besar.
4. **Analisis Segmentasi:** Menghentikan *capture* dan menggunakan filter `http` untuk mencari pesan GET, kemudian beralih ke filter `tcp` untuk mengamati pemecahan paket.

---

### Analisis Segmentasi TCP (Multi-packet Response)

File yang diminta memiliki ukuran sekitar 4500 byte. Mengingat ukuran tersebut melampaui **Maximum Segment Size (MSS)**, server melakukan fragmentasi data ke dalam beberapa paket TCP.

#### 1. Mekanisme TCP Segment Reassembly
Pada antarmuka Wireshark, satu respon HTTP dipecah menjadi beberapa segmen transport. Hal ini ditandai dengan keterangan **"TCP segment of a reassembled PDU"** pada kolom informasi paket.

#### 2. Proses Reassembly
Meskipun pada *transport layer* data terbagi, pada *application layer* (HTTP), Wireshark mampu mengonsolidasikan kembali segmen-segmen tersebut sebagai satu kesatuan respon `200 OK`.

---

### Kesimpulan
Praktikum ini memberikan pembuktian bahwa:
* Protokol HTTP sangat bergantung pada protokol TCP untuk manajemen segmentasi data.
* Dokumen berkapasitas besar akan dipecah menjadi segmen kecil (sekitar 1460-1514 byte) demi kelancaran transmisi di infrastruktur jaringan.
* Wireshark memiliki fitur untuk menyatukan kembali (*reassemble*) potongan data sehingga konten aplikasi dapat ditinjau secara utuh.

---

## 3.4 Dokumen HTML dengan Embedded Objects

Bagian ini meninjau bagaimana browser menangani halaman HTML yang mengandung objek tertanam (*embedded objects*), seperti gambar, yang memerlukan permintaan HTTP tambahan.

---

### Prosedur Eksperimen

1. **Reset Cache:** Menghapus data sementara browser agar seluruh objek diunduh secara *fresh*.
2. **Akses URL:** Mengunjungi URL:
   `