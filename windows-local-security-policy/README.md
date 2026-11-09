# Windows Local Security Policy: User Rights and Security Auditing

**Environment:** Windows 10 virtual machine (JCCC Cyber Skills Lab 2)  
**Focus:** Local users and groups, user rights, interactive logon settings, security auditing, and Event Viewer  
**Status:** Lab completed; seven required assignment screenshots captured. Screenshot files will be added to this repository separately.

## Objective

Practice applying local security settings to a Windows workstation and evaluating the resulting security events. The exercise covered account membership, logon rights, shutdown privileges, logon messaging, logon/logoff auditing, and file-access auditing.

> **Scope:** This exercise was performed in a training virtual machine. The sample account names and security settings below are for the lab, not recommendations for a production workstation.

## Tools and Concepts

- **Computer Management / Local Users and Groups:** Create and inspect standard local user accounts.
- **Local Security Policy (`secpol.msc`):** Configure user rights, security options, and advanced audit policy.
- **File auditing (SACL):** Choose which attempts to access a test file should generate security audit events.
- **Event Viewer:** Filter the Windows Security log and examine relevant event details.

## Lab Procedure

### 1. Create and inspect local accounts

The assignment used two standard local accounts, `secpol` and `secpol2`. In Computer Management, the account properties' **Member Of** tab was used to inspect group membership. These standard accounts were associated with the local **Users** group.

**Required evidence:** Creating User Accounts — Step 7 (group membership).

### 2. Configure local security controls

In **Local Security Policy**, the exercise covered these settings:

| Area | Setting or activity | Purpose |
| --- | --- | --- |
| User Rights Assignment | Add `secpol2` to **Deny log on locally** | Prevent that account from signing in at the workstation console |
| User Rights Assignment | Remove **Users** from **Shut down the system** | Restrict shutdown rights for standard users who have no other qualifying rights |
| Security Options | Configure interactive logon warning text and title | Display a notice before interactive sign-in |
| Advanced Audit Policy — Logon/Logoff | Audit successful logoffs and failed logons | Record authentication-related activity |
| Advanced Audit Policy — Object Access | Audit successful and failed file-system access | Make eligible file activity available for analysis |

The lab's warning title was **WARNING**, with message text **You are being monitored**.

**Required evidence:** Configuring a Local Security Policy — Steps 15, 20, and 28.

### 3. Test the policies

The assignment included testing account logon behavior, reviewing the warning at the sign-in screen, attempting shutdown from a standard account, generating a deliberately failed sign-in, and accessing a test text file.

For file auditing, the exercise created `secpoltest.txt`, configured an auditing entry, and edited the file using Notepad. Event Viewer was then used to examine the Windows Security log.

**Required evidence:** Testing Security Policies — Steps 2, 15, and 23.

## Verified Event Viewer Findings

The following results were observed during the completed lab, rather than copied from the instructor's example screenshots.

| Event ID | Meaning | Observed evidence |
| --- | --- | --- |
| **4663** | An attempt was made to access an object | Successful file-access event associated with `secpoltest.txt`, user **Tameshia**, process `notepad.exe`, **WriteData/AppendData**, access mask `0x6`, at **1:21:52 PM** |
| **4647** | User initiated logoff | Logoff by the **secpol** account at **11:56:41 AM**; Logon ID `0x1914C84` |
| **4625** | An account failed to log on | Audit Failure for user **Tameshia** at **12:42:28 PM**; interactive Logon Type **2**; status `0xC000006D`, substatus `0xC000006A` (incorrect password) |

### Troubleshooting observation

We also investigated whether a **4634** (account logged off) event could be matched to the **4647** user-initiated logoff. A matching 4634 was **not located** in the events reviewed. Nearby events involving `UMFD` or `DWM` were not treated as proof of the same user session because their logon identifiers differed.

This is an important distinction: events should be correlated using relevant fields such as **account**, **Logon ID**, and **timestamp**, rather than assumed related solely because their timestamps are close.

## Screenshot Evidence

Seven screenshots were captured for the course assignment. They are **not yet included in this GitHub folder**. Once added, this section can link to the actual files without broken image references.

| Screenshot | Lab section | Required step | Evidence to show |
| --- | --- | --- | --- |
| 1 | Creating User Accounts | 7 | Local account group membership |
| 2 | Configuring a Local Security Policy | 15 | Shutdown user-rights assignment |
| 3 | Configuring a Local Security Policy | 20 | Interactive logon message/title setting |
| 4 | Configuring a Local Security Policy | 28 | File-system audit policy |
| 5 | Testing Security Policies | 2 | Interactive logon warning displayed |
| 6 | Testing Security Policies | 15 | Test-file auditing entry |
| 7 | Testing Security Policies | 23 | Event Viewer file-access event (4663) |

**Privacy note:** Before publishing screenshots, review them for personal account identifiers, machine names, internal network details, or other information that should not be public. Redact sensitive elements while retaining the technical evidence.

## Lessons Learned

- **User rights are not the same as group membership:** A local account can belong to the Users group while a specific policy explicitly denies a capability.
- **Audit policy and file-level auditing work together:** Enabling file-system auditing alone is not sufficient; the file or folder also needs an appropriate auditing entry to produce relevant events.
- **Event IDs need context:** Event 4625 describes a failed sign-in, but the failure reason, status/substatus, and logon type explain *what happened*.
- **Avoid false correlations:** A nearby Event 4634 is not necessarily related to a particular Event 4647 without matching session details.

## Repository Next Step

Add the seven redacted assignment screenshots to a `screenshots/` directory and replace the evidence table entries with links to the corresponding files.

---

**Related work:** [IT Systems Administration Labs](../README.md)  
**Course context:** CompTIA A+ Core 2 / Cyber Skills Lab 2 — Local Security Policy
