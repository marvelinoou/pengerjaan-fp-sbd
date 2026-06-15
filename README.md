# Panduan Pengerjaan FP Sistem Basis Data
## Sistem Pengelolaan Akademik Mahasiswa dan Nilai — FP-SBD-085-076-079

Status saat ini: struktur folder (`config`, `routes`, `sql`), file dasar (`.env.example`, `.gitignore`, `README.md`, `index.js`, `package.json`) sudah di branch `main`, dan branch `dev` sudah dibuat. Panduan ini lanjut dari situ sampai selesai.

---

## TIMELINE 2 HARI

```
SENIN (hari ini)
  Pagi-Sore   -> Hugo (database), Lo (backend), Rapa (laporan) kerja paralel
  Sore-Malam  -> begitu satu branch selesai, langsung merge ke dev (jangan tunggu semua)

SELASA
  Pagi  -> pull dev, test semua endpoint + trigger + view
  Siang -> fix bug
  Sore  -> merge dev ke main
  Malam -> latihan demo per orang

RABU -> Demo
```

Prinsip: begitu satu branch siap, langsung merge ke `dev` — jangan tunggu semua orang selesai. Ini supaya conflict ketauan lebih awal, bukan numpuk di hari kedua.

---

## BAGIAN HUGO — Database Layer (branch `database`)

```cmd
git clone https://github.com/username-lo/FP-SBD-085-076-079.git
cd FP-SBD-085-076-079
git checkout dev
git pull origin dev
git checkout -b database
code .
```

### `sql\ddl.sql`

```sql
CREATE DATABASE akademik_db;
USE akademik_db;

CREATE TABLE mahasiswa (
    nrp VARCHAR(20) PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    angkatan YEAR NOT NULL,
    program_studi VARCHAR(50) NOT NULL,
    status ENUM('aktif','cuti','lulus','keluar') DEFAULT 'aktif'
);

CREATE TABLE dosen (
    nip VARCHAR(20) PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE mata_kuliah (
    kode_mk VARCHAR(10) PRIMARY KEY,
    nama_mk VARCHAR(100) NOT NULL,
    sks TINYINT NOT NULL
);

CREATE TABLE semester (
    id_semester INT AUTO_INCREMENT PRIMARY KEY,
    tahun YEAR NOT NULL,
    periode ENUM('ganjil','genap') NOT NULL
);

CREATE TABLE krs (
    id_krs INT AUTO_INCREMENT PRIMARY KEY,
    nrp VARCHAR(20) NOT NULL,
    kode_mk VARCHAR(10) NOT NULL,
    id_semester INT NOT NULL,
    status ENUM('aktif','batal') DEFAULT 'aktif',
    FOREIGN KEY (nrp) REFERENCES mahasiswa(nrp),
    FOREIGN KEY (kode_mk) REFERENCES mata_kuliah(kode_mk),
    FOREIGN KEY (id_semester) REFERENCES semester(id_semester)
);

CREATE TABLE nilai (
    id_nilai INT AUTO_INCREMENT PRIMARY KEY,
    id_krs INT NOT NULL UNIQUE,
    nilai_huruf ENUM('A','AB','B','BC','C','D','E') NOT NULL,
    nilai_angka DECIMAL(3,2) NOT NULL,
    FOREIGN KEY (id_krs) REFERENCES krs(id_krs)
);

CREATE TABLE dosen_mengajar (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nip VARCHAR(20) NOT NULL,
    kode_mk VARCHAR(10) NOT NULL,
    id_semester INT NOT NULL,
    FOREIGN KEY (nip) REFERENCES dosen(nip),
    FOREIGN KEY (kode_mk) REFERENCES mata_kuliah(kode_mk),
    FOREIGN KEY (id_semester) REFERENCES semester(id_semester)
);

CREATE TABLE trash (
    id_trash INT AUTO_INCREMENT PRIMARY KEY,
    nama_tabel VARCHAR(50) NOT NULL,
    data_json JSON NOT NULL,
    deleted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_by VARCHAR(100)
);

CREATE TABLE mahasiswa_alumni (
    id_alumni INT AUTO_INCREMENT PRIMARY KEY,
    nrp VARCHAR(20) NOT NULL,
    nama VARCHAR(100) NOT NULL,
    angkatan YEAR NOT NULL,
    program_studi VARCHAR(50) NOT NULL,
    keterangan ENUM('lulus','pindah','keluar') NOT NULL,
    tanggal_arsip TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (nrp) REFERENCES mahasiswa(nrp)
);

DELIMITER $$

CREATE TRIGGER before_delete_mahasiswa
BEFORE DELETE ON mahasiswa FOR EACH ROW
BEGIN
    INSERT INTO trash (nama_tabel, data_json, deleted_by)
    VALUES ('mahasiswa', JSON_OBJECT(
        'nrp', OLD.nrp, 'nama', OLD.nama,
        'angkatan', OLD.angkatan, 'program_studi', OLD.program_studi,
        'status', OLD.status
    ), 'system');
END$$

CREATE TRIGGER before_delete_dosen
BEFORE DELETE ON dosen FOR EACH ROW
BEGIN
    INSERT INTO trash (nama_tabel, data_json, deleted_by)
    VALUES ('dosen', JSON_OBJECT(
        'nip', OLD.nip, 'nama', OLD.nama, 'email', OLD.email
    ), 'system');
END$$

CREATE TRIGGER before_delete_mata_kuliah
BEFORE DELETE ON mata_kuliah FOR EACH ROW
BEGIN
    INSERT INTO trash (nama_tabel, data_json, deleted_by)
    VALUES ('mata_kuliah', JSON_OBJECT(
        'kode_mk', OLD.kode_mk, 'nama_mk', OLD.nama_mk, 'sks', OLD.sks
    ), 'system');
END$$

CREATE TRIGGER before_delete_krs
BEFORE DELETE ON krs FOR EACH ROW
BEGIN
    INSERT INTO trash (nama_tabel, data_json, deleted_by)
    VALUES ('krs', JSON_OBJECT(
        'id_krs', OLD.id_krs, 'nrp', OLD.nrp,
        'kode_mk', OLD.kode_mk, 'id_semester', OLD.id_semester,
        'status', OLD.status
    ), 'system');
END$$

CREATE TRIGGER after_update_status_mahasiswa
AFTER UPDATE ON mahasiswa
FOR EACH ROW
BEGIN
    IF NEW.status IN ('lulus','keluar') AND OLD.status != NEW.status THEN
        INSERT INTO mahasiswa_alumni (nrp, nama, angkatan, program_studi, keterangan)
        VALUES (NEW.nrp, NEW.nama, NEW.angkatan, NEW.program_studi, NEW.status);
    END IF;
END$$

DELIMITER ;
```

