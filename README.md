# 🚩 The Cipher Breach — Cryptography CTF Challenge

Welcome to **The Cipher Breach**, a custom 3-stage cryptographic and forensic Capture The Flag (CTF) challenge themed around *Stranger Things*. This repository contains the framework architectures used to generate the challenge artifacts alongside programmatic solvers capable of extracting each component of the flag.

**Final Flag format:** `questCON{part1-part2-part3}`

---

## 🌌 Challenge Breakdown & Mechanics

### 🕵️ Stage 1: Mindflayer Logs
* **Core Vulnerability:** ECDSA Partial Nonce Leakage.
* **Context:** The system signs transactional messages using an Elliptic Curve Digital Signature Algorithm, but leaks the upper 12 bits of the secret nonce ($k$) inside `mindflayer_logs.json`.
* **Exploitation Theory:** Because the safety of ECDSA relies entirely on the complete entropy of the nonce, an attacker can relate two signatures mathematically. By guessing the missing bits of a nonce, a candidate private key $d$ can be derived and verified against subsequent leak profiles to fully recover the underlying private key.

### 🧪 Stage 2: Eleven's Randomness
* **Core Vulnerability:** MT19937 PRNG Predictability.
* **Context:** 624 sequential numbers are output from Python's default `random` engine into a binary payload (`elevens_numbers.bin`). The next consecutive four values are secretly packed to construct a 128-bit AES key.
* **Exploitation Theory:** The Mersenne Twister algorithm is not cryptographically secure. Since its outputs are derived via bit-shift operations and bitwise masks applied to its internal state (known as "tempering"), an attacker can pass 624 integers through an **untempering filter** to reverse-engineer the entire internal array and clone the PRNG to predict the future key bytes.

### 📻 Stage 3: Intercepted Signals
* **Core Vulnerability:** AES-CTR Many-Time Pad (MTP) Reused Nonce & LSB Steganography.
* **Context:** Radio signals are intercepted inside `intercepted_signals/`. Multiple distinct plaintexts are encrypted using identical key and nonce configurations in AES-CTR mode. Additionally, a metadata hint (`hint_image.png`) contains one of the original messages hidden within its pixel arrays.
* **Exploitation Theory:** 1. **LSB Extraction:** The red channel's Least Significant Bit (LSB) of the hint image is scraped to extract the known initialization string: `From: Dr. Brenner <mbrenner@hkinslab.gov>`.
    2. **MTP Cryptanalysis:** Reusing a key/counter combination reduces a stream cipher to a classic running-key problem where $C_1 \oplus C_2 = P_1 \oplus P_2$. By applying the discovered known plaintext to the corresponding ciphertext, the raw keystream is isolated, stripping the encryption layer from the surrounding signals.

---

## 📁 Repository Structure

```text
├── The Cipher Breach.pdf      # Complete official challenge writeup document
├── verify_flag.py             # SHA-256 validation engine for final flag inputs
├── stage1/
│   ├── stage1_generator.py    # Simulates ECDSA infrastructure & logs leaks
│   └── stage1_solver.py       # Reconstructs private key via bit brute-forcing
├── stage2/
│   ├── stage2_generator.py    # Generates PRNG sequences & encrypts payloads
│   └── stage2_solver.py       # Implements MT19937 untempering and decryption
└── stage3/
    ├── stage3_generator.py    # Orchestrates MTP transmissions & LSB encoding
    └── stage3_solver.py       # Decodes stego image & runs XOR stream recovery
