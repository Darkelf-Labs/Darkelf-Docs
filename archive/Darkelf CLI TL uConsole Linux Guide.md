# Darkelf CLI TL Edition

**Darkelf CLI TL Edition** is a secure, privacy-focused, post-quantum-ready command-line web browser and OSINT toolkit. It is designed for privacy researchers, journalists, and security professionals who need advanced anonymity, anti-forensics, and OSINT capabilities in a pure terminal environment. The project is cross-platform, with primary support for Linux (including ARM devices like uConsole), macOS, and Windows.

---

## Features

- **Complete CLI Browser** with Tor integration (via `stem` and `obfs4proxy`)
- **Post-Quantum Cryptography** (Kyber, via `py-oqs`)
- **Stealth & Anti-Forensics** (memory locking, swap detection, ephemeral logs)
- **Integrated OSINT Tools** (search, onion discovery, phishing detection, etc.)
- **Rich Terminal UI** (with `rich`)
- **Secure Messaging** (end-to-end encrypted using post-quantum keys)
- **Threat & Phishing Detection** (no LLM/network required)
- **Plug-and-Play with External Tools** (`nmap`, `sherlock`, `theHarvester`, etc.)

---

## Quickstart (uConsole / Linux)

### 1. **Update your system**

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. **Install system dependencies**

```bash
sudo apt install -y python3 python3-pip tor obfs4proxy git build-essential libssl-dev
```

### 3. **(Recommended) Set up a Python virtual environment**

```bash
python3 -m venv darkelf-env
source darkelf-env/bin/activate
```

### 4. **Install Python package dependencies**

```bash
pip install --upgrade pip
pip install stem requests beautifulsoup4 psutil rich cryptography pycryptodome pyopenssl tls-client py-oqs
```

### 5. **Download the script**

```bash
wget "https://raw.githubusercontent.com/Darkelf2024/Darkelf-Browser-v3-PQC/main/Darkelf%20CLI%20TL%20Edition.py" -O darkelf_cli_tl.py
```

### 6. **(Optional) Install extra OSINT tools**

```bash
sudo apt install -y nmap theharvester masscan mat2 exiftool amass
pip install yt-dlp gitleaks phoneinfoga
```

### 7. **Start Tor (in the background)**

```bash
tor &
```

### 8. **Run Darkelf CLI**

```bash
python3.11 darkelf_cli_tl.py
```

---

## Usage

You’ll see a prompt like:
```
darkelf>
```
Type `help` for a list of commands.

### Example commands:
- `search <keywords>` — Search DuckDuckGo (onion)
- `open <url>` — Open and fetch a full URL
- `findonions <keywords>` — Discover .onion services
- `genkeys` — Generate post-quantum keys
- `sendmsg` — Encrypt and send a message
- `recvmsg` — Decrypt and read a received message
- `osintscan <term|url>` — Extract emails, phones, etc.
- `tools` — List available terminal OSINT tools

---

## Dependencies

- **Python 3.11+**
- **Tor** (`tor`, `obfs4proxy`)
- **Python packages:**  
  `stem`, `requests`, `beautifulsoup4`, `psutil`, `rich`, `cryptography`, `pycryptodome`, `pyopenssl`, `tls-client`, `py-oqs`
- **Optional OSINT tools:**  
  `nmap`, `theharvester`, `masscan`, `mat2`, `exiftool`, `amass`, `yt-dlp`, `gitleaks`, `phoneinfoga`, `sherlock`, `recon-ng`, etc.

If `py-oqs` fails to install, try `sudo apt install liboqs-dev` and re-run pip.

---

## Security & Privacy Notes

- All logs are kept in memory and encrypted; logs are wiped on exit or signal.
- Swap and kernel state are monitored; sensitive data is never written to disk.
- Tor integration is default; most internet traffic will go through Tor.
- Post-quantum cryptography is used for messaging and some internal communications.

---

## License

This software is licensed under [LGPL-3.0-or-later](https://www.gnu.org/licenses/lgpl-3.0.html).

(c) 2025 Dr. Kevin Moore

---

## Troubleshooting

- **ModuleNotFoundError**: Make sure you've activated your virtualenv and installed all pip packages.
- **py-oqs install errors**: Install `liboqs-dev` via your package manager and try again.
- **Tor not connecting**: Make sure Tor is running in the background (`tor &`).

If you have any issues, please open an issue on the [GitHub repository](https://github.com/Darkelf2024/Darkelf-Browser-v3-PQC).

---

## Credits

Darkelf CLI TL Edition by Dr. Kevin Moore, 2025.

Special thanks to the open-source community and the Open Quantum Safe project.