### `sql\dml.sql`

```sql
USE akademik_db;

INSERT INTO semester (tahun, periode) VALUES
(2024,'ganjil'),(2024,'genap'),(2025,'ganjil');

INSERT INTO mahasiswa VALUES
('5027231001','Andi Pratama',2023,'Informatika','aktif'),
('5027231002','Budi Santoso',2023,'Informatika','aktif'),
('5027231003','Citra Dewi',2023,'Sistem Informasi','aktif'),
('5027231004','Dani Rahmat',2023,'Sistem Informasi','aktif'),
('5027231005','Eka Putri',2022,'Informatika','aktif'),
('5027231006','Fajar Nugroho',2022,'Teknik Komputer','aktif'),
('5027231007','Gita Kusuma',2023,'Sistem Informasi','cuti'),
('5027231008','Hadi Santoso',2021,'Informatika','lulus');

INSERT INTO dosen VALUES
('197501012000031001','Dr. Hendra Wijaya','hendra@its.ac.id'),
('198003052005012002','Dr. Siti Aminah','siti@its.ac.id'),
('197812152003121003','Prof. Budi Raharjo','budi@its.ac.id');

INSERT INTO mata_kuliah VALUES
('IF2110','Algoritma dan Pemrograman',3),
('IF2210','Struktur Data',3),
('IF2310','Basis Data',3),
('IF2410','Sistem Operasi',3),
('IF2510','Jaringan Komputer',3);

INSERT INTO krs (nrp, kode_mk, id_semester, status) VALUES
('5027231001','IF2110',1,'aktif'),
('5027231001','IF2210',1,'aktif'),
('5027231001','IF2310',2,'aktif'),
('5027231001','IF2410',2,'aktif'),
('5027231002','IF2110',1,'aktif'),
('5027231002','IF2210',1,'aktif'),
('5027231002','IF2310',2,'aktif'),
('5027231003','IF2110',1,'aktif'),
('5027231003','IF2210',1,'aktif'),
('5027231003','IF2510',3,'aktif'),
('5027231004','IF2110',1,'aktif'),
('5027231004','IF2310',2,'aktif'),
('5027231005','IF2110',1,'aktif'),
('5027231005','IF2410',2,'aktif'),
('5027231005','IF2510',3,'aktif'),
('5027231006','IF2110',1,'aktif'),
('5027231006','IF2210',1,'aktif'),
('5027231006','IF2110',2,'aktif');

INSERT INTO nilai (id_krs, nilai_huruf, nilai_angka) VALUES
(1,'A',4.00),(2,'AB',3.50),(3,'A',4.00),(4,'AB',3.50),
(5,'B',3.00),(6,'A',4.00),(7,'B',3.00),
(8,'AB',3.50),(9,'B',3.00),(10,'AB',3.50),
(11,'BC',2.50),(12,'B',3.00),
(13,'A',4.00),(14,'A',4.00),(15,'AB',3.50),
(16,'C',2.00),(17,'BC',2.50),(18,'B',3.00);

INSERT INTO dosen_mengajar (nip, kode_mk, id_semester) VALUES
('197501012000031001','IF2110',1),
('198003052005012002','IF2210',1),
('197812152003121003','IF2310',2),
('197501012000031001','IF2410',2),
('198003052005012002','IF2510',3);

INSERT INTO mahasiswa_alumni (nrp, nama, angkatan, program_studi, keterangan) VALUES
('5027231008','Hadi Santoso',2021,'Informatika','lulus');
```

