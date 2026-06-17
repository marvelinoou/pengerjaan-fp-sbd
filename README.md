# Skrip Presentasi FP Sistem Basis Data
## Sistem Pengelolaan Akademik Mahasiswa dan Nilai

Durasi target: 15-25 menit. Setiap bagian punya dua lapis: SKRIP (yang diomongin) dan PEMAHAMAN MATERI (konsep di baliknya, buat jaga-jaga kalau dosen nanya di luar skrip).

---

# BAGIAN 1 — PPT

## Slide 1: Judul

**SKRIP:**
"Selamat pagi/siang, perkenalkan kami dari kelompok FP-SBD-085-076-079. Saya Marvelino Davas, bersama rekan saya Muhammad Hugo Rayandra dan Raffa Al Azmi. Hari ini kami akan mempresentasikan Final Project mata kuliah Sistem Basis Data, yaitu Sistem Pengelolaan Akademik Mahasiswa dan Nilai."

**PEMAHAMAN MATERI:**
Tidak ada konsep teknis di slide ini, tapi pastikan nama dan NRP yang disebut sudah benar dan urutannya konsisten dengan yang tertulis di slide — kelihatan sepele tapi sering jadi hal pertama yang dikoreksi dosen kalau salah sebut nama rekan sendiri.

---

## Slide 2: Nama Sistem

**SKRIP:**
"Sistem ini kami beri nama Sistem Monitoring Nilai Mahasiswa, yang secara teknis di laporan kami sebut Sistem Pengelolaan Akademik Mahasiswa dan Nilai. Fokus utama sistem ini adalah membantu dosen wali memantau perkembangan akademik mahasiswa bimbingannya."

**PEMAHAMAN MATERI:**
Penting ditegaskan sejak awal: target pengguna sistem ini adalah DOSEN WALI, bukan mahasiswa. Ini krusial karena seluruh keputusan desain fitur tambahan (mengulang, alumni) dibuat dari sudut pandang kebutuhan dosen wali memonitor, bukan mahasiswa mengakses data sendiri. Kalau dosen tanya "kenapa nggak ada login mahasiswa?", jawabannya karena scope sistem ini memang backend monitoring untuk dosen wali, bukan portal akademik mahasiswa.

---

## Slide 3: Identifikasi Masalah

**SKRIP:**
"Ada tiga masalah utama yang mendasari pembuatan sistem ini. Pertama, dengan jumlah mahasiswa yang banyak, dosen wali kesulitan memantau perkembangan setiap mahasiswa secara individual dan berkelanjutan. Kedua, dosen wali kesulitan mengidentifikasi mahasiswa yang membutuhkan perhatian khusus, misalnya yang mengulang mata kuliah atau mengalami penurunan performa akademik. Ketiga, proses pencatatan dan pelaporan data akademik masih manual, sehingga tidak efisien dan rentan terhadap inkonsistensi data."

**PEMAHAMAN MATERI:**
Tiga masalah ini bukan cuma pemanasan presentasi — masing-masing punya jawaban konkret di sistem yang dibangun. Masalah pertama dijawab oleh keberadaan backend API yang bisa diakses kapan saja tanpa proses manual. Masalah kedua dijawab langsung oleh dua fitur tambahan (view mengulang dan tabel alumni) yang akan dibahas nanti. Masalah ketiga dijawab oleh trigger otomatis yang menghilangkan kebutuhan pencatatan manual saat status mahasiswa berubah. Kalau dosen tanya "mana buktinya sistem ini menjawab masalah yang disebutkan?", lo bisa langsung tarik balik ke tiga poin ini.

---

## Slide 4: Tujuan

**SKRIP:**
"Berdasarkan masalah tersebut, kami menetapkan empat tujuan. Pertama, merancang skema basis data relasional yang komprehensif untuk mendukung kebutuhan pengelolaan dan monitoring data akademik. Kedua, mengimplementasikan struktur penyimpanan data yang efisien melalui normalisasi hingga bentuk normal ketiga. Ketiga, membangun fitur monitoring akademik yang dapat diakses melalui backend, tanpa antarmuka pengguna. Keempat, menyediakan informasi yang mendukung pengambilan keputusan oleh dosen wali, seperti ranking IPK, daftar mata kuliah yang diulang, dan status mahasiswa lulus atau pindah."

