# UsnJrnlShield

Windows kernel minifilter that fakes USN Journal timestamps so forensic tools like **JournalTrace** cannot detect when logs were cleaned.

![Windows](https://img.shields.io/badge/platform-Windows%2010%2F11%20x64-blue)
![C++](https://img.shields.io/badge/language-C%2B%2B17-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What it does

- Intercepts `FSCTL_READ_USN_JOURNAL` at **kernel level**
- Rewrites `TimeStamp` field in every `USN_RECORD_V2` **on the fly**
- JournalTrace always shows the **original date**, even after wiping logs
- Survives reboots, wipes, and JournalTrace relaunches

**Result:** forensic tools cannot determine when you cleaned the logs.

---

## How it works

1. **User-mode part** (`UsnJrnlShield.exe`) saves the current USN state to the registry.
2. **Kernel minifilter** (`UsnJrnlShieldDrv.sys`) intercepts every USN read request.
3. Before the data reaches the application, the driver replaces each `TimeStamp` with the saved reference value.
4. The forensic tool sees old timestamps. The actual USN Journal may contain anything.

---

## Requirements

- **Windows 10 / 11 x64**
- **Administrator** rights
- **Secure Boot** disabled in BIOS/UEFI
- **Test Mode** enabled (`bcdedit /set testsigning on`)
- Self-signed certificate installed (`UsnJrnlShieldTest.cer`)

---

## Installation

### 1. Disable Secure Boot

Reboot -> enter BIOS/UEFI (Del, F2, F10, F12) -> find **Secure Boot** -> set to **Disabled** -> Save & Exit.

### 2. Run the installer

Right-click **install.bat** -> **Run as administrator**.

### 3. Reboot Windows

You will see a **Test Mode** watermark in the corner. This is normal.

### 4. Load the driver

Open **cmd as Administrator**:

fltmc load UsnJrnlShieldDrv

### 5. Save the reference date

In the same cmd:

UsnJrnlShield.exe snapshot

This saves the current "Oldest entry" as the reference for future wipes.

---

## Daily usage

Every time before cleaning logs, run:

fltmc load UsnJrnlShieldDrv
UsnJrnlShield.exe wipe
UsnJrnlShield.exe restore

Then open **JournalTrace.exe** - it will show the **old date**, not the current one.

---

## Commands

| Command | Description |
|---|---|
| `UsnJrnlShield.exe snapshot` | Save current USN state to registry |
| `UsnJrnlShield.exe wipe` | Delete USN Journal and recreate |
| `UsnJrnlShield.exe restore` | Restore saved timestamps |
| `UsnJrnlShield.exe status` | Print current USN state |
| `UsnJrnlShield.exe watch` | Background watchdog (Ctrl+C to stop) |

---

## Disable / Enable

### Disable (when not in use)

bcdedit /set testsigning off
shutdown /r /t 0

The "Test Mode" watermark disappears. The driver stops loading.

### Enable again

bcdedit /set testsigning on
shutdown /r /t 0
fltmc load UsnJrnlShieldDrv

**Nothing needs to be reinstalled** - the driver and certificate stay in place.

---

## Uninstall

1. Run **uninstall.bat** as Administrator
2. Reboot Windows
3. Re-enable Secure Boot in BIOS/UEFI

---

## Files in this repository

| File | Description |
|---|---|
| `UsnJrnlShieldDrv.sys` | Kernel minifilter driver (signed) |
| `UsnJrnlShieldDrv.inf` | Driver installation script |
| `UsnJrnlShieldTest.cer` | Self-signed public certificate |
| `UsnJrnlShield.exe` | User-mode CLI tool |
| `install.bat` | Automated installer |
| `uninstall.bat` | Automated uninstaller |
| `README.txt` | Short plain-text instructions |

---

## Warning

**Test Mode reduces system security.** Do not use on production systems without understanding the risks.

- Use only on **your own machine** or in a **lab / VM** environment.
- Do not distribute the `.sys` file separately from the `.cer` certificate.
- The reference timestamp is currently **hardcoded** inside the driver. To change it, rebuild from source.

---

## License

MIT License.

---

## Credits

Created by **xD4rk**
