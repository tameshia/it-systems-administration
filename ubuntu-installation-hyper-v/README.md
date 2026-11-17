# Installing Ubuntu 22.04 LTS in Hyper-V: Installation and GRUB Troubleshooting

## Project Overview

In this CompTIA A+ Core 2 lab, I installed **Ubuntu 22.04 LTS Desktop** on the school-provided **ws1L** virtual machine in Hyper-V. The assignment originally referenced VMware, but the classroom environment had transitioned to Hyper-V. The initial installation media failed during GRUB startup; I investigated the failure, tested a newer Ubuntu 22.04 point-release ISO, completed the installation, and verified that the system booted to its login screen.

**Environment:** School-provided Windows Hyper-V host; Generation 2 / UEFI-style VM **ws1L**  
**Resources observed:** 8,192 MB startup RAM; 2 virtual processors; dynamic memory off; Secure Boot disabled; SCSI virtual hard disk `ws1L.vhdx`; virtual DVD drive  
**Media tested:** Original school-provided `Ubuntu22.iso` (identified as Ubuntu 22.04.1); replacement `ubuntu-22.04.5-desktop-amd64.iso`  
**Tools:** Hyper-V Manager, Ubuntu installer, Windows PowerShell, `Get-FileHash`, `Mount-DiskImage`, `Get-ChildItem`

> **Lab-only safety note:** “Erase disk and install Ubuntu” formats the selected target disk. This was performed in the designated lab VM, not on the Windows host. Verify VM identity, attached storage, and any existing data before using this option. Do not run destructive installation steps on production systems.

### Learning Objectives

1. Configure installation media and boot order for a Hyper-V Linux VM.
2. Recognize the roles of UEFI, GRUB, the Linux kernel, and the initial RAM disk.
3. Install Ubuntu using automatic disk partitioning and understand ESP and ext4.
4. Troubleshoot an early boot failure with controlled configuration checks.
5. Verify successful installation, remove the installation medium, and boot from the virtual disk.
6. Separate observed resolution from an unproven root-cause hypothesis.

## Part 1 — Prepare the Virtual Machine

The school assignment specified **ws1L** and an Ubuntu 22.04 Desktop ISO. I inspected the existing VM rather than creating a replacement.

| Setting | Verified configuration |
| --- | --- |
| VM | `ws1L` |
| Memory | 8,192 MB (8 GB) |
| Dynamic Memory | Disabled |
| Virtual processors | 2 |
| Firmware | UEFI-style Hyper-V firmware |
| Secure Boot | Disabled |
| Virtual disk | `ws1L.vhdx` attached under SCSI Controller |
| Installation media | ISO attached to virtual DVD drive |
| Boot priority | Changed to DVD first for installation |

The original boot order prioritized the virtual hard disk; I moved the DVD drive ahead of it. This change alone did **not** resolve the boot error.

**Concept:** Hyper-V presents virtualized hardware to the guest. Firmware selects a boot device; the bootloader then loads the operating system. Those are separate stages.

## Part 2 — Troubleshooting the Initial Boot Failure

### Symptom

The original school-provided Ubuntu image reached **GNU GRUB 2.06** and showed the normal installation menu, including **Try or Install Ubuntu** and **Ubuntu (safe graphics)**. After choosing a boot option, the VM displayed:

```text
error: can't allocate initrd.
Press any key to continue...
```

**Interpretation:** GRUB reported a failure allocating memory for the initial RAM disk (`initrd`), which is used during early Linux startup. This error does not, by itself, establish that the VM lacks enough total RAM.

### Troubleshooting log