**PEMAHAMAN MATERI:**
Poin ketiga ini penting ditegaskan secara eksplisit: TANPA FRONTEND. Ini bukan kekurangan, tapi keputusan desain yang sesuai scope FP. Kalau dosen tanya "kenapa nggak dibuat web-nya juga?", jawabannya: scope FP ini fokus pada perancangan basis data dan logikanya, bukan UI/UX, dan backend REST API sudah cukup untuk membuktikan seluruh logika data berjalan dengan benar — pembuktian dilakukan lewat curl/Postman, bukan browser.

---

## Slide 5: Desain ERD

**SKRIP:**
"Ini adalah hasil akhir desain basis data kami, yang terdiri dari sembilan tabel. Enam tabel inti — Mahasiswa, Dosen, Mata Kuliah, Semester, KRS, dan Nilai — adalah hasil dari proses normalisasi data mentah hingga bentuk normal ketiga. Tabel Dosen Mengajar adalah tabel penghubung untuk relasi banyak-ke-banyak antara dosen, mata kuliah, dan semester. Dua tabel lainnya, Mahasiswa Alumni dan Trash, adalah tabel tambahan yang tidak berasal dari normalisasi data mentah, melainkan dibuat khusus untuk memenuhi kebutuhan tambahan dari dosen wali, yang akan kami jelaskan lebih detail di slide berikutnya."

**PEMAHAMAN MATERI:**
Ini saat paling tepat untuk menjelaskan relasi PK-FK secara singkat kalau dosen minta detail. Hal yang harus dihafal: Mahasiswa terhubung ke KRS (satu-ke-banyak), Mata Kuliah dan Semester juga terhubung ke KRS (masing-masing satu-ke-banyak), KRS terhubung ke Nilai (satu-ke-satu, karena satu KRS punya tepat satu nilai), Dosen terhubung ke Dosen Mengajar, dan Mahasiswa terhubung ke Mahasiswa Alumni (satu-ke-banyak, meski secara praktik biasanya satu mahasiswa cuma punya satu entri alumni). Trash sengaja tidak punya relasi FK ke tabel apa pun karena dia adalah arsip generik lintas tabel — desain ini sengaja dibuat fleksibel, bukan terikat ke satu tabel tertentu.

Kalau ditanya soal kasus mengulang: tunjuk ke tabel KRS, jelaskan bahwa PK aslinya adalah gabungan tiga kolom (nrp, kode_mk, id_semester), sehingga satu mahasiswa bisa punya lebih dari satu entri KRS dengan kode_mk yang sama, selama id_semester-nya berbeda.

---

## Slide 6: Pemrosesan Data

**SKRIP:**
"Dari sisi arsitektur, sistem ini terbagi dalam dua lapisan. Lapisan aplikasi menggunakan Node.js dan Express.js untuk menangani protokol HTTP, routing API, dan enkapsulasi data dalam format JSON. Lapisan basis data menggunakan MySQL sebagai mesin komputasi relasional utama. Untuk kalkulasi yang kompleks, seperti perhitungan IPK dan akumulasi SKS, kami serahkan seratus persen ke fungsi bawaan MySQL, yaitu SUM dan ROUND, yang dipanggil pada endpoint nilai per IPK."

**PEMAHAMAN MATERI:**
Poin penting di sini: kenapa kalkulasi IPK dilakukan di level database (SQL), bukan di level aplikasi (JavaScript)? Jawaban yang baik: MySQL dirancang khusus untuk operasi agregasi data dalam jumlah besar secara efisien — JOIN antar tabel dan SUM/GROUP BY itu pekerjaan native database engine. Kalau dikerjakan di Node.js, backend harus menarik semua baris data mentah dulu ke memori aplikasi, baru menghitung manual di JavaScript — itu boros memori dan lebih lambat dibanding membiarkan MySQL yang sudah dioptimasi untuk itu mengerjakannya langsung.

Rumus IPK yang dipakai: SUM(nilai_angka × sks) dibagi SUM(sks), dibulatkan dua desimal. Ini weighted average berdasarkan bobot SKS — bukan rata-rata biasa, karena mata kuliah dengan SKS lebih besar harus punya pengaruh lebih besar terhadap IPK.

---

## Slide 7: Abstraksi Data Menggunakan View