### `sql\query.sql`

```sql
USE akademik_db;

-- 1. Transkrip lengkap satu mahasiswa
SELECT m.nrp, m.nama, mk.kode_mk, mk.nama_mk, mk.sks,
       s.tahun, s.periode, n.nilai_huruf, n.nilai_angka
FROM mahasiswa m
JOIN krs k ON m.nrp = k.nrp
JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
JOIN semester s ON k.id_semester = s.id_semester
JOIN nilai n ON k.id_krs = n.id_nilai
WHERE m.nrp = '5027231001';

-- 2. Hitung IPK satu mahasiswa
SELECT m.nrp, m.nama,
       ROUND(SUM(n.nilai_angka * mk.sks) / SUM(mk.sks), 2) AS ipk,
       SUM(mk.sks) AS total_sks
FROM mahasiswa m
JOIN krs k ON m.nrp = k.nrp
JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
JOIN nilai n ON k.id_krs = n.id_nilai
WHERE m.nrp = '5027231001' AND k.status = 'aktif'
GROUP BY m.nrp, m.nama;

-- 3. Ranking IPK semua mahasiswa
SELECT m.nrp, m.nama, m.program_studi,
       ROUND(SUM(n.nilai_angka * mk.sks) / SUM(mk.sks), 2) AS ipk,
       SUM(mk.sks) AS total_sks
FROM mahasiswa m
JOIN krs k ON m.nrp = k.nrp
JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
JOIN nilai n ON k.id_krs = n.id_nilai
WHERE k.status = 'aktif'
GROUP BY m.nrp, m.nama, m.program_studi
ORDER BY ipk DESC;

-- 4. KHS per semester
SELECT m.nama, mk.nama_mk, mk.sks, n.nilai_huruf, n.nilai_angka,
       s.tahun, s.periode
FROM krs k
JOIN mahasiswa m ON k.nrp = m.nrp
JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
JOIN nilai n ON k.id_krs = n.id_nilai
JOIN semester s ON k.id_semester = s.id_semester
WHERE m.nrp = '5027231001' AND s.id_semester = 1;

-- 5. Dosen mengajar matkul apa saja
SELECT d.nama AS dosen, mk.nama_mk, mk.sks, s.tahun, s.periode
FROM dosen_mengajar dm
JOIN dosen d ON dm.nip = d.nip
JOIN mata_kuliah mk ON dm.kode_mk = mk.kode_mk
JOIN semester s ON dm.id_semester = s.id_semester;

-- 6. Lihat isi trash
SELECT * FROM trash ORDER BY deleted_at DESC;

-- 7. View: matkul yang diulang per mahasiswa
CREATE VIEW view_mahasiswa_mengulang AS
SELECT m.nrp, m.nama, mk.kode_mk, mk.nama_mk, COUNT(*) AS jumlah_ambil
FROM krs k
JOIN mahasiswa m ON k.nrp = m.nrp
JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
GROUP BY m.nrp, m.nama, mk.kode_mk, mk.nama_mk
HAVING COUNT(*) > 1;

SELECT * FROM view_mahasiswa_mengulang;
```

