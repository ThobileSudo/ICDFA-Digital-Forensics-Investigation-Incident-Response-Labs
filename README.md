# Lab 1B: PC System Fundamentals, Disk Sectors and Partition Analysis

**Course:** CIP-B101 Basic Computer Skills for Digital Forensics (ICDFA)
**Author:** Thobile Ngcobo
**Date completed:** 7 November 2026
**Platforms and tools:** Kali Linux, Windows PowerShell, Windows CMD, Disk Management, DiskPart



## Overview

This lab covers core computer-system concepts and introductory digital forensics techniques: system identification, evidence sources, process investigation, disk-sector calculations, partition analysis, boot architecture and file-system interpretation. It combines hands-on inspection with Windows tools and Kali Linux with written explanations and calculations.

## What I demonstrated

- Identified basic computer and storage-system specifications.
- Recognised where digital evidence can sit across different system layers.
- Inspected running processes and explained process IDs (PIDs).
- Explained the relationship between programs, processes and threads.
- Calculated disk capacity from geometry and sector size.
- Interpreted partition entries and calculated partition sizes.
- Described the Windows boot process for legacy BIOS/MBR and UEFI/GPT.
- Identified common file systems and why they matter in investigations.
- Compared calculated partition information with what the operating system reports.

## Part A: System identification and evidence layers

Evidence can exist at several layers, and traces of one activity can appear in more than one of them. Cross-checking those traces helps an investigator corroborate findings.

| Layer | Example evidence |
|---|---|
| Users | Usernames, passwords, user activity traces |
| Applications | Chat records, browsing activity, shopping history |
| Operating system | Event logs, registry information, user profiles, installed programs, connected USB devices |
| Device hardware | Serial numbers, boot order, BIOS/UEFI settings, Secure Boot state |
| Memory (RAM) | Running programs, process IDs, open network connections |

## Part B: Programs, processes and threads

| Process | PID recorded | Likely purpose |
|---|---|---|
| AggregatorHost | 6928 | Windows background diagnostics component |
| Chrome | 14980 | Web browsing |
| ApplicationFrameHost | 5712 | Hosts modern Windows application windows |

- **Program:** a saved set of instructions.
- **Process:** a program that has started executing, with a process ID and memory space.
- **Thread:** a unit of work inside a process; a process can have several.

Process information supports live forensic triage and malware analysis, because suspicious activity may not be visible from disk information alone.

## Part C: Disk sector and size calculations

Sector size used: 512 bytes. Partition size in GB = sectors x 512 / 1024 / 1024 / 1024.

| Partition | Type | Starting sector | Number of sectors | File system | Calculated size |
|---|---|---|---|---|---|
| 1 | System | 2,048 | 204,800 | FAT32 (EFI) | about 0.0977 GB (100 MB) |
| 2 | Reserved | 206,848 | 32,768 | None (MSR) | about 0.0156 GB (16 MB) |

## Part D: Windows disk inspection

Disks and partitions were inspected with Disk Management, Windows PowerShell and DiskPart. Each tool shows the layout and storage configuration in a different way.

## Part E: MBR, GPT and the boot process

1. Power-on and CPU startup
2. Power-On Self-Test (POST)
3. BIOS/UEFI firmware initialisation
4. Boot device chosen from the boot order
5. Boot record or EFI boot entry located
6. Boot loader or boot manager started
7. Operating-system kernel loaded
8. Services and drivers started, login screen shown

- **Legacy BIOS with MBR:** firmware reads the Master Boot Record in sector 0 and uses its boot code to find the active partition.
- **UEFI with GPT:** firmware uses a boot entry to load a boot file from the EFI System Partition.

The system used for this lab is UEFI-based.

## Part F: File-system identification

- **NTFS:** the C: and D: volumes
- **FAT32:** the EFI System partition

File systems organise raw sectors into files and folders. Forensic relevance includes metadata and timestamps, the NTFS Master File Table (MFT), deleted-file recovery and unallocated space, slack space, and choosing suitable tools for each file system.

## Part G: Final comparison

The partition-table calculations agree with the information the operating system reports. Small differences in volume sizes come from file-system structures and rounding.

## Key takeaways

- Evidence can sit across hardware, operating-system, application, user and memory layers.
- Running-process information shows system activity at the time of examination.
- Disk and partition sizes can be derived from sector counts.
- MBR and GPT systems locate boot information differently.
- File systems preserve metadata that supports timelines and recovery.
- Comparing command-line output, graphical tools and manual calculations validates findings.

*Educational portfolio. Written from my own completed Lab 1B assignment.*
