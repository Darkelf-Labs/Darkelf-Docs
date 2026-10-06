## Permanent Swap Disable Guide (macOS M1–M4, 2025)

This guide shows how to **permanently disable macOS swap** on Apple Silicon (M1–M4) with **Permissive Security**, **SIP disabled**, and **dynamic pager off**. Ideal for secure, memory-only workloads like Darkelf, or forensics-safe use cases.

---

## :white_check_mark: STEP 1: Boot into Recovery Mode

1. Shut down your Mac.
2. Press and **hold the power button** until you see **“Loading startup options.”**
3. Click **“Options”**, then click **Continue**.

---

## :white_check_mark: STEP 2: Set Security Policy to Permissive

1. From the top menu: **Utilities → Startup Security Utility**
2. Select your disk (e.g., `Macintosh HD`)
3. Choose:
   - :white_check_mark: **Permissive Security**
   - :white_check_mark: Enable both checkboxes:
     - "Allow user management of kernel extensions"
     - "Allow remote management of kernel extensions"
4. Click **OK**

---

## :white_check_mark: STEP 3: Open Terminal (still in Recovery)

From the top menu: **Utilities → Terminal**

---

## :wrench: STEP 4: Disable SIP (System Integrity Protection)

```bash
csrutil disable
csrutil authenticated-root disable
nvram boot-args="vm_compressor=2"
```

---

## :wrench: STEP 5: Disable the Dynamic Pager (macOS Swap Manager)

```bash
launchctl disable system/com.apple.dynamic_pager
```

---

## :mag: STEP 6: Verify Your Volume Name

```bash
ls /Volumes
```

Look for something like:

```
Macintosh HD
```

---

## :broom: STEP 7: Delete Existing Swap and Sleep Files

```bash
rm /Volumes/Macintosh\ HD/private/var/vm/sleepimage
rm -f /Volumes/Macintosh\ HD/private/var/vm/swapfile*
```

Verify the directory is empty:

```bash
ls -lh /Volumes/Macintosh\ HD/private/var/vm/
```

Expected:

```
total 0
```

---

## :repeat: STEP 8: Reboot

```bash
reboot
```

---

## :white_check_mark: STEP 9: After Reboot — Confirm It Worked (in macOS Terminal)

### :mag: Check Swap Usage

```bash
sysctl vm.swapusage
```

Expected:

```
total = 0.00M  used = 0.00M  free = 0.00M
```

### :mag: Check Pager is Permanently Disabled

```bash
launchctl print-disabled system | grep dynamic_pager
```

Expected:

```
"com.apple.dynamic_pager" => true
```

---

## :white_check_mark: Optional: Re-enable Swap Later

If needed, you can re-enable macOS swap by doing the following in Terminal:

```bash
sudo launchctl enable system/com.apple.dynamic_pager
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist
```

Then verify:

```bash
sysctl vm.swapusage
```

Expected (example):

```
total = 2048.00M  used = 0.00M  free = 2048.00M
```

---

## :tada: Result

You are now running a **RAM-only**, **swap-free**, **SIP-disabled**, and **forensically secure macOS environment**.

Use cases:
- Darkelf
- Encrypted Tor gateways
- Air-gapped systems
- Crypto apps with sensitive keys in memory only

---

## :jigsaw: Warning

Swap may return if:
- You reset NVRAM (rare on Apple Silicon)
- You re-enable SIP or Secure Boot
- You update macOS (re-check after updates)
- You re-enable `dynamic_pager` manually

For best results: confirm swap is off after every system update.

In Terminal: 
csrutil status 
sysctl vm.swapusage
