Thobile Ngcobo
================================================================================
 
# PC System Fundamentals, Disk Sectors and Partition Analysis
 
![Course](https://img.shields.io/badge/Course-CIP--B101-0B2A5B)
![Lab](https://img.shields.io/badge/Lab-1B-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%7C%20Kali%20(WSL)-informational)
![Approach](https://img.shields.io/badge/Approach-Read--only%20forensics-success)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
 
A practical digital-forensics lab for **ICDFA CIP-B101: Basic Computer Skills for Digital Forensics**.
It documents a safe, read-only examination of a Windows 10 laptop: identifying the system,
mapping evidence layers, observing processes, calculating disk and partition sizes from
sector counts, and **verifying two partitions directly from the raw GPT partition table**.
 
> **Forensic mindset:** observe, record, calculate, verify, and document. Never change the evidence.
 
---
 
## Table of contents
 
- [Objectives](#objectives)
- [Key findings](#key-findings)
- [Environment](#environment)
- [Repository structure](#repository-structure)
- [Methodology](#methodology)
- [Commands used](#commands-used)
- [Highlight: verifying partitions from the raw GPT table](#highlight-verifying-partitions-from-the-raw-gpt-table)
- [Skills demonstrated](#skills-demonstrated)
- [Safety and ethics](#safety-and-ethics)
- [Academic integrity](#academic-integrity)
- [Author](#author)
- [Acknowledgements](#acknowledgements)
 
---
 
## Objectives
 
| # | Objective |
|---|-----------|
| 1 | Capture the system specification and storage layout without altering the machine |
| 2 | Map digital-evidence sources to the layers of a computer system |
| 3 | Distinguish a program, a process and a thread, and use live process data (PIDs) |
| 4 | Calculate disk and partition sizes from sector counts and sector size |
| 5 | Inspect the partition table with read-only tools (Disk Management, PowerShell, raw bytes) |
| 6 | Explain the PC boot sequence and compare MBR with GPT |
| 7 | Identify file systems (NTFS, FAT32, ext4) and explain why they matter in forensics |
| 8 | **Verify partitions 1 and 2 from partition-table evidence and compare with the OS** |
 
---
 
## Key findings
 
| Area | Result |
|------|--------|
| Boot mode | **UEFI** (BIOS Mode: UEFI, Secure Boot: Off) |
| Partition style | **GPT** on both internal disks (512-byte logical sectors) |
| Partition 1 (Disk 0) | EFI System, FAT32 volume. Sectors 2,048 to 206,847 = **204,800 sectors = 0.0977 GB (100 MB)** |
| Partition 2 (Disk 0) | Microsoft Reserved (no file system). Sectors 206,848 to 239,615 = **32,768 sectors = 0.0156 GB (16 MB)** |
| Verification | Raw GPT values **match Windows `Get-Partition` exactly** (104,857,600 and 16,777,216 bytes) |
| File systems found | NTFS (C:, D:, Recovery), FAT32 (EFI System partitions), ext4 (Kali on WSL, supplementary) |
| Slide exercise | 400 x 12 x 17,000 sectors x 512 bytes = **38.91 GB** |
 
---
 
## Environment
 
| Item | Value |
|------|-------|
| Machine | CONNEX L1430-PRO laptop |
| OS | Microsoft Windows 10 Home, version 10.0.19045 (build 19045), x64 |
| Processor | Intel Celeron N3350 @ 1.10 GHz, 2 cores |
| Memory | 4.00 GB installed |
| Firmware | American Megatrends BIOS dated 10/10/2020, **UEFI mode** |
| Storage | Disk 0: Hitachi HTS545050B9A300 (465.76 GB, GPT). Disk 1: 58.5 GB (GPT) |
| Tools | System Information (`msinfo32`), Command Prompt, Windows PowerShell, Disk Management, Kali Linux on WSL |
 
---
 
## Repository structure
 
```text
.
├── README.md          # Project overview (this file)
├── REPORT.md          # Full lab report with calculations and evidence
└── screenshots/       # Evidence screenshots referenced by REPORT.md
    ├── A1_msinfo32.png
    ├── A1_systeminfo.png
    ├── A2_storage.png
    ├── B1_processes.png
    ├── C2_get_partition.png
    ├── D1_disk_management.png
    ├── D1_raw_gpt_read.png
    ├── E1_boot_mode.png
    ├── F_file_systems.png
    └── G_os_comparison.png
```
 
---
 
## Methodology
 
1. **Document first, touch nothing.** Only read-only commands were used. No format, delete, resize,
   clean, convert or write operations were run.
2. **Identify** the system (`msinfo32`, `systeminfo`) and storage (`Get-Disk`, `Get-Partition`, `Get-Volume`, `wmic`).
3. **Map** evidence to layers: user, application, operating system, disk, memory/process, hardware/firmware.
4. **Observe** live processes and record names and PIDs.
5. **Calculate** sizes with `size = sectors x bytes per sector`, divided by 1024 three times for GB.
6. **Verify** the first two partitions by reading the GPT partition entries directly from the physical disk (read-only).
7. **Cross-check** against what Windows reports and explain any difference (GB vs GiB, rounding, file-system overhead).
 
---
 
## Commands used
 
```powershell
# System and storage identification
systeminfo
Get-Disk | Select Number,FriendlyName,PartitionStyle,LogicalSectorSize
Get-Partition -DiskNumber 0 | Select PartitionNumber,Type,Offset,Size
Get-Volume
wmic logicaldisk get caption,filesystem,size,freespace,volumename
 
# Process observation
Get-Process | Select-Object -First 20 Name,Id,CPU,WorkingSet
tasklist
 
# Boot mode evidence
msinfo32        # BIOS Mode, Secure Boot State
```
 
```bash
# Linux (Kali on WSL), read-only
df -Th
lsblk -f
sudo blkid
```
 
---
 
## Highlight: verifying partitions from the raw GPT table
 
A GPT disk stores its partition entries starting at **LBA 2 (byte offset 0x400)**. Each entry is
128 bytes. The first LBA is at byte +32 and the last LBA at byte +40. Values are **little-endian**, so
the bytes are reversed before converting from hex.
 
| Partition | Field | Raw bytes (as read) | Reversed | Decimal |
|-----------|-------|---------------------|----------|---------|
| 1 | First LBA | `00 08 00 00` | `00 00 08 00` | 2,048 |
| 1 | Last LBA | `FF 27 03 00` | `00 03 27 FF` | 206,847 |
| 2 | First LBA | `00 28 03 00` | `00 03 28 00` | 206,848 |
| 2 | Last LBA | `FF A7 03 00` | `00 03 A7 FF` | 239,615 |
 
GPT stores the **last** sector, so `sectors = last - first + 1`:
 
```text
Partition 1: 206,847 - 2,048   + 1 = 204,800 sectors
             204,800 x 512 = 104,857,600 bytes
             104,857,600 / 1024 / 1024 / 1024 = 0.0977 GB (100 MB)
 
Partition 2: 239,615 - 206,848 + 1 = 32,768 sectors
             32,768 x 512 = 16,777,216 bytes
             16,777,216 / 1024 / 1024 / 1024 = 0.0156 GB (16 MB)
```
 
The read-only PowerShell used to read the table is included in [REPORT.md](REPORT.md#appendix-a-read-only-script-used-to-read-the-gpt-entries).
 
---
 
## Skills demonstrated
 
- Digital-forensics workflow: read-only acquisition mindset, documentation and verification
- Disk geometry, sectors, partitions and unit conversion (KB, MB, GB, GiB)
- GPT vs MBR structure, little-endian values, hexadecimal-to-decimal conversion
- Windows administration tools: PowerShell, CMD, Disk Management, System Information
- Basic Linux (Kali/WSL) file-system inspection
- Evidence-style technical writing and reporting
 
---
 
## Safety and ethics
 
- All work was performed on **my own computer**.
- Only read-only commands were used. The physical disk was opened for reading only.
- Personal identifiers (serial numbers, e-mail, user and PC names) should be blurred in published screenshots.
  Partition sizes and file-system evidence are kept visible, as the lab requires.
 
---
 
## Academic integrity
 
This repository is a learning record. If you are taking this course, use it to understand the method,
then perform the lab on **your own machine** and write your own report. Your partition table will be different.
 
---
 
## Author
 
**Thobile Ngcobo**
BSc Information Technology | Digital forensics and cybersecurity student
Course: ICDFA CIP-B101, Basic Computer Skills for Digital Forensics
Completed: 07 November 2026
 
---
 
## Acknowledgements
 
Lab manual and slide material: *Computer System Introduction*, International Cybersecurity and
Digital Forensics Academy (ICDFA), CIP-B101.
 
================================================================================
END README.md
=======================================



