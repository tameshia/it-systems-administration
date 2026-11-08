# Windows Command Prompt: File Management, Networking, and System Diagnostics

## Project Overview

In this Windows Command Prompt lab, I practiced managing files and directories, viewing active network connections and network adapter settings, and initiating a Windows system file integrity scan. All tasks were performed using Command Prompt in a Windows virtual machine.

## Lab Environment and Tools

- **Environment:** Windows virtual machine (the assignment permits Windows 10 or Windows 11)
- **Tools:** Command Prompt (`cmd.exe`)
- **Commands:** `cd`, `mkdir`, `dir`, `echo`, `type`, `copy`, `rmdir`, `netstat`, `ipconfig`, and `sfc`

## Objectives

1. Navigate directories and create folders using commands.
2. Create, inspect, copy, and remove files and folders.
3. Read command help and inspect network connections.
4. Review detailed network adapter configuration.
5. Start a Windows System File Checker scan.
6. Compare file-copy tools and practice chaining commands.

## Procedure and Observations

### 1. Create Backup Folders

I navigated to the Documents directory, created `Backup1` and `Backup2`, then used `dir` to verify that both folders existed.

```cmd
cd /d "%USERPROFILE%\Documents"
mkdir Backup1
mkdir Backup2
dir
```
![Creating Backup1 and Backup2 in the Documents folder](screenshots/01-create-directories.png)

*Created `Backup1` and `Backup2` from Command Prompt and verified both folders with `dir`.*
**Observation:** An elevated Command Prompt runs with administrator privileges, although basic file operations do not always require elevation.

### 2. Create and Read a Log File

I created `log.txt` with sample text using output redirection and displayed the result:

```cmd
echo This is my first Windows Command Prompt lab. > log.txt
type log.txt
```

The `>` operator directs output to a file and overwrites existing contents if the file already exists.
![Creating log.txt with echo and reading it with type](screenshots/02-create-log.png)

*Used `echo` with output redirection to create `log.txt`, then displayed the file contents with `type`.*
### 3. Copy and Verify the File

I copied `log.txt` into `Backup2`, then checked the folder contents and displayed the copied text:

```cmd
copy log.txt Backup2
dir Backup2
type Backup2\log.txt
```
![Copying log.txt into Backup2](screenshots/03-copy-file.png)
![Verifying the copied file in Backup2](screenshots/04-verify-copy.png)

*Verified that `Backup2\\log.txt` existed and contained the expected text.*

*Copied `log.txt` into the `Backup2` folder and confirmed the copy operation succeeded.*
Command Prompt reported that one file was copied, and my checks confirmed the file existed and contained the intended text.

### 4. Troubleshoot Directory Removal

My initial command:

```cmd
rmdir Backup2
```

returned **"The directory is not empty."** Because `Backup2` contained the copied log, I used:

```cmd
rmdir /s Backup2
dir
```

After confirming the removal, `dir` showed that `Backup2` had been deleted while `Backup1` remained.
![Removing Backup2 after the first rmdir attempt failed](screenshots/05-remove-directory.png)

*The first `rmdir Backup2` attempt failed because the directory was not empty, so `rmdir /s Backup2` was used to remove the folder and its contents.*
**Troubleshooting lesson:** `rmdir /s` removes the target directory and its contents, so the path should be checked carefully before confirming deletion.

### 5. Examine Network Connections with Netstat

I reviewed help for the command and then checked active connections:

```cmd
netstat /?
netstat
```

The screenshot showed TCP connections with states including `ESTABLISHED` and `SYN_RECEIVED`.

**Observation:** If `netstat` shows no connections, the computer may have little network activity at that moment or may be disconnected.

### 6. Inspect Network Adapter Settings

I used:

```cmd
ipconfig /all
```

The output displayed adapter configuration, including IP addressing, subnet masks, default gateways, DNS servers, DHCP settings, and a Hyper-V virtual network adapter.

**Documentation note:** Hostnames, identifying details, and network information should be reviewed and redacted as appropriate before publishing screenshots publicly.

### 7. Initiate a System File Checker Scan

I ran:

```cmd
sfc /scannow
```

The screenshot showed the scan beginning and entering its verification phase. The assignment permitted stopping it with Ctrl+C without waiting for completion.

**Important distinction:** The screenshot demonstrates that the scan started; it does not show a completed scan or establish that any files were repaired.

### 8. Optional Challenges

I explained the differences between `COPY` (basic file copying), `XCOPY` (files and directory trees), and `ROBOCOPY` (more advanced directory copying with options for retries and metadata preservation).

For the command-chaining exercise, I wrote:

```cmd
cd /d "%USERPROFILE%\Documents" && mkdir Challenge && echo Challenge completed > Challenge\log.txt && dir Challenge
```

With `&&`, the next command runs only when the previous command succeeds. The submitted lab includes the written command but not an execution screenshot.

## Skills Demonstrated

- Command-line navigation and file management
- File creation, redirection, and verification
- Troubleshooting directory deletion errors
- Reading command help and interpreting `netstat` output
- Inspecting network settings with `ipconfig /all`
- Initiating Windows system diagnostics
- Using command chaining

## Lessons Learned

This exercise emphasized verifying outcomes rather than assuming a command worked. I confirmed directory creation and copied-file contents, recognized why an empty-directory removal command failed, and used diagnostic utilities to inspect system and network behavior.

## Screenshots

Screenshots from the completed lab will be added in a `screenshots/` folder and embedded alongside the relevant steps.

---

*Based on a completed IT120 A+ Core 2 Windows Command Prompt lab, documented as a technical portfolio case study.*