### Commit & Push (Hugo)

```cmd
git add .
git commit -m "feat: DDL, trigger, DML, query, view mengulang, tabel alumni"
git push origin database
```

---

## BAGIAN LO — Backend Layer (branch `backend`)

```cmd
cd FP-SBD-085-076-079
git checkout dev
git pull origin dev
git checkout -b backend
code .
```

### `config\mysql.js`

```javascript
const mysql = require('mysql2');
require('dotenv').config();

const pool = mysql.createPool({
    host: process.env.MYSQL_HOST,
    user: process.env.MYSQL_USER,
    password: process.env.MYSQL_PASSWORD,
    database: process.env.MYSQL_DB
});

module.exports = pool.promise();
```

### `routes\mahasiswa.js`

```javascript
const express = require('express');
const router = express.Router();
const db = require('../config/mysql');

// route spesifik diletakkan SEBELUM /:nrp supaya tidak ketangkep jadi parameter nrp
router.get('/alumni/list', async (req, res) => {
    try {
        const [rows] = await db.query('SELECT * FROM mahasiswa_alumni');
        res.json(rows);
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.get('/mengulang/list', async (req, res) => {
    try {
        const [rows] = await db.query('SELECT * FROM view_mahasiswa_mengulang');
        res.json(rows);
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.get('/', async (req, res) => {
    try {
        const [rows] = await db.query('SELECT * FROM mahasiswa');
        res.json(rows);
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.get('/:nrp', async (req, res) => {
    try {
        const [rows] = await db.query('SELECT * FROM mahasiswa WHERE nrp = ?', [req.params.nrp]);
        if (!rows.length) return res.status(404).json({ message: 'Tidak ditemukan' });
        res.json(rows[0]);
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.post('/', async (req, res) => {
    const { nrp, nama, angkatan, program_studi, status } = req.body;
    try {
        await db.query('INSERT INTO mahasiswa VALUES (?,?,?,?,?)',
            [nrp, nama, angkatan, program_studi, status || 'aktif']);
        res.status(201).json({ message: 'Mahasiswa ditambahkan' });
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.put('/:nrp', async (req, res) => {
    const { nama, angkatan, program_studi, status } = req.body;
    try {
        await db.query(
            'UPDATE mahasiswa SET nama=?, angkatan=?, program_studi=?, status=? WHERE nrp=?',
            [nama, angkatan, program_studi, status, req.params.nrp]);
        res.json({ message: 'Mahasiswa diupdate' });
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.delete('/:nrp', async (req, res) => {
    try {
        await db.query('DELETE FROM mahasiswa WHERE nrp = ?', [req.params.nrp]);
        res.json({ message: 'Mahasiswa dihapus, data masuk trash' });
    } catch (err) { res.status(500).json({ error: err.message }); }
});

module.exports = router;
```

### `routes\nilai.js`

