# Cryptographic Foundations of Bitcoin

Bitcoin is based on complex cryptographic principles developed over recent decades [1]. Essentially, it uses irreversible (one-way) mathematical functions that are easy to compute in one direction but infeasible to reverse. These tools enable the creation of **digital secrets** and **unforgeable digital signatures**, which are the pillars of blockchain security. Specifically, Bitcoin employs public-key cryptography: each user has a pair of mathematically related keys (a private key and a derived public key). The public key is shared openly to receive transactions, while the private key, kept secret, is used to sign transactions and spend the associated funds [2][3].

---

## Fundamentals of Cryptography

**Asymmetric cryptography** (public key) is the foundation of Bitcoin. Its essence lies in the fact that from a very large private key (a random number with hundreds of digits), a public key is derived in a one-way manner [1][2]. This is done through operations on elliptic curves: given a generator point \(G\) and the private key \(k\), the public key is obtained as \(K = k \cdot G\), an operation easy to perform but whose inverse (the discrete logarithm problem) is computationally infeasible [4]. Thanks to this relationship, whoever possesses the private key can **sign** messages and transactions; anyone can verify these signatures with the corresponding public key without knowing the private key [4][5].

**Public and private keys.** Practically, a **public key** is widely distributed (e.g., to generate receiving addresses), while the **private key** is kept securely. The private key is the “root of control” over the funds: whoever knows it can create valid signatures to move the corresponding bitcoins. Therefore, safeguarding the private key is the user’s essential responsibility: losing it means losing access; revealing it means giving control to others. This principle is often summarized in the community as: **“Not your keys, not your coins”** [5][6].

---

## Hash Functions and Hash Pointers

Cryptographic **hash functions** are another cornerstone of Bitcoin. A hash function takes arbitrary-sized data (e.g., transaction details) and compresses it into a fixed-length bit string (the *hash*), such that even a minimal change in the original data results in a completely different hash. Bitcoin uses SHA-256 (double SHA-256 for block headers) and RIPEMD-160 in some address construction steps; these functions practically guarantee integrity and collision resistance [7].

Bitcoin combines hash functions with **hash pointers**: each block contains the hash of the previous block. A hash pointer is a pointer that, besides referencing an object, stores the hash of the pointed object. Thus, blocks become chained; modifying an earlier block changes its hash and breaks the pointer in the subsequent block, forcing recalculation of all following blocks to hide the alteration. This chain dependency makes the structure tamper-resistant if the chain is protected by proof-of-work (PoW) and honest majority computing power [8].

*(Suggestion: here you could include a simple diagram showing two or three blocks linked by hash pointers illustrating how a change breaks the chain.)*

---

## The Role of Ledgers (Blockchain) and Timestamping

The **blockchain** acts as a public, distributed ledger: it records transactions in blocks chained by hash pointers and distributed among participating nodes. Each block carries a *timestamp* field that approximately and chronologically situates the block creation. Although the timestamp is not perfectly precise (miners’ clocks may vary), it serves to order and make the history consistent and prevents simple reordering attacks [9].

Timestamping and the linked structure allow the network to establish a verifiable transaction history: rewriting an old transaction would require redoing the work of all subsequent blocks (a computationally very costly task if the chain is secured), which impedes double spending and retrospective censorship [9][12].

---

## Merkle Trees and Data Integrity

Within each block, Bitcoin uses a **Merkle tree** to summarize all transactions into a single root hash (Merkle root). In a Merkle tree, the leaves are the transaction hashes; internal nodes contain the hash of the concatenation of their children’s hashes; and the root summarizes the entire set. If one transaction changes, the effect propagates to the root, which will no longer match [3][10].

This structure has two practical advantages: (1) it allows verifying the inclusion of a particular transaction in a block without downloading the entire block (useful for SPV nodes), and (2) it reduces the cryptographic work needed to calculate and verify the integrity of the transaction set since the root condenses the integrity of the whole block [10].

*(Suggestion: a diagram of a Merkle tree with 4 leaves and the verification path of a transaction greatly aids understanding.)*

---

## Digital Signatures and Their Role in Bitcoin

**Digital signatures** allow authenticating transactions without revealing the private key. In Bitcoin, the sender signs the transaction data (or a hash thereof) with their private key; any verifier can check the signature with the public key and confirm the transaction was authorized by the private key holder [4][5].

Signatures provide two guarantees: **authentication** (the transaction comes from who it claims) and **integrity** (the data was not altered). In practice, signature validation is a central step in transaction verification by nodes: transaction inputs must contain valid signatures proving the right to spend the referenced outputs [5][11].

