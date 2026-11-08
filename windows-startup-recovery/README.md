# Windows Startup Troubleshooting and Recovery

## Project Overview

In this Windows 11 virtual-machine lab, I examined firmware boot settings, used Safe Mode, inspected Windows Boot Configuration Data (BCD), intentionally introduced an invalid startup configuration, and recovered the system.

**Outcome:** After changing the Windows boot entry's `systemroot` setting to an invalid directory, Windows failed to start normally. I restored the setting to `\Windows` and successfully returned the virtual machine to normal startup.

> **Lab safety:** The invalid boot setting was introduced deliberately in a disposable training VM. Do not perform this experiment on a production computer.

## Lab Environment and Tools

- Windows 11 virtual machine
- Hyper-V Manager (firmware/boot-order inspection)
- Windows Recovery Environment (WinRE)
- Windows Safe Mode
- Administrator Command Prompt
- `bcdedit` (Boot Configuration Data Editor)

## Objectives

- Inspect the virtual machine's firmware boot order
- Start Windows in Safe Mode
- Examine the Windows Boot Manager and Windows Boot Loader entries
- Observe the effect of an invalid Windows system directory in BCD
- Restore the correct boot configuration and verify recovery

## Procedure and Observations

### 1. Inspect firmware boot settings

I opened the Windows 11 VM's firmware settings through **Hyper-V Manager** and reviewed the boot order. The network adapter (`LabServicesSwitch`) was listed last.

**Observation:** The available Hyper-V settings allowed me to reorder boot devices but did not offer a way to disable PXE/network boot. Moving a network adapter to the bottom of the list does **not** disable network boot, so I left the setting unchanged rather than claim it was disabled.

### 2. Explore Safe Mode

I used Windows recovery/startup options to enter **Safe Mode**, which loads Windows with a reduced set of drivers and services. I documented the Safe Mode desktop before returning to normal startup.

### 3. Inspect Boot Configuration Data

From an elevated Command Prompt, I used:

```cmd
bcdedit
```

I examined the **Windows Boot Manager** and **Windows Boot Loader** entries, including the current Windows installation's `systemroot` value.

### 4. Simulate a startup failure

In the training VM, I intentionally changed the current boot entry's Windows system directory:

```cmd
bcdedit /set {current} systemroot \fakedirectory
```

I ran `bcdedit` again to verify that `systemroot` showed `\fakedirectory`. After restarting the VM, Windows could not start normally because the boot configuration referenced an invalid system directory.

### 5. Recover Windows

After the startup failure, I accessed Windows recovery tools and corrected the boot configuration, restoring `systemroot` to:

```text
\Windows
```

I verified the corrected value with `bcdedit` and confirmed that the VM could boot normally again.

The lab record includes a screenshot of the corrected BCD entry. This write-up does not assume an exact recovery command beyond what was documented.

## Troubleshooting Notes

- **Firmware limitation:** Hyper-V exposed boot-order controls but no PXE disable option in the settings I inspected. I distinguished *boot priority* from *disabling a boot method*.
- **Recovery access:** Entering recovery and Safe Mode required some troubleshooting during the exercise.
- **Root cause:** The incorrect `systemroot` value pointed Windows startup to a nonexistent system directory.
- **Resolution:** Restoring the valid Windows directory allowed the system to start normally.

## Skills Demonstrated

- Inspecting virtual machine firmware settings
- Using Safe Mode and Windows recovery tools
- Reading Windows boot configuration entries
- Diagnosing an intentionally misconfigured boot entry
- Verifying configuration changes and successful recovery
- Documenting technical observations accurately

## Lessons Learned

A small configuration change in the boot process can prevent Windows from loading. Before changing startup settings, it is important to understand what each value controls and have a recovery path available. I also learned to document environmental limitations precisely rather than report a step as completed when the available interface did not support it.

## Screenshots

Selected screenshots from my completed lab will be added here, including firmware settings, Safe Mode, the invalid `systemroot` entry, and the corrected `\Windows` entry.

---

*Training project completed in a Windows 11 virtual-machine environment. Documentation is written in my own words from my lab observations.*