```javascript
const express = require('express');
const router = express.Router();
const db = require('../config/mysql');

router.get('/ipk', async (req, res) => {
    try {
        const [rows] = await db.query(`
            SELECT m.nrp, m.nama, m.program_studi,
                   ROUND(SUM(n.nilai_angka * mk.sks) / SUM(mk.sks), 2) AS ipk,
                   SUM(mk.sks) AS total_sks
            FROM mahasiswa m
            JOIN krs k ON m.nrp = k.nrp
            JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
            JOIN nilai n ON k.id_krs = n.id_nilai
            WHERE k.status = 'aktif'
            GROUP BY m.nrp, m.nama, m.program_studi
            ORDER BY ipk DESC
        `);
        res.json(rows);
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.post('/', async (req, res) => {
    const { id_krs, nilai_huruf, nilai_angka } = req.body;
    try {
        await db.query(
            'INSERT INTO nilai (id_krs, nilai_huruf, nilai_angka) VALUES (?,?,?)',
            [id_krs, nilai_huruf, nilai_angka]);
        res.status(201).json({ message: 'Nilai diinput' });
    } catch (err) { res.status(500).json({ error: err.message }); }
});

module.exports = router;
```

### `routes\krs.js`

```javascript
const express = require('express');
const router = express.Router();
const db = require('../config/mysql');

router.get('/:nrp/semester/:id_semester', async (req, res) => {
    try {
        const [rows] = await db.query(`
            SELECT mk.nama_mk, mk.sks, n.nilai_huruf, n.nilai_angka,
                   s.tahun, s.periode
            FROM krs k
            JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
            JOIN nilai n ON k.id_krs = n.id_nilai
            JOIN semester s ON k.id_semester = s.id_semester
            WHERE k.nrp = ? AND k.id_semester = ?
        `, [req.params.nrp, req.params.id_semester]);
        res.json(rows);
    } catch (err) { res.status(500).json({ error: err.message }); }
});

router.post('/', async (req, res) => {
    const { nrp, kode_mk, id_semester } = req.body;
    try {
        await db.query(
            'INSERT INTO krs (nrp, kode_mk, id_semester) VALUES (?,?,?)',
            [nrp, kode_mk, id_semester]);
        res.status(201).json({ message: 'KRS ditambahkan' });
    } catch (err) { res.status(500).json({ error: err.message }); }
});

module.exports = router;
```

### `index.js`

```javascript
const express = require('express');
require('dotenv').config();

const app = express();
app.use(express.json());

app.use('/mahasiswa', require('./routes/mahasiswa'));
app.use('/nilai',     require('./routes/nilai'));
app.use('/krs',       require('./routes/krs'));

app.listen(process.env.PORT, () => {
    console.log(`Server jalan di port ${process.env.PORT}`);
});
```

### Update `README.md` — tambahkan bagian Endpoint di paling bawah

```markdown
## Endpoint
GET    /mahasiswa                 semua mahasiswa
GET    /mahasiswa/:nrp            detail mahasiswa
POST   /mahasiswa                 tambah mahasiswa
PUT    /mahasiswa/:nrp            update mahasiswa
DELETE /mahasiswa/:nrp            hapus (masuk trash otomatis)
GET    /mahasiswa/alumni/list     daftar mahasiswa lulus/pindah
GET    /mahasiswa/mengulang/list  daftar mahasiswa yang mengulang matkul
GET    /nilai/ipk                 ranking IPK semua mahasiswa
POST   /nilai                     input nilai
GET    /krs/:nrp/semester/:id     KHS per semester
POST   /krs                       tambah KRS
```

### Commit & Push (Lo)

```cmd
npm install express mysql2 dotenv

git add .
git commit -m "feat: config mysql, semua routes (termasuk alumni & mengulang), index.js"
git push origin backend
```

---

## BAGIAN RAPA — Laporan (branch `laporan`)

```cmd
git clone https://github.com/username-lo/FP-SBD-085-076-079.git
cd FP-SBD-085-076-079
git checkout dev
git pull origin dev
git checkout -b laporan
```

