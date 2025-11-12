# Pengembangan Prototype Sistem E-Commerce Terdesentralisasi Menggunakan C++ dan Konsep Blockchain

## Deskripsi Project

Project ini merupakan implementasi prototype sistem e-commerce terdesentralisasi yang menggunakan konsep blockchain. Sistem ini dibangun menggunakan bahasa pemrograman C++ dan memanfaatkan berbagai struktur data untuk mengimplementasikan fitur-fitur blockchain dan e-commerce.

## Latar Belakang

E-commerce tradisional bergantung pada sistem terpusat yang rentan terhadap:
- Single point of failure
- Manipulasi data
- Ketergantungan pada pihak ketiga
- Biaya transaksi tinggi

Dengan menggunakan konsep blockchain, sistem e-commerce dapat menjadi lebih:
- Transparan
- Aman
- Terdesentralisasi
- Dapat diaudit

## Tujuan Project

1. Mengimplementasikan konsep blockchain menggunakan C++
2. Menerapkan struktur data yang tepat untuk sistem e-commerce
3. Membuat sistem transaksi yang aman dan terdesentralisasi
4. Mengintegrasikan fitur-fitur e-commerce dengan blockchain

## Struktur Data yang Digunakan

### 1. Linked List
Digunakan untuk menyimpan chain of blocks. Setiap block terhubung dengan block sebelumnya melalui pointer, membentuk blockchain.

**Kegunaan:**
- Menyimpan history transaksi secara berurutan
- Memudahkan traversal dari genesis block ke block terbaru
- Menjaga integritas data dengan hash linking

### 2. Hash Table
Digunakan untuk menyimpan dan mengakses data dengan cepat.

**Kegunaan:**
- Menyimpan user accounts dengan key berupa user ID
- Menyimpan produk dengan key berupa product ID
- Fast lookup untuk validasi transaksi

### 3. Tree (Merkle Tree)
Struktur tree khusus yang digunakan dalam blockchain untuk memverifikasi integritas data.

**Kegunaan:**
- Verifikasi efisien terhadap transaksi dalam block
- Menyimpan hash transaksi secara hierarkis
- Memungkinkan proof-of-inclusion yang efisien

### 4. Queue
Digunakan untuk mengelola pending transactions.

**Kegunaan:**
- Transaction pool (mempool) untuk transaksi yang belum dikonfirmasi
- FIFO processing untuk fairness
- Buffer untuk transaksi sebelum mining

### 5. Priority Queue
Untuk mengelola transaksi berdasarkan prioritas (misalnya fee).

**Kegunaan:**
- Prioritas processing berdasarkan transaction fee
- Memaksimalkan incentive untuk miners
- Optimasi throughput sistem

### 6. Stack
Untuk implementasi undo/rollback operations.

**Kegunaan:**
- Rollback transaksi yang gagal
- Undo operations dalam smart contracts
- Tracking state changes

## Komponen Utama Sistem

### 1. Block
```cpp
class Block {
private:
    int index;
    string timestamp;
    vector<Transaction> transactions;
    string previousHash;
    string hash;
    int nonce;
    
public:
    Block(int idx, vector<Transaction> trans, string prevHash);
    string calculateHash();
    void mineBlock(int difficulty);
    bool isValid();
};
```

### 2. Blockchain
```cpp
class Blockchain {
private:
    vector<Block> chain;
    int difficulty;
    vector<Transaction> pendingTransactions;
    double miningReward;
    
public:
    Blockchain();
    Block createGenesisBlock();
    Block getLatestBlock();
    void minePendingTransactions(string minerAddress);
    void addTransaction(Transaction transaction);
    bool isChainValid();
    double getBalance(string address);
};
```

### 3. Transaction
```cpp
class Transaction {
private:
    string fromAddress;
    string toAddress;
    double amount;
    string timestamp;
    string productId;
    
public:
    Transaction(string from, string to, double amt, string prodId);
    string calculateHash();
    void signTransaction(string privateKey);
    bool isValid();
};
```

### 4. Product
```cpp
class Product {
private:
    string productId;
    string name;
    string description;
    double price;
    string sellerId;
    int quantity;
    vector<string> reviews;
    
public:
    Product(string id, string name, double price, string seller);
    void updateQuantity(int amount);
    void addReview(string review);
    bool isAvailable();
};
```