| Check or action | Observed outcome |
| --- | --- |
| Inspected VM memory | 8 GB configured; error still occurred |
| Verified two virtual processors | Configuration was present; no change made |
| Verified Secure Boot | Already disabled |
| Prioritized virtual DVD in firmware boot order | Same error |
| Selected Ubuntu safe graphics option | Same error |
| Temporarily tested 4 GB startup RAM, dynamic memory off | Same error; later restored 8 GB |
| Checked original ISO file size | 3,826,831,360 bytes |
| Calculated original ISO SHA-256 | Hash collected and compared with Ubuntu 22.04.1 release information |
| Replaced installation media with Ubuntu 22.04.5 Desktop ISO | Reached Ubuntu startup splash and installer welcome screen |
| Installed Ubuntu and restarted without installation ISO | Ubuntu login screen appeared |

The original image's recorded SHA-256 was:

```text
C396E956A9E52C418397867D1EA5C0CE1A99A49DCE648B086D2EB762330CC88D
```

Example read-only PowerShell command used:

```powershell
Get-FileHash -Path 'C:\Users\adminuser\Desktop\ISO_Folder\IT 120\Ubuntu22.iso' -Algorithm SHA256
```

### Resolution and evidence limits

**Observed resolution:** Booting the same designated Hyper-V VM from **Ubuntu 22.04.5 LTS Desktop** allowed the installation to proceed. The installation completed and Ubuntu subsequently booted to the login screen.

**Root cause: not conclusively established.** The newer ISO differs in several components. The successful boot is consistent with an ISO-version-related compatibility difference, but no controlled test isolated a particular GRUB patch, kernel, initrd change, or firmware interaction. Changing the ISO was an effective workaround, **not** proof of a specific software defect.

### Additional investigation (condensed)

After the successful installation, I mounted the ISOs in elevated PowerShell and inspected the `casper` and `EFI\BOOT` directories. Windows PowerShell initially returned `0x80070522` (“A required privilege is not held by the client”) when mounting an ISO without elevation; opening **Administrator: Windows PowerShell** allowed the mount operation.

| File | Ubuntu 22.04.1 (bytes) | Ubuntu 22.04.5 (bytes) |
| --- | ---: | ---: |
| `casper/initrd` | 119,611,265 | 146,916,214 |
| `casper/vmlinuz` | 11,090,688 | 14,928,264 |
| `EFI/BOOT/grubx64.efi` | 1,480,584 | 2,275,208 |

The newer initrd was **larger**, despite booting successfully. These size differences demonstrate changed media contents, not which change fixed the problem. A subsequent attempt to compare package manifests did not establish a valid cross-version package comparison because mounted-drive identity was uncertain. I stopped the deeper root-cause investigation rather than claim an unsupported conclusion.

## Part 3 — Install Ubuntu

1. Booted the VM from the replacement Ubuntu 22.04.5 ISO.
2. Reached the Ubuntu installer **Welcome** screen and chose **Install Ubuntu**.
3. Continued through installer setup to **Installation type**.
4. Selected **Erase disk and install Ubuntu** for the designated lab virtual disk.
5. Reviewed the **Write the changes to disks?** confirmation. The installer identified `sda` and proposed:
   - **Partition 1:** EFI System Partition (ESP), used for UEFI boot files.
   - **Partition 2:** **ext4**, the Linux filesystem.
6. Confirmed the partition changes and completed the remaining installation prompts, including account setup.
7. Reached the **Installation Complete** dialog and selected **Restart Now**.

**Evidence boundary:** The screenshots confirm the partition confirmation and completed installation, but do not independently show every intermediate installer choice (for example, keyboard layout or software selection). The assignment also requested adding Terminal to Favorites and capturing a desktop screenshot; those final actions are not independently evidenced in the screenshots available for this portfolio write-up.

### Disk concepts: Windows and Linux

| Concept | In this Ubuntu installation | Related Windows concept |
| --- | --- | --- |
| Virtual disk | Hyper-V `ws1L.vhdx`, presented to Linux as a disk such as `sda` | A virtual disk presented to Windows |
| ESP | Partition containing UEFI boot files | EFI System Partition on UEFI Windows systems |
| ext4 | Filesystem for the Linux installation | NTFS is a common Windows filesystem |
| `sda1`, `sda2` | Linux disk/partition naming | Disk and partition identifiers in Disk Management/DiskPart |
| Bootloader | GRUB starts the Linux boot process | Windows Boot Manager serves a related role |