**SKRIP:**
"Salah satu prinsip yang kami terapkan adalah abstraksi data menggunakan view. Tujuannya untuk memisahkan skema logis basis data yang rumit dari visualisasi data di tingkat aplikasi, sehingga backend developer tidak perlu memahami detail kerumitan struktur tabel penyusun data historis. Studi kasus konkretnya adalah pelacakan mahasiswa yang mengulang mata kuliah, yang membutuhkan operasi relasi multi-tabel lewat JOIN, pengelompokan data lewat GROUP BY, dan filter kondisi berbasis hasil fungsi agregasi lewat HAVING."

**PEMAHAMAN MATERI:**
Ini bagian paling sering ditanya detail teknisnya, jadi harus benar-benar paham, bukan cuma hafal. View yang dipakai adalah `view_mahasiswa_mengulang`:

```sql
CREATE VIEW view_mahasiswa_mengulang AS
SELECT m.nrp, m.nama, mk.kode_mk, mk.nama_mk, COUNT(*) AS jumlah_ambil
FROM krs k
JOIN mahasiswa m ON k.nrp = m.nrp
JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
GROUP BY m.nrp, m.nama, mk.kode_mk, mk.nama_mk
HAVING COUNT(*) > 1;
```

Cara baca query ini kalau diminta jelaskan baris per baris: JOIN menggabungkan data KRS dengan nama mahasiswa dari tabel Mahasiswa dan nama mata kuliah dari tabel Mata Kuliah (karena tabel KRS sendiri cuma punya kode/nrp, bukan nama). GROUP BY mengelompokkan semua baris KRS yang punya kombinasi nrp dan kode_mk yang sama jadi satu grup. COUNT(*) menghitung berapa baris ada dalam satu grup. HAVING COUNT(*) > 1 menyaring, hanya menampilkan grup yang punya lebih dari satu baris — artinya mahasiswa itu mengambil mata kuliah yang sama lebih dari sekali.

Kalau ditanya beda WHERE dan HAVING: WHERE menyaring baris sebelum dikelompokkan, HAVING menyaring grup setelah dikelompokkan — dipakai HAVING di sini karena syarat penyaringannya (COUNT > 1) baru bisa dihitung setelah pengelompokan terjadi.

Keuntungan pakai view: backend cukup memanggil `SELECT * FROM view_mahasiswa_mengulang`, tidak perlu menulis ulang JOIN yang panjang di kode JavaScript setiap kali endpoint itu dipanggil.

---

## Slide 8: Otomatisasi Terintegrasi via Database Trigger (BEFORE DELETE)

**SKRIP:**
"Untuk otomatisasi dan audit, kami menggunakan database trigger, yaitu blok kode SQL prosedural yang dieksekusi otomatis oleh database engine sebagai respons terhadap peristiwa INSERT, UPDATE, atau DELETE. Mekanisme pertama adalah BEFORE DELETE, yang kami implementasikan sebagai empat trigger terpisah, masing-masing spesifik untuk satu tabel: before_delete_mahasiswa, before_delete_dosen, before_delete_mata_kuliah, dan before_delete_krs. Setiap trigger berjalan sebelum baris data dihapus dari tabel master yang bersangkutan, memanfaatkan fungsi JSON_OBJECT untuk membungkus seluruh data lama ke dalam format dokumen semistruktur, lalu menyimpannya ke tabel trash."

**PEMAHAMAN MATERI:**
Kalau dosen tanya "kenapa empat trigger terpisah, bukan satu saja?" — jawabannya: trigger di MySQL itu sifatnya per tabel, tidak bisa satu trigger menangani DELETE dari banyak tabel berbeda sekaligus. Jadi keempatnya memang harus didefinisikan terpisah, meskipun logikanya serupa (sama-sama menyalin OLD data ke trash).

Contoh isi salah satu trigger:
```sql
CREATE TRIGGER before_delete_krs
BEFORE DELETE ON krs FOR EACH ROW
BEGIN
    INSERT INTO trash (nama_tabel, data_json, deleted_by)
    VALUES ('krs', JSON_OBJECT(
        'id_krs', OLD.id_krs, 'nrp', OLD.nrp,
        'kode_mk', OLD.kode_mk, 'id_semester', OLD.id_semester,
        'status', OLD.status
    ), 'system');
END;
```

`OLD` di sini merujuk ke nilai kolom SEBELUM baris itu dihapus — ini keyword khusus yang hanya bisa dipakai di dalam trigger. `BEFORE DELETE` artinya kode ini dijalankan sebelum penghapusan benar-benar terjadi, sehingga datanya masih bisa diselamatkan ke trash dulu sebelum hilang permanen dari tabel asalnya.

---