*(Suggestion: a flow diagram “transaction → hash → sign with private key → verify with public key” clarifies this process well.)*

---

## Elliptic Curve ECDSA and secp256k1 — and Schnorr Signatures?

Bitcoin uses the **ECDSA** (Elliptic Curve Digital Signature Algorithm) over the **secp256k1** curve to generate and verify digital signatures. Elliptic curves enable smaller keys with high security, and secp256k1 was chosen by Bitcoin’s original design [4][14].

ECDSA produces compact signatures (pairs \(r, s\)) and its security is based on the hardness of the elliptic curve discrete logarithm problem. In 2021, with the activation of **Taproot**, Bitcoin added support for **Schnorr** signatures (BIP-340/BIP-341), which offer benefits: signature aggregation (better privacy and scalability in multisig transactions) and useful algebraic properties for new cryptographic constructions [15]. Schnorr is also mathematically simpler and more elegant than ECDSA.

---

## Public-Key Cryptography: Practical Operation

Practically speaking:

- The user generates a **private key** (a secure random number).  
- From it, the **public key** is computed via an elliptic curve operation.  
- The public key (or a derived address) is shared to receive funds.  
- To spend, the user signs the transaction with their private key; the signature and public key (or address) travel in the transaction so nodes can verify validity.

This design enables decentralization: the network does not need to trust any entity to authorize payments; verifying the signature against the claimed public key suffices [2][4][11].

---

## Practical Principle: “Not Your Keys, Not Your Coins”

All the logical and mathematical security described converges in a simple operational conclusion: **custody of private keys is the foundation of ownership in Bitcoin**. Although the ledger records balances linked to addresses, that record only allows *viewing* balances; only whoever controls the private key can create valid signatures to transfer those balances. Losing the key means losing access; giving it away means surrendering funds. This principle explains why backup and custody (hardware wallets, multisig, BIP32-HD wallets) are emphasized as pillars of operational security [5][16].

---

## Conclusion

Bitcoin’s cryptographic foundations combine: (a) hash functions and hash pointers that guarantee integrity and linkage; (b) Merkle trees that summarize and allow efficient transaction verification; (c) digital signatures on elliptic curves providing authenticity and control; and (d) timestamping and proof-of-work consensus making the transaction history tamper-resistant. Together, these elements create a system where value ownership is determined by control of private keys and the chain’s protection by computational work. Understanding these components is essential for any user who wants to assess Bitcoin’s security, risks, and custody practices.

---

## References

1. Diffie, W. & Hellman, M. (1976). *New Directions in Cryptography*. IEEE Transactions on Information Theory, 22(6), 644–654.  
2. Rivest, R., Shamir, A., & Adleman, L. (1978). *A Method for Obtaining Digital Signatures and Public-Key Cryptosystems*. Communications of the ACM, 21(2), 120–126.  
3. Merkle, R. C. (1980). *Protocols for Public Key Cryptosystems*. Proceedings of the IEEE Symposium on Security and Privacy.  
4. Chaum, D. (1983). *Blind Signatures for Untraceable Payments*. Advances in Cryptology — CRYPTO '82 Proceedings, 199–203.  
5. Nakamoto, S. (2008). *Bitcoin: A Peer-to-Peer Electronic Cash System*. https://bitcoin.org/bitcoin.pdf  
6. Key custody considerations and “your keys, your coins” practices — security documentation and best practices from sources like bitcoin.org.  
7. NIST (FIPS 180-4). *Secure Hash Standard (SHS)* — SHA-256 specification.  
8. Back, A. (1997). *Hashcash — A Denial of Service Counter-Measure*. https://www.hashcash.org/papers/hashcash.pdf  
9. Bitcoin Developer Documentation: timestamping and block structure (https://developer.bitcoin.org / https://bitcoin.org).  
10. Merkle tree: technical descriptions and blockchain applications — see Merkle (1980) and related literature.  
11. Signature operation and verification in Bitcoin (ECDSA): technical and operational descriptions — see whitepaper and Bitcoin Core docs.  
12. Security against rewriting and cost of redoing PoW: explanations in Satoshi’s whitepaper and subsequent analyses.  
13. Technical documentation for secp256k1 curve — SECG, SEC2; Bitcoin Core uses libsecp256k1.  
14. BIP-340 / BIP-341 (Schnorr signatures and Taproot).  
15. Educational resources and advanced guides (Bitcoin Core, technical books, and cryptography research papers).  