### 5. User
```cpp
class User {
private:
    string userId;
    string username;
    string publicKey;
    string privateKey;
    double balance;
    vector<string> purchaseHistory;
    
public:
    User(string id, string name);
    void generateKeys();
    bool makePayment(double amount);
    void receivePayment(double amount);
};
```

## Fitur-Fitur Sistem

### 1. Manajemen Produk
- Tambah produk baru
- Update informasi produk
- Hapus produk
- Cari produk
- Filter dan sorting produk

### 2. Manajemen User
- Registrasi user baru
- Login/logout
- Update profile
- View transaction history
- Manage wallet balance

### 3. Transaksi E-Commerce
- Browse produk
- Add to cart
- Checkout
- Payment processing menggunakan blockchain
- Order tracking

### 4. Blockchain Operations
- Mining blocks
- Validasi transaksi
- Verify blockchain integrity
- Consensus mechanism
- Smart contracts untuk escrow

### 5. Security Features
- Digital signatures untuk transaksi
- Hash-based integrity checking
- Proof of Work untuk mining
- Encryption untuk data sensitif

## Algoritma Kriptografi

### 1. SHA-256 Hashing
Digunakan untuk:
- Menghash block data
- Generate block hash
- Create transaction IDs
- Merkle tree construction

### 2. Digital Signatures
Menggunakan asymmetric cryptography:
- Private key untuk signing transactions
- Public key untuk verifikasi
- Memastikan authenticity dan non-repudiation

### 3. Proof of Work (PoW)
Mekanisme consensus untuk mining:
```cpp
void Block::mineBlock(int difficulty) {
    string target(difficulty, '0');
    
    while (hash.substr(0, difficulty) != target) {
        nonce++;
        hash = calculateHash();
    }
    
    cout << "Block mined: " << hash << endl;
}
```

## Arsitektur Sistem

```
┌─────────────────────────────────────────────────────────┐
│                   User Interface Layer                   │
│  (CLI/GUI untuk interaksi dengan sistem)                │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│              Application Logic Layer                     │
│  - Product Management                                    │
│  - User Management                                       │
│  - Transaction Processing                                │
│  - Order Management                                      │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│              Blockchain Layer                            │
│  - Block Creation & Mining                               │
│  - Transaction Validation                                │
│  - Chain Validation                                      │
│  - Consensus Mechanism                                   │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│              Data Structure Layer                        │
│  - Linked List (Chain)                                   │
│  - Hash Tables (Users, Products)                         │
│  - Queue (Transaction Pool)                              │
│  - Merkle Tree (Transaction Verification)                │
└─────────────────────────────────────────────────────────┘
```

## Flow Transaksi E-Commerce

1. **Browsing Produk**
   - User browse katalog produk
   - Filter berdasarkan kategori, harga, dll
   - View detail produk dan reviews

2. **Pembelian**
   - User menambahkan produk ke cart
   - Review order dan total harga
   - Confirm purchase

3. **Payment Processing**
   - Create transaction object
   - Sign transaction dengan private key user
   - Add transaction ke pending transactions pool

4. **Mining & Konfirmasi**
   - Miner mengambil pending transactions
   - Create new block dengan transactions
   - Mine block (solve PoW puzzle)
   - Add block ke blockchain

5. **Finalisasi**
   - Transaction dikonfirmasi
   - Update product quantity
   - Update user balances
   - Generate receipt/invoice

## Implementasi Smart Contract Sederhana

```cpp
class Escrow {
private:
    string buyerAddress;
    string sellerAddress;
    double amount;
    bool buyerConfirmed;
    bool sellerConfirmed;
    
public:
    Escrow(string buyer, string seller, double amt) {
        buyerAddress = buyer;
        sellerAddress = seller;
        amount = amt;
        buyerConfirmed = false;
        sellerConfirmed = false;
    }
    
    void confirmDelivery() {
        buyerConfirmed = true;
        if (canRelease()) {
            releaseFunds();
        }
    }
    
    void confirmShipment() {
        sellerConfirmed = true;
    }
    
    bool canRelease() {
        return buyerConfirmed && sellerConfirmed;
    }
    
    void releaseFunds() {
        // Transfer funds from escrow to seller
        cout << "Releasing " << amount << " to " << sellerAddress << endl;
    }
};
```