## Slide 9: AFTER UPDATE — Sistem Pipeline Arsip Otomatis

**SKRIP:**
"Mekanisme kedua adalah AFTER UPDATE, yang berjalan setelah kolom status pada tabel mahasiswa diperbarui. Trigger ini memicu penyalinan data riwayat hidup secara otomatis ke tabel mahasiswa_alumni, saat mahasiswa dinyatakan lulus, keluar, atau pindah. Penting untuk ditekankan: data pada tabel mahasiswa asli tidak dihapus. Tabel mahasiswa_alumni adalah arsip tambahan, bukan pemindahan data."

**PEMAHAMAN MATERI:**
Ini bagian yang paling rawan salah ucap kalau tidak hati-hati — jangan sampai terlontar kata "dipindahkan" atau "memindahkan", karena itu salah secara teknis. Kata yang benar adalah MENYALIN atau COPY. Penjelasannya: setelah trigger ini jalan, kalau lo SELECT dari tabel mahasiswa, datanya masih ada di sana, hanya statusnya yang berubah. Yang baru ditambahkan adalah baris baru di tabel mahasiswa_alumni — itu salinan, bukan pemindahan dari mahasiswa ke mahasiswa_alumni.

Kode triggernya:
```sql
CREATE TRIGGER after_update_status_mahasiswa
AFTER UPDATE ON mahasiswa
FOR EACH ROW
BEGIN
    IF NEW.status IN ('lulus','keluar','pindah') AND OLD.status != NEW.status THEN
        INSERT INTO mahasiswa_alumni (nrp, nama, angkatan, program_studi, keterangan)
        VALUES (NEW.nrp, NEW.nama, NEW.angkatan, NEW.program_studi, NEW.status);
    END IF;
END;
```

`NEW` merujuk ke nilai kolom SETELAH UPDATE, `OLD` merujuk ke nilai SEBELUM UPDATE. Kondisi `OLD.status != NEW.status` penting — ini memastikan trigger hanya jalan kalau status BERUBAH, bukan setiap kali ada UPDATE apa pun di tabel mahasiswa (misalnya kalau cuma ganti nama, trigger ini tidak akan terpicu).

---

## Slide 10: Optimalisasi Performa dan Integritas Referensial

**SKRIP:**
"Dari sisi performa, kami menerapkan connection pooling menggunakan metode mysql.createPool pada file config/mysql.js, yang mengeliminasi overhead pemborosan waktu akibat siklus jabat tangan TCP yang berulang setiap kali ada pemanggilan endpoint API. Dari sisi integritas data, kami menerapkan foreign key constraint yang mengunci konsistensi data antar tabel relasional. Sebagai analisis arsitektur, kami juga ingin menjelaskan kenapa data alumni di-copy, bukan di-cut: atribut nrp pada tabel mahasiswa adalah fondasi relasi utama bagi tabel KRS dan tabel Nilai. Jika data alumni dihapus total dari tabel mahasiswa dan dipindahkan ke tabel eksternal, seluruh riwayat nilai pada masa lampau akan kehilangan relasi induknya, atau menjadi orphan record, dan merusak validitas laporan akademik."

**PEMAHAMAN MATERI:**
Connection pooling: tanpa pooling, setiap kali ada request masuk ke backend, Node.js harus membuka koneksi baru ke MySQL (proses TCP handshake), lalu menutupnya lagi setelah selesai — ini mahal secara waktu kalau dilakukan berulang-ulang untuk setiap request. Dengan pool, sejumlah koneksi dibuka di awal dan disimpan dalam "kolam", lalu dipakai bergantian oleh request yang masuk, tanpa perlu buka-tutup koneksi setiap saat.

Foreign key constraint: ini alasan teknis kenapa kemarin sempat muncul error pas mencoba menghapus KRS yang masih punya entri Nilai terkait — MySQL menolak penghapusan itu karena akan menyisakan baris Nilai yang menunjuk ke KRS yang sudah tidak ada (orphan record). Constraint inilah yang menjaga supaya hal itu tidak terjadi.

Slide ini juga jadi pasangan logis dari slide 9 — kalau dosen baru saja dengar "menyalin, bukan memindahkan" di slide sebelumnya, slide ini menjawab PERTANYAAN LANJUTANNYA: "kenapa harus menyalin, bukan memindahkan?" Jawabannya ya karena alasan integritas referensial ini.

---

## Slide 11 (Penutup PPT / Link GitHub)

