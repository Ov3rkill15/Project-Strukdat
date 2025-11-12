# Project Struktur Data: Sistem E-Commerce Blockchain

## Pengembangan Prototype Sistem E-Commerce Terdesentralisasi Menggunakan C++ dan Konsep Blockchain

Dokumentasi lengkap implementasi tugas besar struktur data yang mengeksplorasi penerapan berbagai struktur data dalam konteks sistem e-commerce berbasis blockchain.

## 📚 Dokumentasi

Repository ini berisi dokumentasi komprehensif untuk project tugas besar struktur data dengan topik:

**"Pengembangan Prototype Sistem E-Commerce Terdesentralisasi Menggunakan C++ dan Konsep Blockchain"**

### Daftar Dokumen

1. **[DOKUMENTASI.md](DOKUMENTASI.md)** - Dokumentasi utama project
   - Latar belakang dan tujuan
   - Overview sistem
   - Arsitektur dan komponen
   - Algoritma kriptografi
   - Flow transaksi
   - Keamanan dan validasi

2. **[STRUKTUR_DATA.md](STRUKTUR_DATA.md)** - Pembahasan mendalam struktur data
   - Linked List untuk Blockchain
   - Hash Table untuk manajemen data
   - Merkle Tree untuk verifikasi
   - Queue untuk transaction pool
   - Priority Queue untuk fee-based processing
   - Stack untuk rollback operations
   - Analisis kompleksitas dan perbandingan

3. **[IMPLEMENTASI.md](IMPLEMENTASI.md)** - Panduan implementasi kode
   - Setup dan dependencies
   - Core classes (Transaction, Block, Blockchain, dll)
   - Contoh implementasi lengkap
   - Cara kompilasi dan menjalankan
   - Testing dan troubleshooting

## 🎯 Tujuan Project

1. Mengimplementasikan konsep blockchain menggunakan C++
2. Menerapkan berbagai struktur data untuk sistem e-commerce
3. Membuat sistem transaksi yang aman dan terdesentralisasi
4. Mengintegrasikan fitur e-commerce dengan blockchain

## 🔑 Fitur Utama

- ✅ Blockchain implementation dengan Proof of Work
- ✅ Transaction management dengan digital signatures
- ✅ Product catalog management
- ✅ User wallet dan balance tracking
- ✅ Mining mechanism dengan rewards
- ✅ Chain validation dan integrity checking
- ✅ Smart contract untuk escrow

## 🛠️ Teknologi

- **Bahasa**: C++ (C++11 atau lebih baru)
- **Libraries**: OpenSSL untuk kriptografi
- **Struktur Data**: Linked List, Hash Table, Merkle Tree, Queue, Priority Queue, Stack

## 📖 Cara Menggunakan

1. Baca [DOKUMENTASI.md](DOKUMENTASI.md) untuk memahami konsep dan arsitektur sistem
2. Pelajari [STRUKTUR_DATA.md](STRUKTUR_DATA.md) untuk detail implementasi struktur data
3. Ikuti [IMPLEMENTASI.md](IMPLEMENTASI.md) untuk compile dan run code

## 🚀 Quick Start

```bash
# Clone repository
git clone https://github.com/Ov3rkill15/Project-Strukdat.git
cd Project-Strukdat

# Install dependencies (Ubuntu/Debian)
sudo apt-get install g++ libssl-dev

# Compile (lihat IMPLEMENTASI.md untuk detail)
g++ -std=c++11 main.cpp -o ecommerce -lssl -lcrypto

# Run
./ecommerce
```

## 📊 Struktur Data yang Digunakan

| Struktur Data | Penggunaan | Kompleksitas |
|---------------|------------|--------------|
| Linked List | Chain of blocks | O(1) insert, O(n) search |
| Hash Table | User/Product storage | O(1) average |
| Merkle Tree | Transaction verification | O(log n) verify |
| Queue | Transaction pool | O(1) |
| Priority Queue | Fee-based processing | O(log n) |
| Stack | Rollback operations | O(1) |

## 👥 Kontributor

Project ini dibuat sebagai tugas besar mata kuliah Struktur Data.

## 📄 Lisensi

Project ini dibuat untuk keperluan akademis.
