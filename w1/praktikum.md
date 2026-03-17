# Panduan Operasional Burp Suite: Intersepsi dan Analisis Trafik

Dokumentasi ini merincikan prosedur standar dalam melakukan intersepsi trafik menggunakan Burp Suite untuk keperluan analisis keamanan aplikasi web.

---

## 1. Inisialisasi Proyek (Startup)

Tahap awal melibatkan pembukaan aplikasi serta penentuan basis proyek yang akan dikerjakan.

* Pilih opsi **Temporary project** (pilihan standar pada versi Community). Klik **Next**.

* Gunakan **Use Burp defaults** untuk menerapkan konfigurasi standar sistem. Klik **Start Burp**.

---

## 2. Aktivasi Browser Internal

Burp Suite telah dilengkapi dengan browser bawaan yang terintegrasi secara otomatis dengan pengaturan proxy.

* Navigasikan ke tab **Proxy** > **Intercept**. Pastikan fitur **Intercept is on** dalam kondisi aktif, lalu pilih **Open Browser**.

---

## 3. Proses Intersepsi Request

Prosedur penangkapan data digital yang dikirimkan dari sisi klien (browser) menuju server target.

* Masukkan URL sasaran pada browser internal agar Burp dapat menangkap request tersebut. Gunakan opsi **Forward** untuk meneruskan data secara bertahap, atau **Drop** untuk membatalkan transmisi.

* Jika ingin trafik berjalan tanpa penahanan manual (alir lancar), nonaktifkan fitur dengan mengubah status tombol hingga menjadi **Intercept is off**.

---

## 4. Evaluasi Riwayat HTTP

Meninjau rekaman seluruh aktivitas data yang telah melewati jalur proxy selama sesi berlangsung.

* Pindah ke sub-tab **HTTP history**. Di panel ini, Anda dapat memantau daftar URL, metode permintaan (GET/POST), serta kode status HTTP.

* **Detail:** Pilih salah satu baris riwayat untuk meninjau detail **Request** (data keluar) dan **Response** (umpan balik server) pada panel bagian bawah.

---

## 5. Manipulasi dan Pengiriman Ulang (Repeater)

Memanfaatkan modul Repeater untuk memodifikasi parameter dan mengeksekusi ulang permintaan tanpa perlu mengisi formulir browser berulang kali.

* Klik kanan pada entri di HTTP history, pilih **Send to Repeater**, lalu beralihlah ke tab **Repeater**.

* Di tab Repeater, Anda dapat melakukan modifikasi pada isi Request secara manual. Klik **Send** untuk mengeksekusi permintaan tersebut.

* **Hasil:** Output dari modifikasi akan langsung muncul di panel **Response**. Fitur ini sangat krusial dalam mengidentifikasi kerentanan keamanan seperti SQL Injection atau IDOR.

---

> **Catatan:** Selalu pastikan Anda memiliki otoritas legal dan izin resmi sebelum melakukan pengujian keamanan pada domain atau aplikasi tertentu.