**SKRIP:**
"Baik, itu adalah rancangan dan arsitektur sistem kami secara keseluruhan. Selanjutnya, kami akan menunjukkan repository project ini di GitHub, sebelum melanjutkan ke demo langsung melalui terminal."

**PEMAHAMAN MATERI:**
Tidak ada konten teknis baru, ini transisi murni. Pastikan tab GitHub sudah dibuka di background sebelum bagian ini dimulai, supaya tidak ada waktu terbuang menunggu loading saat pindah dari slide ke browser.

---

# BAGIAN 2 — GITHUB

**SKRIP:**
"Ini adalah repository kami di GitHub, dengan nama FP-SBD-085-076-079. Strukturnya terdiri dari folder config yang menyimpan konfigurasi koneksi database, folder routes yang menyimpan seluruh endpoint API per entitas, dan folder sql yang menyimpan DDL, DML, dan query. Ada juga folder docs yang menyimpan laporan, ERD, dan dokumentasi normalisasi. File index.js adalah entry point dari aplikasi backend kami."

**PEMAHAMAN MATERI:**
Kalau dosen minta dibuka salah satu file langsung dari GitHub (bukan dari terminal), siap saja — isinya sama dengan yang akan dijelaskan detail di sesi penjelasan kode nanti. Tidak perlu jelaskan detail isi file di sini, cukup tunjukkan struktur foldernya saja supaya transisi ke demo terminal tidak kepanjangan.

---

# BAGIAN 3 — DEMO LIVE TERMINAL

## Persiapan (sebelum mulai berbicara ke dosen)
Pastikan sudah dilakukan SEBELUM sesi presentasi mulai:
```cmd
mysql -u root -p -e "DROP DATABASE IF EXISTS akademik_db;"
mysql -u root -p < sql\ddl.sql
mysql -u root -p akademik_db < sql\dml.sql
ysql -u root -p akademik_db < sql\query.sql
```
Server BELUM dinyalakan — supaya bagian menyalakan server kelihatan sebagai bagian dari demo live, bukan sudah disiapkan dari sebelumnya.

## Test Case 1: Menyalakan Server

**SKRIP:**
"Sekarang saya akan menyalakan server backend kami secara langsung."

```cmd
node index.js
```

"Seperti yang terlihat, server sudah berjalan di port 3000."

**PEMAHAMAN MATERI:**
Kalau muncul error di sini (port sudah terpakai, dll), jangan panik — biasanya disebabkan oleh proses sebelumnya yang belum benar-benar berhenti. Solusi cepat: cek dengan `netstat -ano | findstr :3000` untuk melihat proses apa yang memakai port itu, atau cukup tutup semua terminal lama dan buka yang baru.

## Test Case 2: Endpoint Dasar — Semua Mahasiswa

**SKRIP:**
"Pertama, saya akan mengambil seluruh data mahasiswa lewat endpoint GET /mahasiswa."

```cmd
curl http://localhost:3000/mahasiswa
```

"Seperti yang terlihat, muncul delapan data mahasiswa lengkap dengan status masing-masing, termasuk yang aktif, cuti, dan lulus."

**PEMAHAMAN MATERI:**
Ini endpoint paling sederhana, query-nya cukup `SELECT * FROM mahasiswa`. Tujuan menampilkan ini di awal adalah membuktikan koneksi backend ke database berjalan normal sebelum masuk ke endpoint yang lebih kompleks.

## Test Case 3: Ranking IPK

**SKRIP:**
"Selanjutnya, saya akan menunjukkan endpoint untuk ranking IPK seluruh mahasiswa."

```cmd
curl http://localhost:3000/nilai/ipk
```

"Hasil ini diurutkan dari IPK tertinggi ke terendah, dihitung menggunakan weighted average berdasarkan SKS masing-masing mata kuliah."

**PEMAHAMAN MATERI:**
Kalau ditanya kenapa weighted average bukan rata-rata biasa: karena mata kuliah dengan SKS lebih besar (misalnya 4 SKS) seharusnya punya bobot lebih besar terhadap IPK dibanding mata kuliah 2 SKS. Rumus yang dipakai: SUM(nilai_angka × sks) dibagi SUM(sks).

## Test Case 4: Fitur Tambahan — Mahasiswa yang Mengulang

**SKRIP:**
"Ini adalah salah satu fitur tambahan yang diminta oleh dosen wali, yaitu daftar mahasiswa yang mengulang mata kuliah."