Isi folder `docs\` dengan:
- File Excel normalisasi v3 (data mentah -> 1NF -> 2NF -> 3NF, sudah termasuk kasus Fajar mengulang dan tabel `MAHASISWA_ALUMNI`)
- Laporan formal (docx/PDF) berisi ERD final, normalisasi, struktur tabel, dan penjelasan fitur tambahan
- Penjelasan kenapa data alumni **disalin** bukan **dipindah** dari tabel `mahasiswa` — karena FK dari `krs` dan `nilai` ke `mahasiswa.nrp` akan rusak kalau dipindah

```cmd
git add docs\
git commit -m "docs: ERD final, normalisasi v3 (kasus mengulang + tabel alumni), laporan"
git push origin laporan
```

---

## MERGE KE DEV (Lo) — Lakukan Bertahap, Jangan Tunggu Semua

Begitu branch `database` siap (biasanya paling cepat, prioritaskan):

```cmd
git checkout dev
git pull origin dev
git merge database
git push origin dev
```

Begitu branch `backend` siap:

```cmd
git merge backend
git push origin dev
```

Begitu branch `laporan` siap:

```cmd
git merge laporan
git push origin dev
```

**Kalau ada conflict** (biasanya di `README.md` karena dua orang edit file yang sama):

```cmd
:: VS Code highlight bagian yang conflict
:: Buka file, cari tanda:
:: <<<<<<< HEAD
:: ...versi dev...
:: =======
:: ...versi branch lain...
:: >>>>>>> nama-branch

:: Pilih yang benar, hapus semua tanda <<<, ===, >>>
git add .
git commit -m "fix: resolve conflict"
git push origin dev
```

---

## TEST LOKAL (Selasa Pagi)

```cmd
git checkout dev
git pull origin dev

copy .env.example .env
code .env
```

Isi `.env`:
```
MYSQL_HOST=localhost
MYSQL_USER=root
MYSQL_PASSWORD=isi_password_lo
MYSQL_DB=akademik_db
PORT=3000
```

Setup database:

```cmd
mysql -u root -p < sql\ddl.sql
mysql -u root -p akademik_db < sql\dml.sql
```

Jalankan server:

```cmd
npm install
node index.js
```

Test semua endpoint:

```cmd
curl http://localhost:3000/mahasiswa
curl http://localhost:3000/mahasiswa/5027231001
curl http://localhost:3000/mahasiswa/alumni/list
curl http://localhost:3000/mahasiswa/mengulang/list
curl http://localhost:3000/nilai/ipk
curl http://localhost:3000/krs/5027231001/semester/1
```

Hasil yang diharapkan:
- `/mahasiswa/alumni/list` -> muncul Hadi Santoso (lulus)
- `/mahasiswa/mengulang/list` -> muncul Fajar Nugroho, IF2110, jumlah_ambil = 2

Test trigger alumni (live, langsung di MySQL):

```sql
UPDATE mahasiswa SET status = 'lulus' WHERE nrp = '5027231005';
SELECT * FROM mahasiswa_alumni;
```

Eka Putri harus otomatis muncul di `mahasiswa_alumni` tanpa di-INSERT manual — ini bukti trigger jalan.

---

## FINAL MERGE KE MAIN (Selasa Sore)

```cmd
git checkout main
git pull origin main
git merge dev
git push origin main
```

---

## CHECKLIST DEMO (Selasa Malam)

Tiap orang harus bisa:

**Hugo** — jelasin tiap tabel di `ddl.sql`, kenapa trigger `before_delete_*` beda dari `after_update_status_mahasiswa` (BEFORE vs AFTER, DELETE vs UPDATE), dan cara kerja view `view_mahasiswa_mengulang` (GROUP BY + HAVING COUNT > 1).

**Lo** — jelasin alur request masuk dari `index.js` -> `routes/*.js` -> `config/mysql.js` -> MySQL, dan kenapa route `/alumni/list` & `/mengulang/list` diletakkan sebelum `/:nrp`.

**Rapa** — jelasin proses normalisasi 1NF->3NF pakai data Fajar sebagai contoh konkret "mengulang", dan kenapa `mahasiswa_alumni` itu tabel arsip (disalin) bukan pemindahan data.

**Semua** — siap kalau diminta tulis ulang salah satu query dari nol, atau modifikasi tabel/data on-the-spot saat demo.

---

## CHEATSHEET GIT

```cmd
git status               :: cek file yang berubah
git branch                :: lihat semua branch
git checkout nama-branch  :: pindah branch
git checkout -b nama-baru :: bikin branch baru, pindah ke situ
git pull origin dev       :: ambil update terbaru dari dev
git add .                 :: tandai semua perubahan
git commit -m "pesan"     :: bikin snapshot
git push origin branch    :: upload ke GitHub
git merge nama-branch     :: gabungin branch lain ke branch sekarang
git log --oneline         :: lihat riwayat commit
```