## Keamanan dan Validasi

### Validasi Transaksi
```cpp
bool Transaction::isValid() {
    // Check if transaction is signed
    if (signature.empty()) return false;
    
    // Verify signature
    if (!verifySignature()) return false;
    
    // Check if amount is positive
    if (amount <= 0) return false;
    
    // Check if sender has sufficient balance
    if (!hasSufficientBalance()) return false;
    
    return true;
}
```

### Validasi Blockchain
```cpp
bool Blockchain::isChainValid() {
    for (int i = 1; i < chain.size(); i++) {
        Block currentBlock = chain[i];
        Block previousBlock = chain[i - 1];
        
        // Verify current block hash
        if (currentBlock.getHash() != currentBlock.calculateHash()) {
            return false;
        }
        
        // Verify link to previous block
        if (currentBlock.getPreviousHash() != previousBlock.getHash()) {
            return false;
        }
        
        // Verify all transactions in block
        for (auto& tx : currentBlock.getTransactions()) {
            if (!tx.isValid()) return false;
        }
    }
    return true;
}
```

## Optimasi dan Performance

### 1. Indexing
- Menggunakan hash tables untuk O(1) lookup
- Index produk berdasarkan kategori
- Cache frequently accessed data

### 2. Parallel Processing
- Mining dapat dilakukan secara parallel
- Transaction validation dapat dipercepat
- Multiple miners dapat bekerja simultaneously

### 3. Memory Management
- Efficient memory allocation
- Proper cleanup dan garbage collection
- Avoid memory leaks dengan RAII

### 4. Database Integration
- Persist blockchain data ke disk
- Use efficient serialization
- Implement checkpointing untuk fast recovery

## Testing dan Validasi

### Unit Tests
- Test individual components (Block, Transaction, dll)
- Verify hashing functions
- Test data structure operations

### Integration Tests
- Test interaction antar komponen
- Verify end-to-end transaction flow
- Test blockchain synchronization

### Security Tests
- Test untuk double-spending attacks
- Verify signature validation
- Test chain tampering detection

## Pengembangan Lebih Lanjut

### Future Enhancements
1. **Implementasi P2P Network**
   - Node discovery
   - Data synchronization
   - Distributed consensus

2. **Advanced Smart Contracts**
   - Conditional payments
   - Multi-signature transactions
   - Time-locked contracts

3. **Scalability Improvements**
   - Sharding implementation
   - Layer 2 solutions
   - Off-chain transactions

4. **Enhanced Security**
   - Multi-factor authentication
   - Advanced encryption
   - Anomaly detection

5. **User Experience**
   - Web interface
   - Mobile application
   - Real-time notifications

## Kesimpulan

Project ini mendemonstrasikan implementasi konsep blockchain dalam konteks e-commerce menggunakan C++. Dengan memanfaatkan berbagai struktur data seperti linked list, hash table, queue, tree, dan lainnya, sistem ini mampu menyediakan platform e-commerce yang:

- **Terdesentralisasi**: Tidak bergantung pada single authority
- **Transparan**: Semua transaksi tercatat dan dapat diaudit
- **Aman**: Menggunakan kriptografi untuk keamanan
- **Efficient**: Struktur data optimal untuk performance

Project ini juga menunjukkan penerapan praktis dari konsep-konsep struktur data yang dipelajari dalam mata kuliah, serta memberikan pemahaman tentang teknologi blockchain yang sedang berkembang.

## Referensi

1. Nakamoto, S. (2008). Bitcoin: A Peer-to-Peer Electronic Cash System
2. Antonopoulos, A. M. (2017). Mastering Bitcoin: Programming the Open Blockchain
3. Narayanan, A., et al. (2016). Bitcoin and Cryptocurrency Technologies
4. Cormen, T. H., et al. (2009). Introduction to Algorithms
5. Stroustrup, B. (2013). The C++ Programming Language

## Kontributor

- Tim Pengembang Struktur Data
- Dosen Pembimbing: [Nama Dosen]
- Institusi: [Nama Institusi]

## Lisensi

Project ini dibuat untuk keperluan akademis dalam rangka tugas besar mata kuliah Struktur Data.