```cmd
curl http://localhost:3000/mahasiswa/mengulang/list
```

"Terlihat di sini Fajar Nugroho mengambil mata kuliah Algoritma dan Pemrograman sebanyak dua kali, dengan jumlah_ambil bernilai 2."

**PEMAHAMAN MATERI:**
Ini hasil dari view yang sudah dijelaskan di slide 7. Kalau dosen minta jelaskan ulang query-nya di sini, bisa diulang singkat: GROUP BY per nrp dan kode_mk, lalu HAVING COUNT lebih dari satu.

## Test Case 5: Fitur Tambahan — Mahasiswa Alumni

**SKRIP:**
"Fitur tambahan kedua adalah daftar mahasiswa yang telah lulus atau pindah."

```cmd
curl http://localhost:3000/mahasiswa/alumni/list
```

"Saat ini baru muncul satu data, yaitu Hadi Santoso yang sudah lulus. Nanti di sesi CRUD, kami akan menunjukkan bagaimana data baru bisa masuk ke tabel ini secara otomatis."

**PEMAHAMAN MATERI:**
Ini sengaja ditunjukkan SEBELUM CRUD supaya nanti pas demo trigger di sesi CRUD, dosen punya pembanding "sebelum" dan "sesudah" yang jelas — sebelumnya cuma satu data, nanti setelah UPDATE status akan bertambah.

## Test Case 6: Endpoint Trash

**SKRIP:**
"Terakhir, kami juga menyediakan endpoint untuk melihat arsip data yang telah dihapus, sebagai bukti bahwa mekanisme soft-delete kami bisa diakses bukan hanya dari level database, tapi juga dari backend."

```cmd
curl http://localhost:3000/trash
```

"Saat ini masih kosong karena belum ada data yang dihapus. Nanti akan kami tunjukkan isinya setelah sesi CRUD."

**PEMAHAMAN MATERI:**
Sama seperti test case 5, ini sengaja ditunjukkan kosong dulu sebagai pembanding sebelum-sesudah pada sesi CRUD nanti.

---

# BAGIAN 4 — CRUD

**SKRIP (transisi):**
"Sekarang kami akan menunjukkan operasi CRUD secara langsung melalui MySQL, sekaligus membuktikan trigger yang sudah kami jelaskan tadi benar-benar berjalan."

```cmd
mysql -u root -p
```
```sql
USE akademik_db;
```

## Test Case 7: CREATE

**SKRIP:**
"Pertama, operasi CREATE, menambahkan mahasiswa baru."

```sql
INSERT INTO mahasiswa VALUES ('5027231009','Demo Testing',2024,'Informatika','aktif');
SELECT * FROM mahasiswa WHERE nrp = '5027231009';
```

**PEMAHAMAN MATERI:**
Operasi paling dasar, tidak banyak yang perlu dijelaskan kecuali kalau dosen tanya soal urutan kolom di VALUES — itu harus sesuai urutan kolom didefinisikan di DDL (nrp, nama, angkatan, program_studi, status).

## Test Case 8: UPDATE Biasa

**SKRIP:**
"Selanjutnya, operasi UPDATE biasa, mengubah program studi mahasiswa ini."

```sql
UPDATE mahasiswa SET program_studi = 'Sistem Informasi' WHERE nrp = '5027231009';
SELECT * FROM mahasiswa WHERE nrp = '5027231009';
```

**PEMAHAMAN MATERI:**
Tujuan dari UPDATE biasa ini sebelum UPDATE status adalah pembanding: tunjukkan bahwa UPDATE kolom selain status TIDAK memicu trigger apa pun. Kalau diminta buktikan, bisa cek `SELECT * FROM mahasiswa_alumni` di sini dulu — jumlahnya masih sama seperti sebelumnya, karena trigger after_update_status_mahasiswa hanya bereaksi pada perubahan kolom status, bukan kolom lain.

## Test Case 9: UPDATE yang Memicu Trigger Alumni

**SKRIP:**
"Sekarang yang paling penting: kami akan mengubah status mahasiswa ini menjadi pindah, untuk membuktikan trigger after_update_status_mahasiswa bekerja secara otomatis."

```sql
UPDATE mahasiswa SET status = 'pindah' WHERE nrp = '5027231009';
SELECT * FROM mahasiswa_alumni;
```

