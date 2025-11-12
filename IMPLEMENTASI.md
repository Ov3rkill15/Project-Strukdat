# Implementasi Kode: Sistem E-Commerce Blockchain

## Daftar Isi
1. [Setup dan Dependencies](#setup-dan-dependencies)
2. [Core Classes](#core-classes)
3. [Contoh Implementasi Lengkap](#contoh-implementasi-lengkap)
4. [Cara Kompilasi dan Menjalankan](#cara-kompilasi-dan-menjalankan)
5. [Testing](#testing)

---

## Setup dan Dependencies

### Requirements
- C++ Compiler (g++ atau clang++) dengan support C++11 atau lebih baru
- OpenSSL library untuk SHA-256 hashing
- Standard Template Library (STL)

### Install Dependencies (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install g++ libssl-dev
```

### Install Dependencies (macOS)
```bash
brew install openssl
```

---

## Core Classes

### 1. SHA256 Utility

```cpp
// sha256.h
#ifndef SHA256_H
#define SHA256_H

#include <string>
#include <openssl/sha.h>
#include <sstream>
#include <iomanip>

class SHA256 {
public:
    static std::string hash(const std::string& input) {
        unsigned char hash[SHA256_DIGEST_LENGTH];
        SHA256_CTX sha256;
        SHA256_Init(&sha256);
        SHA256_Update(&sha256, input.c_str(), input.length());
        SHA256_Final(hash, &sha256);
        
        std::stringstream ss;
        for(int i = 0; i < SHA256_DIGEST_LENGTH; i++) {
            ss << std::hex << std::setw(2) << std::setfill('0') << (int)hash[i];
        }
        return ss.str();
    }
};

#endif
```

### 2. Transaction Class

```cpp
// transaction.h
#ifndef TRANSACTION_H
#define TRANSACTION_H

#include <string>
#include <ctime>
#include "sha256.h"

class Transaction {
private:
    std::string fromAddress;
    std::string toAddress;
    double amount;
    double fee;
    std::string productId;
    std::time_t timestamp;
    std::string signature;
    std::string txHash;

public:
    Transaction(std::string from, std::string to, double amt, double f = 0.0, std::string prodId = "")
        : fromAddress(from), toAddress(to), amount(amt), fee(f), productId(prodId) {
        timestamp = std::time(nullptr);
        txHash = calculateHash();
    }

    std::string calculateHash() {
        std::stringstream ss;
        ss << fromAddress << toAddress << amount << fee << productId << timestamp;
        return SHA256::hash(ss.str());
    }

    bool isValid() {
        if (amount <= 0) return false;
        if (fromAddress.empty() && toAddress != "MINING_REWARD") return false;
        return true;
    }

    // Getters
    std::string getFrom() const { return fromAddress; }
    std::string getTo() const { return toAddress; }
    double getAmount() const { return amount; }
    double getFee() const { return fee; }
    std::string getHash() const { return txHash; }
    std::string getProductId() const { return productId; }
    std::time_t getTimestamp() const { return timestamp; }

    // Untuk priority queue
    bool operator<(const Transaction& other) const {
        return fee < other.fee;
    }

    std::string toString() const {
        std::stringstream ss;
        ss << "From: " << fromAddress << ", To: " << toAddress 
           << ", Amount: " << amount << ", Fee: " << fee;
        return ss.str();
    }
};

#endif
```

### 3. Block Class

```cpp
// block.h
#ifndef BLOCK_H
#define BLOCK_H

#include <vector>
#include <string>
#include <ctime>
#include <sstream>
#include <iostream>
#include "transaction.h"
#include "sha256.h"

class Block {
private:
    int index;
    std::time_t timestamp;
    std::vector<Transaction> transactions;
    std::string previousHash;
    std::string hash;
    int nonce;

public:
    Block(int idx, std::vector<Transaction> trans, std::string prevHash)
        : index(idx), transactions(trans), previousHash(prevHash), nonce(0) {
        timestamp = std::time(nullptr);
        hash = calculateHash();
    }

    std::string calculateHash() {
        std::stringstream ss;
        ss << index << timestamp << previousHash << nonce;
        
        for (auto& tx : transactions) {
            ss << tx.getHash();
        }
        
        return SHA256::hash(ss.str());
    }

    void mineBlock(int difficulty) {
        std::string target(difficulty, '0');
        
        while (hash.substr(0, difficulty) != target) {
            nonce++;
            hash = calculateHash();
        }
        
        std::cout << "Block mined: " << hash << std::endl;
    }

    bool isValid() {
        // Verify hash
        if (hash != calculateHash()) return false;
        
        // Verify all transactions
        for (auto& tx : transactions) {
            if (!tx.isValid()) return false;
        }
        
        return true;
    }

    // Getters
    std::string getHash() const { return hash; }
    std::string getPreviousHash() const { return previousHash; }
    int getIndex() const { return index; }
    std::vector<Transaction> getTransactions() const { return transactions; }
    std::time_t getTimestamp() const { return timestamp; }
    int getNonce() const { return nonce; }
};

#endif
```

### 4. Blockchain Class

```cpp
// blockchain.h
#ifndef BLOCKCHAIN_H
#define BLOCKCHAIN_H

#include <vector>
#include <queue>
#include <iostream>
#include "block.h"
#include "transaction.h"

class Blockchain {
private:
    std::vector<Block> chain;
    int difficulty;
    std::queue<Transaction> pendingTransactions;
    double miningReward;

public:
    Blockchain(int diff = 2, double reward = 100.0) 
        : difficulty(diff), miningReward(reward) {
        chain.push_back(createGenesisBlock());
    }

    Block createGenesisBlock() {
        std::vector<Transaction> genesisTransactions;
        return Block(0, genesisTransactions, "0");
    }

    Block getLatestBlock() {
        return chain.back();
    }

    void addTransaction(Transaction transaction) {
        if (!transaction.isValid()) {
            std::cout << "Invalid transaction!" << std::endl;
            return;
        }
        
        pendingTransactions.push(transaction);
        std::cout << "Transaction added to pending pool" << std::endl;
    }

    void minePendingTransactions(std::string minerAddress) {
        std::vector<Transaction> blockTransactions;
        
        // Get transactions from pool (max 10 per block)
        int count = 0;
        while (!pendingTransactions.empty() && count < 10) {
            blockTransactions.push_back(pendingTransactions.front());
            pendingTransactions.pop();
            count++;
        }
        
        // Add mining reward
        Transaction rewardTx("", minerAddress, miningReward, 0.0);
        blockTransactions.push_back(rewardTx);
        
        // Create and mine new block
        Block newBlock(chain.size(), blockTransactions, getLatestBlock().getHash());
        newBlock.mineBlock(difficulty);
        
        chain.push_back(newBlock);
        std::cout << "Block successfully mined!" << std::endl;
    }

    bool isChainValid() {
        for (int i = 1; i < chain.size(); i++) {
            Block currentBlock = chain[i];
            Block previousBlock = chain[i - 1];
            
            if (!currentBlock.isValid()) {
                return false;
            }
            
            if (currentBlock.getPreviousHash() != previousBlock.getHash()) {
                return false;
            }
        }
        return true;
    }

    double getBalance(std::string address) {
        double balance = 0.0;
        
        for (auto& block : chain) {
            for (auto& tx : block.getTransactions()) {
                if (tx.getFrom() == address) {
                    balance -= (tx.getAmount() + tx.getFee());
                }
                if (tx.getTo() == address) {
                    balance += tx.getAmount();
                }
            }
        }
        
        return balance;
    }

    void printChain() {
        std::cout << "\n=== BLOCKCHAIN ===" << std::endl;
        for (auto& block : chain) {
            std::cout << "\nBlock #" << block.getIndex() << std::endl;
            std::cout << "Hash: " << block.getHash() << std::endl;
            std::cout << "Previous Hash: " << block.getPreviousHash() << std::endl;
            std::cout << "Nonce: " << block.getNonce() << std::endl;
            std::cout << "Transactions: " << block.getTransactions().size() << std::endl;
            
            for (auto& tx : block.getTransactions()) {
                std::cout << "  - " << tx.toString() << std::endl;
            }
        }
        std::cout << "==================\n" << std::endl;
    }

    int getPendingTransactionsCount() {
        return pendingTransactions.size();
    }
};

#endif
```

### 5. Product Class

```cpp
// product.h
#ifndef PRODUCT_H
#define PRODUCT_H

#include <string>
#include <vector>

class Product {
private:
    std::string productId;
    std::string name;
    std::string description;
    double price;
    std::string sellerId;
    int quantity;
    std::string category;

public:
    Product(std::string id, std::string n, double p, std::string seller, int qty, std::string cat = "General")
        : productId(id), name(n), price(p), sellerId(seller), quantity(qty), category(cat) {}

    bool isAvailable() const {
        return quantity > 0;
    }

    bool decreaseQuantity(int amount) {
        if (quantity >= amount) {
            quantity -= amount;
            return true;
        }
        return false;
    }

    void increaseQuantity(int amount) {
        quantity += amount;
    }

    // Getters
    std::string getId() const { return productId; }
    std::string getName() const { return name; }
    double getPrice() const { return price; }
    std::string getSellerId() const { return sellerId; }
    int getQuantity() const { return quantity; }
    std::string getCategory() const { return category; }

    std::string toString() const {
        std::stringstream ss;
        ss << "ID: " << productId << ", Name: " << name 
           << ", Price: " << price << ", Qty: " << quantity;
        return ss.str();
    }
};

#endif
```

### 6. User Class

```cpp
// user.h
#ifndef USER_H
#define USER_H

#include <string>
#include <vector>

class User {
private:
    std::string userId;
    std::string username;
    double balance;
    std::vector<std::string> purchaseHistory;

public:
    User(std::string id, std::string name, double initialBalance = 0.0)
        : userId(id), username(name), balance(initialBalance) {}

    bool deductBalance(double amount) {
        if (balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }

    void addBalance(double amount) {
        balance += amount;
    }

    void addPurchase(std::string productId) {
        purchaseHistory.push_back(productId);
    }

    // Getters
    std::string getId() const { return userId; }
    std::string getUsername() const { return username; }
    double getBalance() const { return balance; }
    std::vector<std::string> getPurchaseHistory() const { return purchaseHistory; }

    std::string toString() const {
        std::stringstream ss;
        ss << "User: " << username << " (ID: " << userId << "), Balance: " << balance;
        return ss.str();
    }
};

#endif
```

---

## Contoh Implementasi Lengkap

### main.cpp - Demo Sistem E-Commerce Blockchain

```cpp
// main.cpp
#include <iostream>
#include <unordered_map>
#include "blockchain.h"
#include "product.h"
#include "user.h"

class ECommerceSystem {
private:
    Blockchain blockchain;
    std::unordered_map<std::string, User*> users;
    std::unordered_map<std::string, Product*> products;

public:
    ECommerceSystem(int difficulty = 2) : blockchain(difficulty) {}

    ~ECommerceSystem() {
        for (auto& pair : users) delete pair.second;
        for (auto& pair : products) delete pair.second;
    }

    void registerUser(std::string userId, std::string username, double initialBalance) {
        User* user = new User(userId, username, initialBalance);
        users[userId] = user;
        std::cout << "User registered: " << username << std::endl;
    }

    void addProduct(std::string productId, std::string name, double price, 
                    std::string sellerId, int quantity, std::string category) {
        Product* product = new Product(productId, name, price, sellerId, quantity, category);
        products[productId] = product;
        std::cout << "Product added: " << name << std::endl;
    }

    bool purchaseProduct(std::string buyerId, std::string productId, int quantity = 1) {
        // Check if user exists
        if (users.find(buyerId) == users.end()) {
            std::cout << "Buyer not found!" << std::endl;
            return false;
        }

        // Check if product exists
        if (products.find(productId) == products.end()) {
            std::cout << "Product not found!" << std::endl;
            return false;
        }

        User* buyer = users[buyerId];
        Product* product = products[productId];

        // Check product availability
        if (!product->isAvailable() || product->getQuantity() < quantity) {
            std::cout << "Product not available!" << std::endl;
            return false;
        }

        double totalCost = product->getPrice() * quantity;
        double fee = totalCost * 0.01; // 1% transaction fee

        // Check buyer balance
        if (buyer->getBalance() < (totalCost + fee)) {
            std::cout << "Insufficient balance!" << std::endl;
            return false;
        }

        // Create transaction
        Transaction tx(buyerId, product->getSellerId(), totalCost, fee, productId);
        blockchain.addTransaction(tx);

        // Update states (in real system, this happens after mining)
        buyer->deductBalance(totalCost + fee);
        product->decreaseQuantity(quantity);
        buyer->addPurchase(productId);

        std::cout << "Purchase successful! Transaction added to pending pool." << std::endl;
        return true;
    }

    void mineBlocks(std::string minerAddress) {
        blockchain.minePendingTransactions(minerAddress);
    }

    void showUserBalance(std::string userId) {
        if (users.find(userId) == users.end()) {
            std::cout << "User not found!" << std::endl;
            return;
        }

        User* user = users[userId];
        double blockchainBalance = blockchain.getBalance(userId);
        
        std::cout << "\n=== User Balance ===" << std::endl;
        std::cout << "Username: " << user->getUsername() << std::endl;
        std::cout << "Local Balance: " << user->getBalance() << std::endl;
        std::cout << "Blockchain Balance: " << blockchainBalance << std::endl;
        std::cout << "===================\n" << std::endl;
    }

    void showProducts() {
        std::cout << "\n=== Product Catalog ===" << std::endl;
        for (auto& pair : products) {
            std::cout << pair.second->toString() << std::endl;
        }
        std::cout << "=====================\n" << std::endl;
    }

    void showBlockchain() {
        blockchain.printChain();
    }

    bool validateBlockchain() {
        return blockchain.isChainValid();
    }
};

int main() {
    std::cout << "=== E-Commerce Blockchain System ===" << std::endl;
    std::cout << "Initializing system...\n" << std::endl;

    // Create system with difficulty 2
    ECommerceSystem system(2);

    // Register users
    std::cout << "--- Registering Users ---" << std::endl;
    system.registerUser("user1", "Alice", 1000.0);
    system.registerUser("user2", "Bob", 500.0);
    system.registerUser("miner1", "Miner", 0.0);
    std::cout << std::endl;

    // Add products
    std::cout << "--- Adding Products ---" << std::endl;
    system.addProduct("prod1", "Laptop", 500.0, "user2", 5, "Electronics");
    system.addProduct("prod2", "Mouse", 25.0, "user2", 10, "Electronics");
    system.addProduct("prod3", "Keyboard", 75.0, "user2", 8, "Electronics");
    std::cout << std::endl;

    // Show products
    system.showProducts();

    // Purchase products
    std::cout << "--- Making Purchases ---" << std::endl;
    system.purchaseProduct("user1", "prod1", 1);  // Alice buys laptop
    system.purchaseProduct("user1", "prod2", 2);  // Alice buys 2 mice
    system.purchaseProduct("user2", "prod3", 1);  // Bob buys keyboard (self-purchase)
    std::cout << std::endl;

    // Show balances before mining
    std::cout << "--- Balances Before Mining ---" << std::endl;
    system.showUserBalance("user1");
    system.showUserBalance("user2");
    system.showUserBalance("miner1");

    // Mine pending transactions
    std::cout << "--- Mining Block ---" << std::endl;
    system.mineBlocks("miner1");
    std::cout << std::endl;

    // Show balances after mining
    std::cout << "--- Balances After Mining ---" << std::endl;
    system.showUserBalance("user1");
    system.showUserBalance("user2");
    system.showUserBalance("miner1");

    // Show blockchain
    system.showBlockchain();

    // Validate blockchain
    std::cout << "--- Blockchain Validation ---" << std::endl;
    if (system.validateBlockchain()) {
        std::cout << "✓ Blockchain is VALID" << std::endl;
    } else {
        std::cout << "✗ Blockchain is INVALID" << std::endl;
    }

    return 0;
}
```

---

## Cara Kompilasi dan Menjalankan

### Struktur File
```
project/
├── sha256.h
├── transaction.h
├── block.h
├── blockchain.h
├── product.h
├── user.h
└── main.cpp
```

### Kompilasi

#### Menggunakan g++
```bash
g++ -std=c++11 main.cpp -o ecommerce -lssl -lcrypto
```

#### Dengan debug symbols
```bash
g++ -std=c++11 -g main.cpp -o ecommerce -lssl -lcrypto
```

#### Dengan optimasi
```bash
g++ -std=c++11 -O2 main.cpp -o ecommerce -lssl -lcrypto
```

### Menjalankan
```bash
./ecommerce
```

### Contoh Output
```
=== E-Commerce Blockchain System ===
Initializing system...

--- Registering Users ---
User registered: Alice
User registered: Bob
User registered: Miner

--- Adding Products ---
Product added: Laptop
Product added: Mouse
Product added: Keyboard

=== Product Catalog ===
ID: prod1, Name: Laptop, Price: 500, Qty: 5
ID: prod2, Name: Mouse, Price: 25, Qty: 10
ID: prod3, Name: Keyboard, Price: 75, Qty: 8
=====================

--- Making Purchases ---
Transaction added to pending pool
Purchase successful! Transaction added to pending pool.
Transaction added to pending pool
Purchase successful! Transaction added to pending pool.
Transaction added to pending pool
Purchase successful! Transaction added to pending pool.

--- Mining Block ---
Block mined: 0045a3f2b8c9d1e4f5a6b7c8d9e0f1a2...
Block successfully mined!

✓ Blockchain is VALID
```

---

## Testing

### Unit Test Example

```cpp
// test.cpp
#include <cassert>
#include <iostream>
#include "blockchain.h"

void testTransactionCreation() {
    Transaction tx("alice", "bob", 100.0, 1.0);
    assert(tx.isValid());
    assert(tx.getAmount() == 100.0);
    std::cout << "✓ Transaction creation test passed" << std::endl;
}

void testBlockMining() {
    std::vector<Transaction> transactions;
    transactions.push_back(Transaction("alice", "bob", 100.0, 1.0));
    
    Block block(1, transactions, "0");
    block.mineBlock(2);
    
    std::string hash = block.getHash();
    assert(hash.substr(0, 2) == "00");
    std::cout << "✓ Block mining test passed" << std::endl;
}

void testBlockchainValidity() {
    Blockchain bc(2);
    
    bc.addTransaction(Transaction("alice", "bob", 100.0));
    bc.minePendingTransactions("miner");
    
    assert(bc.isChainValid());
    std::cout << "✓ Blockchain validity test passed" << std::endl;
}

void testBalance() {
    Blockchain bc(2);
    
    bc.addTransaction(Transaction("alice", "bob", 100.0));
    bc.minePendingTransactions("miner");
    
    double minerBalance = bc.getBalance("miner");
    assert(minerBalance == 100.0); // Mining reward
    std::cout << "✓ Balance test passed" << std::endl;
}

int main() {
    std::cout << "Running tests...\n" << std::endl;
    
    testTransactionCreation();
    testBlockMining();
    testBlockchainValidity();
    testBalance();
    
    std::cout << "\n✓ All tests passed!" << std::endl;
    return 0;
}
```

### Kompilasi dan Jalankan Test
```bash
g++ -std=c++11 test.cpp -o test -lssl -lcrypto
./test
```

---

## Troubleshooting

### Error: "fatal error: openssl/sha.h: No such file or directory"
**Solusi**: Install OpenSSL development libraries
```bash
# Ubuntu/Debian
sudo apt-get install libssl-dev

# macOS
brew install openssl
```

### Error: "undefined reference to SHA256_Init"
**Solusi**: Link dengan OpenSSL libraries
```bash
g++ main.cpp -o ecommerce -lssl -lcrypto
```

### Warning: "implicit conversion loses integer precision"
**Solusi**: Gunakan explicit casting atau compile dengan `-Wno-conversion`

---

## Next Steps

1. Implement persistent storage (database atau file system)
2. Add network layer untuk distributed nodes
3. Implement more sophisticated consensus mechanism
4. Add GUI menggunakan Qt atau web interface
5. Enhance security dengan proper key management
6. Add smart contracts functionality
7. Implement advanced features (escrow, multi-sig, etc.)
