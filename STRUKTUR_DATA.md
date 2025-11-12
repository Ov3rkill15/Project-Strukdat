# Struktur Data dalam Sistem E-Commerce Blockchain

## Daftar Isi
1. [Linked List untuk Blockchain](#linked-list-untuk-blockchain)
2. [Hash Table untuk Data Management](#hash-table-untuk-data-management)
3. [Merkle Tree untuk Verifikasi](#merkle-tree-untuk-verifikasi)
4. [Queue untuk Transaction Pool](#queue-untuk-transaction-pool)
5. [Priority Queue untuk Fee-based Processing](#priority-queue-untuk-fee-based-processing)
6. [Stack untuk Rollback Operations](#stack-untuk-rollback-operations)

---

## Linked List untuk Blockchain

### Konsep
Blockchain pada dasarnya adalah linked list di mana setiap node (block) menyimpan:
- Data transaksi
- Timestamp
- Hash dari block sebelumnya
- Hash dari block saat ini

### Implementasi

```cpp
class Block {
private:
    int index;
    string timestamp;
    vector<Transaction> transactions;
    string previousHash;
    string hash;
    int nonce;
    Block* next; // Pointer ke block selanjutnya
    
public:
    Block(int idx, vector<Transaction> trans, string prevHash) {
        index = idx;
        timestamp = getCurrentTimestamp();
        transactions = trans;
        previousHash = prevHash;
        nonce = 0;
        next = nullptr;
        hash = calculateHash();
    }
    
    string calculateHash() {
        stringstream ss;
        ss << index << timestamp << previousHash << nonce;
        
        for (auto& tx : transactions) {
            ss << tx.toString();
        }
        
        return sha256(ss.str());
    }
    
    void setNext(Block* nextBlock) {
        next = nextBlock;
    }
    
    Block* getNext() {
        return next;
    }
};

class Blockchain {
private:
    Block* head; // Genesis block
    Block* tail; // Latest block
    int length;
    
public:
    Blockchain() {
        head = createGenesisBlock();
        tail = head;
        length = 1;
    }
    
    Block* createGenesisBlock() {
        vector<Transaction> genesisTransactions;
        Block* genesis = new Block(0, genesisTransactions, "0");
        return genesis;
    }
    
    void addBlock(Block* newBlock) {
        tail->setNext(newBlock);
        tail = newBlock;
        length++;
    }
    
    Block* getLatestBlock() {
        return tail;
    }
    
    // Traversal untuk validasi chain
    bool validateChain() {
        Block* current = head->getNext();
        Block* previous = head;
        
        while (current != nullptr) {
            // Verify hash
            if (current->getHash() != current->calculateHash()) {
                return false;
            }
            
            // Verify link
            if (current->getPreviousHash() != previous->getHash()) {
                return false;
            }
            
            previous = current;
            current = current->getNext();
        }
        
        return true;
    }
};
```

### Kompleksitas
- **Insert (add block)**: O(1) - menambahkan di tail
- **Search**: O(n) - harus traverse dari head
- **Validation**: O(n) - harus traverse seluruh chain

### Keuntungan
- Immutable: sekali block ditambahkan, sulit untuk diubah
- Chronological order: block tersusun berdasarkan waktu
- Tamper-evident: perubahan pada block lama akan terdeteksi

---

## Hash Table untuk Data Management

### Konsep
Hash table digunakan untuk menyimpan dan mengakses data dengan cepat menggunakan key-value pairs.

### Implementasi untuk User Management

```cpp
class UserManager {
private:
    unordered_map<string, User*> userTable; // userId -> User
    unordered_map<string, string> usernameToId; // username -> userId
    
public:
    void registerUser(string username, string userId) {
        if (userTable.find(userId) != userTable.end()) {
            throw runtime_error("User ID already exists");
        }
        
        User* newUser = new User(userId, username);
        userTable[userId] = newUser;
        usernameToId[username] = userId;
    }
    
    User* getUserById(string userId) {
        auto it = userTable.find(userId);
        if (it != userTable.end()) {
            return it->second;
        }
        return nullptr;
    }
    
    User* getUserByUsername(string username) {
        auto it = usernameToId.find(username);
        if (it != usernameToId.end()) {
            return getUserById(it->second);
        }
        return nullptr;
    }
    
    bool updateBalance(string userId, double amount) {
        User* user = getUserById(userId);
        if (user != nullptr) {
            user->updateBalance(amount);
            return true;
        }
        return false;
    }
};
```

### Implementasi untuk Product Management

```cpp
class ProductCatalog {
private:
    unordered_map<string, Product*> productTable; // productId -> Product
    unordered_map<string, vector<string>> categoryIndex; // category -> [productIds]
    
public:
    void addProduct(Product* product) {
        string productId = product->getId();
        productTable[productId] = product;
        
        string category = product->getCategory();
        categoryIndex[category].push_back(productId);
    }
    
    Product* getProduct(string productId) {
        auto it = productTable.find(productId);
        if (it != productTable.end()) {
            return it->second;
        }
        return nullptr;
    }
    
    vector<Product*> getProductsByCategory(string category) {
        vector<Product*> products;
        auto it = categoryIndex.find(category);
        
        if (it != categoryIndex.end()) {
            for (string& productId : it->second) {
                Product* product = getProduct(productId);
                if (product != nullptr) {
                    products.push_back(product);
                }
            }
        }
        
        return products;
    }
    
    bool updateProduct(string productId, Product* updatedProduct) {
        if (productTable.find(productId) != productTable.end()) {
            productTable[productId] = updatedProduct;
            return true;
        }
        return false;
    }
    
    bool deleteProduct(string productId) {
        auto it = productTable.find(productId);
        if (it != productTable.end()) {
            delete it->second;
            productTable.erase(it);
            return true;
        }
        return false;
    }
};
```

### Kompleksitas
- **Insert**: O(1) average case
- **Search**: O(1) average case
- **Delete**: O(1) average case
- **Update**: O(1) average case

### Keuntungan
- Fast lookup dan retrieval
- Efficient untuk large datasets
- Flexible key types

---

## Merkle Tree untuk Verifikasi

### Konsep
Merkle tree adalah binary tree di mana:
- Leaf nodes berisi hash dari data (transactions)
- Internal nodes berisi hash dari kombinasi child nodes
- Root node (Merkle root) merepresentasikan semua data

### Implementasi

```cpp
class MerkleNode {
public:
    string hash;
    MerkleNode* left;
    MerkleNode* right;
    
    MerkleNode(string h) : hash(h), left(nullptr), right(nullptr) {}
};

class MerkleTree {
private:
    MerkleNode* root;
    
    string combineHash(string left, string right) {
        return sha256(left + right);
    }
    
    MerkleNode* buildTree(vector<string>& hashes, int start, int end) {
        if (start == end) {
            return new MerkleNode(hashes[start]);
        }
        
        int mid = (start + end) / 2;
        MerkleNode* leftChild = buildTree(hashes, start, mid);
        MerkleNode* rightChild = buildTree(hashes, mid + 1, end);
        
        string combinedHash = combineHash(leftChild->hash, rightChild->hash);
        MerkleNode* node = new MerkleNode(combinedHash);
        node->left = leftChild;
        node->right = rightChild;
        
        return node;
    }
    
public:
    MerkleTree(vector<Transaction>& transactions) {
        vector<string> hashes;
        
        for (auto& tx : transactions) {
            hashes.push_back(tx.getHash());
        }
        
        // Pad dengan duplicate jika jumlah ganjil
        if (hashes.size() % 2 != 0) {
            hashes.push_back(hashes.back());
        }
        
        root = buildTree(hashes, 0, hashes.size() - 1);
    }
    
    string getRootHash() {
        return root ? root->hash : "";
    }
    
    // Verify bahwa transaction termasuk dalam tree
    bool verifyTransaction(string txHash, vector<string>& proof) {
        string currentHash = txHash;
        
        for (string& proofHash : proof) {
            currentHash = combineHash(currentHash, proofHash);
        }
        
        return currentHash == root->hash;
    }
    
    // Generate proof untuk transaction
    vector<string> generateProof(string txHash) {
        vector<string> proof;
        generateProofHelper(root, txHash, proof);
        return proof;
    }
    
private:
    bool generateProofHelper(MerkleNode* node, string targetHash, vector<string>& proof) {
        if (node == nullptr) return false;
        
        if (node->hash == targetHash) return true;
        
        if (node->left && generateProofHelper(node->left, targetHash, proof)) {
            if (node->right) proof.push_back(node->right->hash);
            return true;
        }
        
        if (node->right && generateProofHelper(node->right, targetHash, proof)) {
            if (node->left) proof.push_back(node->left->hash);
            return true;
        }
        
        return false;
    }
};
```

### Penggunaan dalam Block

```cpp
class Block {
private:
    vector<Transaction> transactions;
    string merkleRoot;
    
public:
    void calculateMerkleRoot() {
        MerkleTree tree(transactions);
        merkleRoot = tree.getRootHash();
    }
    
    bool verifyTransactionInBlock(Transaction& tx) {
        MerkleTree tree(transactions);
        vector<string> proof = tree.generateProof(tx.getHash());
        return tree.verifyTransaction(tx.getHash(), proof);
    }
};
```

### Kompleksitas
- **Build tree**: O(n) di mana n = jumlah transaksi
- **Verify transaction**: O(log n)
- **Generate proof**: O(log n)

### Keuntungan
- Efficient verification tanpa perlu semua data
- Compact proof size (O(log n))
- Tamper-proof: perubahan pada leaf akan mengubah root

---

## Queue untuk Transaction Pool

### Konsep
Queue (FIFO) digunakan untuk mengelola pending transactions yang menunggu untuk dimining.

### Implementasi

```cpp
class TransactionPool {
private:
    queue<Transaction> pendingTransactions;
    mutex poolMutex; // Thread safety
    int maxPoolSize;
    
public:
    TransactionPool(int maxSize = 1000) : maxPoolSize(maxSize) {}
    
    bool addTransaction(Transaction tx) {
        lock_guard<mutex> lock(poolMutex);
        
        if (pendingTransactions.size() >= maxPoolSize) {
            return false; // Pool is full
        }
        
        if (!tx.isValid()) {
            return false; // Invalid transaction
        }
        
        pendingTransactions.push(tx);
        return true;
    }
    
    vector<Transaction> getTransactionsForBlock(int maxTransactions) {
        lock_guard<mutex> lock(poolMutex);
        
        vector<Transaction> transactions;
        int count = 0;
        
        while (!pendingTransactions.empty() && count < maxTransactions) {
            transactions.push_back(pendingTransactions.front());
            pendingTransactions.pop();
            count++;
        }
        
        return transactions;
    }
    
    int getPoolSize() {
        lock_guard<mutex> lock(poolMutex);
        return pendingTransactions.size();
    }
    
    bool isEmpty() {
        lock_guard<mutex> lock(poolMutex);
        return pendingTransactions.empty();
    }
};
```

### Kompleksitas
- **Enqueue (add)**: O(1)
- **Dequeue (remove)**: O(1)
- **Peek**: O(1)

### Keuntungan
- Fair processing (FIFO)
- Simple implementation
- Predictable behavior

---

## Priority Queue untuk Fee-based Processing

### Konsep
Priority queue digunakan untuk memproses transaksi berdasarkan fee yang ditawarkan, memberikan incentive kepada users untuk membayar fee lebih tinggi.

### Implementasi

```cpp
class PriorityTransactionPool {
private:
    struct TransactionComparator {
        bool operator()(const Transaction& tx1, const Transaction& tx2) {
            // Higher fee = higher priority (min heap, so we reverse)
            return tx1.getFee() < tx2.getFee();
        }
    };
    
    priority_queue<Transaction, vector<Transaction>, TransactionComparator> pool;
    mutex poolMutex;
    int maxPoolSize;
    
public:
    PriorityTransactionPool(int maxSize = 1000) : maxPoolSize(maxSize) {}
    
    bool addTransaction(Transaction tx) {
        lock_guard<mutex> lock(poolMutex);
        
        if (pool.size() >= maxPoolSize) {
            // Jika pool penuh, hanya terima jika fee lebih tinggi dari minimum
            if (tx.getFee() <= pool.top().getFee()) {
                return false;
            }
            // Remove lowest fee transaction
            pool.pop();
        }
        
        if (!tx.isValid()) {
            return false;
        }
        
        pool.push(tx);
        return true;
    }
    
    vector<Transaction> getHighestFeeTransactions(int maxTransactions) {
        lock_guard<mutex> lock(poolMutex);
        
        vector<Transaction> transactions;
        priority_queue<Transaction, vector<Transaction>, TransactionComparator> tempPool = pool;
        
        while (!tempPool.empty() && transactions.size() < maxTransactions) {
            transactions.push_back(tempPool.top());
            tempPool.pop();
        }
        
        // Remove selected transactions from actual pool
        while (transactions.size() > 0 && !pool.empty()) {
            pool.pop();
        }
        
        return transactions;
    }
    
    Transaction peekHighestFee() {
        lock_guard<mutex> lock(poolMutex);
        if (!pool.empty()) {
            return pool.top();
        }
        throw runtime_error("Pool is empty");
    }
};
```

### Kompleksitas
- **Insert**: O(log n)
- **Extract max**: O(log n)
- **Peek max**: O(1)

### Keuntungan
- Maximize miner revenue
- Incentivize fee payment
- Better throughput untuk high-value transactions

---

## Stack untuk Rollback Operations

### Konsep
Stack (LIFO) digunakan untuk menyimpan state changes dan memungkinkan rollback jika terjadi error.

### Implementasi

```cpp
class StateChange {
public:
    enum Type { BALANCE_UPDATE, PRODUCT_UPDATE, TRANSACTION_ADD };
    
    Type type;
    string entityId;
    string previousState;
    string newState;
    
    StateChange(Type t, string id, string prev, string next) 
        : type(t), entityId(id), previousState(prev), newState(next) {}
};

class StateManager {
private:
    stack<StateChange> changeHistory;
    int maxHistorySize;
    
public:
    StateManager(int maxSize = 1000) : maxHistorySize(maxSize) {}
    
    void recordChange(StateChange change) {
        if (changeHistory.size() >= maxHistorySize) {
            // Remove oldest change (at bottom of stack)
            // This requires rebuilding stack
            stack<StateChange> temp;
            while (changeHistory.size() > 1) {
                temp.push(changeHistory.top());
                changeHistory.pop();
            }
            changeHistory.pop(); // Remove oldest
            
            while (!temp.empty()) {
                changeHistory.push(temp.top());
                temp.pop();
            }
        }
        
        changeHistory.push(change);
    }
    
    bool rollback(int steps = 1) {
        if (changeHistory.size() < steps) {
            return false;
        }
        
        for (int i = 0; i < steps; i++) {
            StateChange change = changeHistory.top();
            changeHistory.pop();
            
            // Apply reverse change
            applyReverseChange(change);
        }
        
        return true;
    }
    
private:
    void applyReverseChange(StateChange& change) {
        switch (change.type) {
            case StateChange::BALANCE_UPDATE:
                // Restore previous balance
                restoreBalance(change.entityId, change.previousState);
                break;
                
            case StateChange::PRODUCT_UPDATE:
                // Restore previous product state
                restoreProduct(change.entityId, change.previousState);
                break;
                
            case StateChange::TRANSACTION_ADD:
                // Remove transaction
                removeTransaction(change.entityId);
                break;
        }
    }
    
    void restoreBalance(string userId, string previousBalance) {
        // Implementation
    }
    
    void restoreProduct(string productId, string previousState) {
        // Implementation
    }
    
    void removeTransaction(string txId) {
        // Implementation
    }
};

// Usage dalam transaction processing
class TransactionProcessor {
private:
    StateManager stateManager;
    
public:
    bool processTransaction(Transaction& tx) {
        try {
            // Record state before changes
            User* sender = getUserById(tx.getFrom());
            User* receiver = getUserById(tx.getTo());
            
            double senderPrevBalance = sender->getBalance();
            double receiverPrevBalance = receiver->getBalance();
            
            // Make changes
            sender->deductBalance(tx.getAmount());
            stateManager.recordChange(
                StateChange(StateChange::BALANCE_UPDATE, 
                           tx.getFrom(), 
                           to_string(senderPrevBalance),
                           to_string(sender->getBalance()))
            );
            
            receiver->addBalance(tx.getAmount());
            stateManager.recordChange(
                StateChange(StateChange::BALANCE_UPDATE,
                           tx.getTo(),
                           to_string(receiverPrevBalance),
                           to_string(receiver->getBalance()))
            );
            
            return true;
            
        } catch (exception& e) {
            // Rollback on error
            stateManager.rollback(2); // Rollback both balance changes
            return false;
        }
    }
};
```

### Kompleksitas
- **Push**: O(1)
- **Pop**: O(1)
- **Peek**: O(1)

### Keuntungan
- Easy rollback mechanism
- LIFO nature cocok untuk undo operations
- Memory efficient

---

## Perbandingan dan Kesimpulan

| Struktur Data | Use Case | Kompleksitas Utama | Keuntungan Utama |
|---------------|----------|-------------------|------------------|
| Linked List | Blockchain chain | Insert: O(1), Search: O(n) | Immutable, chronological |
| Hash Table | User/Product storage | All ops: O(1) avg | Fast lookup |
| Merkle Tree | Transaction verification | Verify: O(log n) | Compact proofs |
| Queue | Transaction pool (FIFO) | All ops: O(1) | Fair processing |
| Priority Queue | Fee-based pool | Insert/Extract: O(log n) | Priority-based |
| Stack | Rollback operations | All ops: O(1) | Easy undo |

Setiap struktur data dipilih berdasarkan karakteristik dan kebutuhan spesifik dalam sistem e-commerce blockchain, memberikan kombinasi optimal antara performance, security, dan functionality.