"Seperti terlihat, tanpa kami melakukan INSERT manual ke tabel mahasiswa_alumni, data ini otomatis muncul di sana, dengan keterangan pindah."

**PEMAHAMAN MATERI:**
Ini test case paling penting di seluruh sesi CRUD. Pastikan betul-betul menunjukkan isi mahasiswa_alumni SEBELUM dan SESUDAH UPDATE ini, supaya perubahannya jelas terlihat. Kalau dosen tanya "coba update lagi statusnya jadi lulus, apa yang terjadi?" — jawaban yang benar: trigger tidak akan terpicu lagi untuk perubahan dari 'pindah' ke 'lulus', KARENA kondisi triggernya hanya cek `OLD.status != NEW.status` DAN `NEW.status IN (...)` — jadi sebenarnya tetap akan terpicu (karena status berubah dan status baru termasuk dalam daftar), TAPI akan menghasilkan ENTRI BARU lagi di mahasiswa_alumni (duplikat untuk nrp yang sama), bukan mengupdate entri yang sudah ada. Ini limitation yang valid untuk diakui kalau ditanya — desain saat ini tidak mengecek duplikasi di tabel alumni.

## Test Case 10: DELETE yang Memicu Trash

**SKRIP:**
"Selanjutnya, operasi DELETE. Kami akan menghapus mahasiswa ini, dan menunjukkan bahwa datanya tidak langsung hilang, melainkan diarsipkan ke tabel trash."

```sql
DELETE FROM mahasiswa WHERE nrp = '5027231009';
SELECT * FROM trash WHERE nama_tabel = 'mahasiswa' ORDER BY deleted_at DESC LIMIT 1;
```

"Terlihat data yang baru dihapus tersimpan dalam format JSON di tabel trash, lengkap dengan waktu penghapusannya."

**PEMAHAMAN MATERI:**
Kalau mahasiswa ini sebelumnya masih punya entri KRS yang aktif, DELETE ini akan GAGAL karena foreign key constraint (seperti yang sempat dialami kemarin). Solusinya jika itu terjadi saat demo: hapus dulu entri KRS dan Nilai terkait sebelum menghapus mahasiswa-nya, atau pilih nrp lain yang memang tidak punya KRS terkait sama sekali (seperti nrp demo yang baru dibuat di test case 7, yang memang belum pernah didaftarkan ke KRS apa pun — ini aman dihapus langsung).

## Test Case 11: Buktikan Trash Bisa Diakses dari Backend

**SKRIP:**
"Untuk menutup sesi CRUD, kami akan kembali ke terminal backend, dan menunjukkan bahwa data yang baru kami hapus ini juga bisa diakses melalui endpoint API, bukan hanya lewat database secara langsung."

```sql
exit;
```
```cmd
curl http://localhost:3000/trash
```

"Seperti terlihat, data yang sama persis yang baru kami lihat di MySQL, sekarang juga muncul lewat backend."

**PEMAHAMAN MATERI:**
Ini penutup yang elegan karena membuktikan dua hal sekaligus dalam satu test case: mekanisme soft-delete di level database BENAR berjalan, dan endpoint backend yang dibuat KHUSUS untuk mengakses arsip itu juga berfungsi. Ini menyatukan kembali jalur "demo live terminal" dan "CRUD" yang sempat berpisah jadi dua sesi.

---

# BAGIAN 5 — PENJELASAN KODE DAN QUERY

Bagian ini fleksibel sesuai waktu yang tersisa. Urutan prioritas kalau waktu terbatas: trigger dulu, lalu view, lalu route ordering. Kalau waktu masih banyak, bisa tambah index.js dan config/mysql.js.

## Penjelasan ddl.sql — Struktur Tabel

**SKRIP:**
"Untuk struktur tabel, kami ingin menyoroti tabel KRS sebagai contoh. Awalnya pada tahap normalisasi 1NF, primary key tabel ini adalah gabungan tiga kolom: nrp, kode_mk, dan id_semester. Pada implementasi final, kami menggunakan id_krs sebagai primary key auto increment, sementara ketiga kolom tersebut tetap dipertahankan sebagai foreign key, supaya relasinya tetap valid."

**PEMAHAMAN MATERI:**
Kalau ditanya kenapa tidak pakai composite key langsung sebagai PK: karena foreign key yang merujuk ke composite key (tiga kolom sekaligus) lebih berat dan rumit dibanding merujuk ke satu kolom integer. Misalnya tabel Nilai yang punya FK ke KRS — kalau PK KRS itu tiga kolom, FK di Nilai juga harus tiga kolom. Dengan id_krs sebagai PK tunggal, FK di Nilai cukup satu kolom saja (id_krs), lebih ringkas dan efisien.

