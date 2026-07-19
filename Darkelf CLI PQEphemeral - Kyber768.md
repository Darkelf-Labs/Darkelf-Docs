# 🔐 Darkelf CLI PQEphemeral — Kyber768 Encrypted Messaging

This is a command-line secure messaging tool using **Kyber768**, a post-quantum key encapsulation mechanism from the [Open Quantum Safe](https://openquantumsafe.org) project. It is based on `liboqs-python 0.13.0`, providing ephemeral encryption for message confidentiality.

---

## ✨ Features

- Post-quantum cryptography using **Kyber768**
- Ephemeral key exchange
- `genkeys`, `sendmsg`, `recvmsg` CLI commands
- AES-GCM authenticated encryption
- Message padding and compression
- Works fully offline
- Minimal and clean interface

---

## 🛠 Requirements

- macOS or Linux
- Python 3.11
- CMake and build tools
- OpenSSL >= 1.1.1
- Virtual environment (recommended)

---

## 📦 Setup Guide

### 1. Clone and Build `liboqs 0.13.0`

```bash
git clone --branch main https://github.com/open-quantum-safe/liboqs.git ~/liboqs
cd ~/liboqs
git checkout 0.13.0

mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..
make -j$(sysctl -n hw.logicalcpu)
sudo make install
```

---

### 2. Create Python Virtual Environment & Install `liboqs-python 0.13.0`

```bash
# Create a new virtual environment
python3.11 -m venv ~/pqcrypto_env
source ~/pqcrypto_env/bin/activate

# Reinstall liboqs-python 0.13.0
pip install --upgrade pip
pip install --force-reinstall --no-cache-dir liboqs-python==0.13.0
```

---

### 3. Confirm Installation

```bash
python -c "import oqs; print(oqs.__version__); print(oqs.get_enabled_KEMs())"
```

Expected output:

```
0.13.0
['Kyber768', ...]
```

---

## 🧪 Usage

```bash
python3 Darkelf_CLI_PQEphemeral_Kyber768.py
```

Then interact with the REPL:

```text
darkelf> genkeys
darkelf> sendmsg
Recipient pubkey path: /Users/.../my_pubkey.bin
Message: Hello world!
darkelf> recvmsg
📥 Message decrypted: Hello world!
```

---

## 🧠 Notes

- `liboqs` must be compiled and installed **before** installing `liboqs-python`.
- Only **Kyber768** is used — no `ML-KEM`, no hybrid modes.
- All messages are encrypted with a derived AES key from the post-quantum shared secret.

---

## 📜 License

MIT License — (c) 2024–2025, Darkelf Research
