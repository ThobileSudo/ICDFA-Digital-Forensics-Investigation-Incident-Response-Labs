# Lab 1C: Windows Command Line for Digital Forensics

**Course:** CIP-B101 Basic Computer Skills for Digital Forensics (ICDFA)
**Author:** Thobile Ngcobo
**Date completed:** 14 November 2026
**Environment:** Windows 10 Home (build 19045), standard Command Prompt, physical machine (Intel Celeron N3350, about 4 GB RAM)

## Overview

I used the built-in Windows command line to collect and document system information the way an analyst would: in a dedicated lab folder, with every result saved to a file and every step recorded. Failed commands were kept in the record as well.

## Workspace

A lab folder with `evidence`, `outputs`, `screenshots`, `scripts` and `workspace` to `workspace4` subfolders, created before any other command was run.

## What I did

| Area | Commands | What it shows |
|---|---|---|
| Navigation and paths | `cd`, `dir`, `tree`, `echo %cd%` | Absolute paths start at the drive letter; relative paths are read from the current folder |
| File handling | `echo >`, `echo >>`, `type`, `copy`, `xcopy /E /I /H`, `ren`, `del`, `rmdir` | Created evidence notes, backed up a whole folder, and deleted only test items |
| Program lookup | `echo %PATH%`, `where`, `vol` | How Windows decides which program runs; `where` shows duplicates of the same program |
| System and hardware | `systeminfo`, `find`, `wmic cpu`, `wmic diskdrive` | Operating system, memory, CPU and disk details saved to text files |
| Network | `ipconfig /all`, `findstr`, `wmic nic`, `ping` | Addresses and adapters recorded; loopback, internet IP and DNS name all replied with 0% loss |
| Searching text | `findstr /i /n` | Case-insensitive search with line numbers in a small case log |
| Ownership and permissions | `dir /q`, `icacls` | File owner and who has full or modify access (all inherited) |
| Processes | `wmic process list brief`, `wmic process where` | Snapshot of running processes; browser processes all ran from the normal Chrome install folder |
| Automation | `hello_world.bat`, `sys_info.bat` | Batch files that collect system, hardware and network details into one report |

## Selected findings

- `>` overwrites a file and `>>` appends to it. For evidence notes and logs, `>>` is the safer choice.
- `where notepad` found two copies of the program, and the System32 copy is searched first. That is the kind of check used to spot a fake program placed ahead of the real one.
- `systeminfo | find "OS Version"` also matched the BIOS Version line, because `find` matches any line containing the text, not whole words.
- Running `sys_info.bat` from another folder after a temporary `set PATH=...` change failed the first time ("The system cannot find the path specified") because the script used a relative output path. I documented the failure, switched to an absolute path and ran it again successfully.

## Takeaways

- A separate lab folder keeps evidence together and reduces the risk of overwriting or deleting the wrong data.
- Batch files make evidence collection fast, consistent and repeatable.
- A malicious program placed in a PATH folder searched early can run in place of a trusted one (search-order hijacking), so PATH is worth checking in a triage.
- Failed commands belong in the record, because they can be evidence themselves and hiding them damages credibility.

## Notes

Educational portfolio. Device identifiers, account names and network addresses from my own machine have been left out of this write-up.