## Penjelasan Trigger (BEFORE DELETE dan AFTER UPDATE)

Sudah dijelaskan detail di slide 8 dan 9 — bagian ini cukup mengulang singkat sambil menunjukkan kode aslinya langsung dari file, bukan dari slide.

**SKRIP:**
"Seperti yang sudah kami jelaskan di presentasi, trigger before_delete bekerja sebelum penghapusan data, menyalin ke tabel trash. Trigger after_update_status_mahasiswa bekerja setelah status berubah, menyalin data ke tabel mahasiswa_alumni."

## Penjelasan View

Sudah dijelaskan di slide 7 — cukup tunjukkan ulang file aslinya dan highlight bagian GROUP BY dan HAVING.

## Penjelasan index.js dan Routing

**SKRIP:**
"Pada file index.js, ini adalah entry point aplikasi kami. Setiap request yang masuk dengan alamat diawali /mahasiswa akan diarahkan ke file routes/mahasiswa.js, begitu juga untuk /nilai, /krs, dan /trash."

```javascript
app.use('/mahasiswa', require('./routes/mahasiswa'));
app.use('/nilai',     require('./routes/nilai'));
app.use('/krs',       require('./routes/krs'));
app.use('/trash',     require('./routes/trash'));
```

**PEMAHAMAN MATERI:**
Poin teknis yang sering ditanya: kenapa urutan route di dalam routes/mahasiswa.js itu penting?

```javascript
router.get('/alumni/list', ...);
router.get('/mengulang/list', ...);
router.get('/', ...);
router.get('/:nrp', ...);
```

`:nrp` adalah parameter dinamis yang akan menangkap APAPUN yang ditulis di posisi itu sebagai nilai nrp. Kalau urutannya dibalik — `/:nrp` ditulis sebelum `/alumni/list` — maka request ke `/mahasiswa/alumni/list` akan dianggap Express sebagai permintaan detail mahasiswa dengan nrp bernilai "alumni", bukan permintaan ke endpoint alumni yang sebenarnya. Itu sebabnya route yang spesifik harus selalu diletakkan SEBELUM route yang generik dengan parameter dinamis.

## Penjelasan config/mysql.js

**SKRIP:**
"Terakhir, file config/mysql.js ini adalah jembatan koneksi ke database, menggunakan connection pooling seperti yang sudah dijelaskan sebelumnya."

```javascript
const pool = mysql.createPool({
    host: process.env.MYSQL_HOST,
    user: process.env.MYSQL_USER,
    password: process.env.MYSQL_PASSWORD,
    database: process.env.MYSQL_DB
});
module.exports = pool.promise();
```

**PEMAHAMAN MATERI:**
`.promise()` di akhir itu mengubah cara pemanggilan query dari gaya callback (lama) menjadi gaya async/await (lebih modern dan mudah dibaca). Ini yang memungkinkan penulisan kode seperti `const [rows] = await db.query(...)` di file routes, bukan pakai callback bersarang yang lebih rumit dibaca.

---

# PENUTUP

**SKRIP:**
"Demikian presentasi dan demonstrasi dari kelompok kami. Sistem ini telah berhasil memenuhi seluruh kebutuhan normalisasi data hingga 3NF, serta dua kebutuhan tambahan dari dosen wali, yaitu pemantauan mata kuliah yang diulang dan pengarsipan mahasiswa lulus atau pindah, tanpa mengorbankan integritas data pada tabel utama. Kami terbuka untuk pertanyaan."

---

# CATATAN UMUM UNTUK SEMUA SESI

Hindari kata "memindahkan" terkait tabel mahasiswa_alumni — selalu gunakan "menyalin" atau "mengarsipkan".

Kalau ditanya soal validasi input atau keamanan (otentikasi), jawab jujur bahwa itu di luar scope FP ini dan menjadi catatan untuk pengembangan lebih lanjut — jangan berusaha mengklaim sistem ini punya fitur yang sebenarnya tidak ada.

Kalau ada error teknis saat demo live (server tidak nyala, password salah, dll), tetap tenang, jelaskan ke dosen apa yang terjadi sambil memperbaikinya — itu lebih baik dibanding terlihat panik atau mencoba menyembunyikan masalah.
