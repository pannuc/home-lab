# Building a Kali Linux Home Lab (VirtualBox) + Connecting to TryHackMe

| | |
|---|---|
| **Project** | Home lab setup |
| **Date** | [10-06-2026] |
| **Host OS** | [Linux] |
| **Hypervisor** | Oracle VirtualBox |
| **Guest** | Kali Linux 2026.2 (pre-built VirtualBox image) |

---

## Objective

Set up a local Kali Linux virtual machine to practice hands-on security skills and connect it to TryHackMe—so I can work in my own environment with no time limits, instead of a browser-based machine.

---

## Setup Process

I used the **pre-built VirtualBox image** rather than a manual ISO install, since it comes pre-configured and skips the full OS installation.

1. Installed VirtualBox from virtualbox.org.
2. Installed 7-Zip to extract the compressed image.
3. Downloaded the pre-built Kali VirtualBox image (`.7z`) from kali.org/get-kali.
4. Extracted the archive to get the `.vbox` and `.vdi` files.
5. Imported it via VirtualBox → Machine → Add → selected the `.vbox` file.
6. Allocated [X] GB RAM and [X] CPU cores in Settings.

---

## Troubleshooting: VM Failed to Boot

On first launch, the VM wouldn't start and threw this error:

```
Not in a hypervisor partition (HVP=0) (VERR_NEM_NOT_AVAILABLE).
AMD-V is disabled in the BIOS (or by the host OS) (VERR_SVM_DISABLED).
Result Code: E_FAIL (0x80004005)
```

**Diagnosis:** CPU virtualization was disabled. VirtualBox needs hardware virtualization (AMD-V / Intel VT-x) enabled to run a VM.

**Key gotcha:** On AMD systems, the BIOS setting isn't labeled "AMD-V" — it's called **SVM Mode**. Looking for "AMD-V" directly turns up nothing.

**Fix:**
1. Rebooted into BIOS/UEFI ([key] on [my board/laptop]).
2. Found **SVM Mode** under [Advanced → CPU Configuration] and set it to **Enabled**.
3. Saved and exited, rebooted.
4. VM booted successfully.

*(If this hadn't worked, the next step would've been checking for a Hyper-V conflict on Windows — the `HVP=0` line hints at it — by disabling Hyper-V, Memory Integrity, and running `bcdedit /set hypervisorlaunchtype off`.)*

---

## First Steps After Boot

- **Took a snapshot** ("clean install") as a rollback point before changing anything.
- **Updated the system:** `sudo apt update && sudo apt full-upgrade -y`
- **Changed the default password** with `passwd` (image default is `kali`/`kali`).

---

## Connecting to TryHackMe

Connected the VM to TryHackMe's network over OpenVPN:

1. Logged into TryHackMe in Kali's Firefox and downloaded my `.ovpn` config from the Access page (downloading inside the VM avoids a host-to-VM file transfer).
2. Connected: `sudo openvpn yourusername.ovpn`
3. Confirmed `Initialization Sequence Completed`, left the session running in its own terminal.
4. Verified the tunnel with `ip addr show tun0` and saw the green connected status on TryHackMe.

> The `.ovpn` file is account-specific — kept out of version control entirely.

---

## Takeaways

- Hardware virtualization (VT-x / AMD-V) must be enabled in BIOS for any VM to run — and on AMD it's hidden under the name **SVM Mode**.
- Windows' own Hyper-V can conflict with VirtualBox even when virtualization is on.
- Snapshots make a lab safe to experiment in — break something, roll back in seconds.
- OpenVPN connects my local machine into a remote target network, and the config file is a credential to protect.
