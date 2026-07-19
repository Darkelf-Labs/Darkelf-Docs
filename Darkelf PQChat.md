# Darkelf PQChat User & Developer Guide

> **Version:** 3.0  
> **Author:** Dr. Kevin Moore / Darkelf2024  
> **License:** LGPL-3.0-or-later  
> **Export Control:** See source code and repository README for details.

---

## What is Darkelf PQChat?

**Darkelf PQChat** is a post-quantum, CLI-based secure chat system for privacy-focused and security-conscious users.  
It uses [Kyber768](https://pq-crystals.org/kyber/) for quantum-resistant key exchange and Ed25519 for identity authentication, combined with an async prekey (X3DH-style) protocol and a symmetric message ratchet for strong forward secrecy.

---

## Key Features

- **Post-Quantum Secure:**  
  Uses Kyber768 for all key exchanges and session establishment, providing security against current and future quantum computers.

- **Ed25519 Identity Authentication:**  
  Each user is assigned an Ed25519 keypair for strong, fast public-key signatures. All prekeys and sessions are authenticated.

- **Async Prekeys (X3DH-style):**  
  Initiators can connect even if responders are offline, thanks to one-time use prekeys published in a secure mailbox file (`prekeys.json.enc`).  
  This mirrors the security model of Signal/WhatsApp but with post-quantum crypto.

- **Encrypted RAM-Only Prekey Mailbox:**  
  All mailbox data (even public prekeys) is AES-encrypted using a passphrase and never written to disk—it's stored in a RAM disk (`/dev/shm` on Linux, `/private/tmp` or `/Volumes/RAMDisk` on macOS).  
  The mailbox is securely wiped (overwritten and deleted) after use.

- **Replay & Ordering Protection:**  
  All messages are chained with counters and HKDF ratchets. Replay and out-of-order messages are automatically rejected.

- **Forward Secrecy:**  
  Each message uses a symmetric ratchet (HKDF + ChaCha20Poly1305). Compromise of one key does not compromise past (or future) messages.

- **Automatic Prekey Management:**  
  The CLI auto-publishes your prekey if it's missing—no need to run a separate script or command.

- **Simple CLI Workflow:**  
  All operations (publishing prekeys, accepting connections, connecting as client) are handled via simple CLI prompts.

- **No GUI Needed:**  
  Designed for terminal/CLI use—runs on any OS with Python and OQS bindings.

- **One-Time Prekey Consumption:**  
  Each prekey is used only once and is securely deleted after use, providing strong deniability.

- **Works Over LAN, Tor, VPN, or Localhost:**  
  No restrictions on network usage.

- **Intrusion Monitoring:**  
  Background thread watches for dangerous tools (nmap, tcpdump, etc.) and alerts in real-time, logging alerts only in RAM.

- **Extensible and Auditable:**  
  Fully open source, modular, and easy to extend for research or integration into other secure systems.

---

## Basic Usage

### 1. Start as Server (Responder)

```sh
darkelf> pqchat
Start as (s)erver or (c)lient? [s/c]: s
Host (default 127.0.0.1):
Port (default 9000):
Your user ID for prekey: alice
[Prekey Mailbox] Enter passphrase for mailbox at /dev/shm/prekeys.json.enc:
[+] Prekey published for user: alice
[Server] Listening on 127.0.0.1:9000 (async mode)
[Server] Connection accepted
[*] Async handshake complete. You can now receive messages.
```

### 2. Start as Client (Initiator)

```sh
darkelf> pqchat
Start as (s)erver or (c)lient? [s/c]: c
Host (default 127.0.0.1):
Port (default 9000):
Recipient user ID (published prekey): alice
[Prekey Mailbox] Enter passphrase for mailbox at /dev/shm/prekeys.json.enc:
[*] Async handshake complete. You can now send messages.
```

### 1. Start External Server

```sh
Start as (s)erver or (c)lient? [s/c]: s
Host (default 127.0.0.1): 0.0.0.0
Port (default 9000): 9000
Your user ID for prekey: 2027
```

### 2. Start as External Client 

```sh
Start as (s)erver or (c)lient? [s/c]: c
Host (default 127.0.0.1): 68.144.21.133
Port (default 9000): 9000
Your user ID for prekey: 2026
Connect to recipient ID: 2027
```

**Note:**  
- The server auto-publishes its prekey if missing.
- The client can connect if it has access to the encrypted mailbox file (`prekeys.json.enc`) containing the recipient's published prekey (same directory or sync the file).
- The mailbox passphrase is required for any access.

---

## Command Reference

| Command                       | Description                                                   |
|-------------------------------|---------------------------------------------------------------|
| `pqchat`                      | Start a secure PQ chat session (as server or client)          |
| `publish-prekey <user_id>`    | (Optional) Manually publish your prekey to mailbox            |
| `accept-prekey <user_id> <port>` | (Optional) Accept as responder (legacy/manual)              |
| `connect-prekey <their_id> <host> <port>` | (Optional) Connect as initiator (legacy/manual)     |

- **You do NOT need to run `publish-prekey` manually.**  
  It is handled automatically when starting as server.
- All chat sessions are end-to-end encrypted.

---

## How to Exit pqchat

To exit a chat session and return to the main CLI menu, simply type one of the following commands at the `You:` prompt during the chat session:

- `exit`
- `quit`
- `/exit`
- `/quit`

You can also press `Ctrl+C` to leave the chat at any time.

After exiting, you will see a message like:

```
[*] Chat closed. Returning to CLI menu.
```

The Darkelf CLI prompt will be restored and you can enter new commands or start another session.

---

## Security Model

- **X3DH Prekey Protocol:**  
  Similar to Signal, the initiator uses the responder’s prekey for the initial session, allowing for asynchronous, “offline” secure messaging setup.

- **Symmetric Ratchet:**  
  Each message advances a key ratchet, providing forward secrecy and replay protection.

- **Identity Verification:**  
  Prekeys are signed by the Ed25519 identity key, which is verified on handshake.

- **Ephemeral Session Keys:**  
  All session keys are ephemeral and not reused.

- **Encrypted Prekey Mailbox:**  
  All mailbox contents are Fernet- and PBKDF2-encrypted, with the key derived from your passphrase and a strong random salt.

- **RAM-Only Storage:**  
  The mailbox is stored only in RAM (never on disk) and securely wiped after use.

- **Zero Knowledge of Private Keys:**  
  Your private keys never touch disk and are kept only in process memory.

- **Intrusion Monitoring:**  
  Real-time alerts for dangerous tools, with logs written only to RAM.

---

## File Structure

- `prekeys.json.enc` — Encrypted mailbox for user prekey bundles (RAM-only, passphrase-protected)
- `Darkelf_CLI_TL_OSINT Tool Kit.py` — Main CLI and chat implementation

---

## Example Workflow

1. **On the responder machine:**
    - Start the server via `pqchat` and enter your user ID.
    - Enter a mailbox passphrase (prompted if missing).
    - Prekey is automatically published if missing.

2. **On the initiator machine:**
    - Make sure the encrypted `prekeys.json.enc` is available (copy if needed).
    - Start as client via `pqchat` and enter the responder’s user ID.
    - Enter the mailbox passphrase.

3. **Chat securely.**

---

## Advanced Usage

- **Changing User ID:**  
  Your user ID is any unique string (e.g., your nickname). Use the same ID every time you want others to connect to you.

- **Prekey Mailbox Sharing:**  
  For cross-machine use, copy the encrypted mailbox file between computers, or extend to use a server.

- **One-Time Prekey:**  
  Each prekey is deleted on use; to accept a new connection, restart as server to republish.

---

## Security Best Practices

- **Always use strong, unique user IDs.**
- **Keep your mailbox passphrase secret and strong.**
- **Keep `prekeys.json.enc` secure and private.**
- **Delete chat logs if not needed.**
- **Run over Tor or VPN for extra network anonymity.**

---

## Extending Darkelf PQChat

- **Support for network mailbox servers:**  
  Replace the local encrypted mailbox with a remote server for distributed use.
- **Integrate with GUI or mobile apps:**  
  The protocol can be adapted to any interface.
- **Batch Prekeys:**  
  Publish multiple prekeys for increased scalability (not yet implemented).

---

## Troubleshooting

- **"No prekey mailbox found":**  
  Ensure you or the recipient have started as server at least once to publish their prekey.
- **"No prekey bundle found for recipient":**  
  Make sure the recipient’s prekey is available in the encrypted mailbox file.
- **"Incorrect passphrase or corrupted mailbox":**  
  Make sure you are using the correct mailbox file and passphrase.

---

## License & Export

- **LGPL-3.0-or-later**  
  See source code and repository for export compliance and usage restrictions.

---
**For more information, see the source code and issues on [GitHub](https://github.com/Darkelf2024/Darkelf-Browser-v3-PQC).**
