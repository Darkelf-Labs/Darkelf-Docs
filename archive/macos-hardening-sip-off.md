
# 🔐 macOS Hardening Guide (When SIP and Swap Are Disabled)

If you’ve disabled **System Integrity Protection (SIP)** and **swap** for secure workloads (e.g., Darkelf, forensics, crypto), use this guide to harden macOS using alternative controls. This Guide is intended for Persistence - Full Time Defense

---

## 🧱 1. Minimal System Configuration

- Disable unnecessary background services:
  ```bash
  sudo launchctl disable system/com.apple.smbd
  sudo launchctl disable system/com.apple.telnetd
  sudo launchctl disable system/com.apple.RemoteLogin
  ```

- Turn off Spotlight indexing:
  ```bash
  sudo mdutil -a -i off
  ```

---

## 🔐 2. Memory Protection Alternatives

- Use `mlock()` or secure memory buffers to avoid paging sensitive data.
- Zero memory on shutdown using secure memory erasure techniques.
- Avoid using plain Python strings or insecure memory objects for secrets.

---

## 📡 3. Disable Network Attack Surfaces

- Disable:
  - Bluetooth
  - AirDrop
  - Bonjour/mDNS
- Use firewall apps like **LuLu** or **Little Snitch** to control outbound traffic.

---

## 💾 4. FileVault 2 + Boot Security

- Enable FileVault 2 to encrypt the disk.
- Set a firmware password (Intel Macs) or configure **Recovery Lock** (Apple Silicon) to restrict access to Recovery Mode.

---

## 🧱 5. Root Filesystem Integrity (Advanced)

- Inspect `/etc/rc.common`, `/Library/LaunchDaemons/`, and `/etc/synthetic.conf` for startup persistence or tampering.
- Use `csrutil authenticated-root disable` to manually mount and inspect sealed system volumes (advanced users only).

---

## 🧼 6. Forensic Self-Checks

Create a boot script to verify swap and SIP status:

```bash
#!/bin/bash
sysctl vm.swapusage | grep 'used = 0.00M' || echo "⚠️ Swap is active"
csrutil status | grep disabled || echo "⚠️ SIP may be enabled"
```

---

## 🚫 7. Disable Automatic Updates

- macOS updates can silently re-enable SIP or dynamic pager.
- Disable auto-updates and run:
  ```bash
  softwareupdate --ignore
  ```

- Manually apply updates and recheck system security status after updating.

---

## 🧬 8. Network & DNS Egress Control

- Use **LuLu** / **Little Snitch** to detect and block:
  - Unwanted Apple telemetry
  - DNS leaks
  - Unexpected outbound connections

- Optionally harden `/etc/hosts` with:
  ```bash
  sudo nano /etc/hosts
  ```

---

## ✅ 9. Optional App Sandboxing / VMs

- Use **UTM**, **VirtualBuddy**, or custom App Sandboxing to isolate sensitive tasks.
- Consider running crypto operations inside a RAM-only VM.

---

## 🔁 10. Check After macOS Updates

After major system updates:

- Re-check:
  ```bash
  sysctl vm.swapusage
  launchctl print-disabled system | grep dynamic_pager
  csrutil status
  ```

- Reset any system hardening that was undone during the update.

---

## ✅ Summary Table

| Goal                         | Alternative (SIP/swap off) |
|------------------------------|-----------------------------|
| Memory safety                | `mlock()`, zero on shutdown |
| Disk protection              | FileVault 2 + firmware password |
| No swap ever                 | disable + delete + monitor |
| Boot integrity               | disable auto updates + manual checks |
| Network hardening            | LuLu, Little Snitch, egress firewall |
| Recovery lock                | Apple ID/iCloud Recovery Lock |
| Service isolation            | Unload unneeded LaunchDaemons |
| Software updates             | Manual only, with rollback snapshot |
| Secure app exec              | Run crypto tools in sandbox or RAM-only VMs |

---

Use this as your fallback security framework when Apple’s default protections (SIP, Secure Boot, Swap) are intentionally disabled.
