# Tugas Basis Data: Eksplorasi JOIN & Denormalisasi

Repositori ini berisi skrip SQL untuk mendemonstrasikan perbandingan kompleksitas *query* sebelum dan sesudah penerapan teknik denormalisasi pada basis data penjualan (`penjualan_db`).

## Struktur Basis Data
Pengujian dilakukan pada tiga tabel utama yang saling berelasi:
1. `pelanggan` (Tabel Master)
2. `pesanan` (Tabel Transaksi)
3. `detail_pesanan` (Tabel Detail Transaksi)

---

## 1. Query JOIN yang Kompleks (Tahap 1)
Pada tahap ini, perhitungan total belanja per pelanggan dilakukan secara *on-the-fly* (kondisi normalisasi penuh). Sistem harus memetakan relasi dari tiga tabel sekaligus dan melakukan kalkulasi matematika secara langsung menggunakan fungsi agregasi `SUM()`. Pendekatan ini memiliki kompleksitas tinggi dan boros *resource*.

```sql
SELECT 
    p.id_pelanggan,
    p.nama_pelanggan,
    SUM(dp.jumlah * dp.harga_satuan) AS kalkulasi_total_belanja
FROM pelanggan p
JOIN pesanan ps ON p.id_pelanggan = ps.id_pelanggan
JOIN detail_pesanan dp ON ps.id_pesanan = dp.id_pesanan
GROUP BY p.id_pelanggan, p.nama_pelanggan;
```

## 2. Script DDL Denormalisasi & Query Panggilannya (Tahap 2 & 3)
Untuk mengatasi beban komputasi dari Tahap 1, dilakukan denormalisasi dengan menambahkan satu atribut turunan yaitu `total_belanja` pada tabel master. Setelah didenormalisasi, pemanggilan data menjadi sangat ringan tanpa perlu `JOIN` atau agregasi.

```sql
-- [Tahap 2] A. DDL: Menambahkan atribut turunan pada tabel master
ALTER TABLE pelanggan 
ADD COLUMN total_belanja DECIMAL(15,2) DEFAULT 0;

-- [Tahap 2] B. DML: Mengisi kolom turunan dengan data historis transaksi
UPDATE pelanggan p
SET total_belanja = (
    SELECT COALESCE(SUM(dp.jumlah * dp.harga_satuan), 0)
    FROM pesanan ps
    JOIN detail_pesanan dp ON ps.id_pesanan = dp.id_pesanan
    WHERE ps.id_pelanggan = p.id_pelanggan
);

-- [Tahap 3] Pembuktian: Query pemanggilan dasbor yang sangat cepat
SELECT 
    id_pelanggan,
    nama_pelanggan,
    total_belanja
FROM pelanggan;
```
