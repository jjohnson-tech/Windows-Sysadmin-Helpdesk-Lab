## 06 - Ticket #005 — Machine Won't Boot

A user's machine displayed "Operating System not found" at boot with no hardware changes made prior. The troubleshooting process covered BIOS boot order verification, Windows Recovery Environment access, boot file repair, and system file integrity checks. Additional recovery scenarios including BSOD investigation and deleted file recovery using Shadow Copies were also documented.

---

## Step 1: Check BIOS Boot Order

The first step when a machine displays "Operating System not found" is to verify the BIOS boot order. If the machine is set to boot from a USB drive, network, or DVD before the hard drive, it will fail to find the OS. This is the quickest fix and should always be checked before assuming boot files are corrupted.

*Access BIOS by pressing DEL, F2, or F12 at startup depending on the manufacturer. Verify the hard drive is set as the primary boot device.*


---

## Step 2: Access Windows Recovery Environment

If the boot order is correct and the machine still won't boot, the Windows Recovery Environment (WinRE) is used to repair boot files. WinRE runs independently of Windows and provides access to repair tools and a command prompt.

WinRE was accessed by booting from a Windows 10 installation USB and selecting:

*Repair your computer → Troubleshoot → Advanced Options → Command Prompt*

![Windows Recovery Environment command prompt accessed from installation USB](images/02-winre-command-prompt.png)

---

## Step 3: Repair Boot Files with bootrec

From the WinRE command prompt, the `bootrec` tool was used to repair the Master Boot Record, boot sector, and Boot Configuration Data. These three commands address the most common causes of boot failures in order from least to most invasive.

```cmd
bootrec /fixmbr
bootrec /fixboot
bootrec /rebuildbcd
```

| Command | Purpose |
|---|---|
| bootrec /fixmbr | Rewrites the Master Boot Record |
| bootrec /fixboot | Rewrites the boot sector |
| bootrec /rebuildbcd | Scans for Windows installations and rebuilds the Boot Configuration Data |

![bootrec commands executed in WinRE command prompt — boot files repaired](images/03-bootrec-repair.png)

---

## Step 4: Repair System Files with sfc and DISM

If bootrec does not resolve the issue, `sfc` and `DISM` are used to repair corrupted Windows system files and the Windows image itself. These commands are run from the WinRE command prompt.

```cmd
sfc /scannow
DISM /Online /Cleanup-Image /RestoreHealth
```

| Command | Purpose |
|---|---|
| sfc /scannow | Scans and repairs corrupted Windows system files |
| DISM /RestoreHealth | Repairs the Windows image using Windows Update as the source — run after sfc if it cannot fix the issue |

![sfc /scannow and DISM /RestoreHealth executed to repair corrupted system files](images/04-sfc-dism.png)

---

## Step 5: Investigate BSOD Crashes

When a machine experiences a Blue Screen of Death (BSOD) and restarts, the stop code and crash details can be found in Event Viewer and the minidump files saved by Windows.

```powershell
Get-EventLog -LogName System -EntryType Error -Newest 20
```

Minidump files are saved automatically at `C:\Windows\Minidump` after every BSOD and can be read using WinDbg to identify the exact stop code and which driver or process caused the crash.

**Common BSOD Stop Codes:**

| Stop Code | Meaning |
|---|---|
| SYSTEM_THREAD_EXCEPTION_NOT_HANDLED | Driver issue |
| CRITICAL_PROCESS_DIED | Critical Windows process crashed |
| UNMOUNTABLE_BOOT_VOLUME | Cannot access boot drive |
| PAGE_FAULT_IN_NONPAGED_AREA | Memory issue |

![Event Viewer showing system errors correlated with BSOD events](images/05-bsod-event-viewer.png)

---

## Step 6: Recover Deleted Files Using Shadow Copies

If a user accidentally deleted files and emptied the Recycle Bin, Shadow Copies can be used to restore previous versions without a full backup restore. Windows automatically takes snapshots of files at scheduled intervals.

Shadow Copies were checked using PowerShell to confirm snapshots existed on the volume:

```powershell
Get-WmiObject -Class Win32_ShadowCopy
```

Users can also restore previous versions directly from File Explorer by right-clicking a folder and selecting **Restore previous versions** without IT involvement.

![Shadow Copies confirmed on the volume — previous versions available for file recovery](images/06-shadow-copies.png)

---

## Summary

| Issue | First Step | Resolution |
|---|---|---|
| OS not found | Check BIOS boot order | Set hard drive as primary boot device |
| Boot files corrupted | Access WinRE | Run bootrec /fixmbr, /fixboot, /rebuildbcd |
| System files corrupted | Run sfc /scannow | Follow with DISM /RestoreHealth if sfc fails |
| BSOD crash | Check Event Viewer | Review minidump in WinDbg for stop code |
| Deleted files | Check Shadow Copies | Restore previous versions from File Explorer |

Always check the simplest fix first — BIOS boot order resolves a large percentage of "OS not found" tickets before any repair tools are needed. Work from least invasive to most invasive: BIOS → bootrec → sfc → DISM.