## Part 4 — Remove Installation Media and Verify Boot

After installation, I opened **Hyper-V Settings → SCSI Controller → DVD Drive** and changed **Media** to **None**. I left the virtual hard disk attached.

**Observed:** The VM restarted and displayed the Ubuntu login screen with the newly created account. This verifies that the installed system could boot from its virtual hard disk without the installer ISO attached.

Disconnecting the ISO prevents the VM from inadvertently returning to the live installer. It does **not** uninstall Ubuntu or delete the virtual disk.

## Lessons Learned

- A boot error's wording is a diagnostic clue, not a complete root-cause explanation.
- Sufficient configured RAM does not guarantee that a bootloader can make a particular memory allocation.
- Change one variable at a time and record both successful and unsuccessful tests.
- A checksum verifies file identity only when compared with a trusted reference.
- Windows administrative membership and an elevated PowerShell process are not the same thing.
- Verify the mounted image and drive letter before drawing conclusions from file comparisons.
- Installation success and post-install boot verification are distinct milestones.
- Technical documentation should distinguish **observed**, **inferred**, and **not verified**.

## Knowledge Check

<details>
<summary><strong>1. What does GRUB do during Linux startup?</strong></summary>

GRUB is a bootloader. After firmware selects a bootable device, GRUB can present boot choices and load the Linux kernel and initial RAM disk.
</details>

<details>
<summary><strong>2. What is initrd, and why does the error matter?</strong></summary>

The initial RAM disk supplies temporary early-boot userspace and drivers/tools needed before the real root filesystem is ready. The reported error means GRUB could not allocate/load that image as required; it does not uniquely identify the underlying cause.
</details>

<details>
<summary><strong>3. How are UEFI and GRUB different?</strong></summary>

UEFI is platform firmware that initializes the machine and selects a boot target. GRUB is software that loads an operating system after firmware transfers control.
</details>

<details>
<summary><strong>4. Why didn't testing 4 GB prove or disprove a GRUB bug?</strong></summary>

The same failure at 4 GB and 8 GB makes a simple total-RAM-shortage explanation less likely, but neither test isolates GRUB code or the firmware memory map.
</details>

<details>
<summary><strong>5. What are ESP and ext4 used for?</strong></summary>

The EFI System Partition stores files used by UEFI boot. ext4 is a Linux filesystem for storing files and directories on a Linux partition.
</details>

<details>
<summary><strong>6. Why disconnect the installation ISO after installation?</strong></summary>

To let the VM boot the newly installed OS from the virtual hard disk instead of returning to installation media.
</details>

<details>
<summary><strong>7. What's the difference between a workaround and a proven root cause?</strong></summary>

A workaround is a change that restores the desired behavior. A proven root cause requires evidence connecting a specific mechanism or defect to the failure, ideally through a controlled reproduction or verified upstream fix.
</details>

<details>
<summary><strong>8. Why was an elevated PowerShell window needed for Mount-DiskImage?</strong></summary>

The first mount attempt failed with Windows error 0x80070522, indicating the process lacked a required privilege. Retrying in an elevated administrator session succeeded. An administrator account does not automatically elevate every process.
</details>

## References

- [Ubuntu — Ubuntu 22.04 LTS release images](https://releases.ubuntu.com/22.04/)
- [Ubuntu — Install Ubuntu Desktop](https://ubuntu.com/tutorials/install-ubuntu-desktop)
- [GNU GRUB Manual](https://www.gnu.org/software/grub/manual/grub/grub.html)
- [Microsoft Learn — Hyper-V on Windows](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/)
- [Microsoft Learn — Get-FileHash](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-filehash)
- [Microsoft Learn — Mount-DiskImage](https://learn.microsoft.com/en-us/powershell/module/storage/mount-diskimage)

---

[← Back to IT Systems Administration Labs](../README.md)
