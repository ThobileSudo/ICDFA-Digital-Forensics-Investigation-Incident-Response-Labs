Lab 1B: PC System Fundamentals, Disk Sectors and Partition Analysis



Overview

This repository documents the completed Lab 1B assignment for CIP-B101:
Basic Computer Skills for Digital Forensics.

The practical assignment explores core computer-system concepts and
introductory digital-forensics techniques. It covers system
identification, evidence sources, process investigation, disk-sector
calculations, partition analysis, boot architecture, and file-system
interpretation.

The work combines practical system inspection using Windows tools and
Kali Linux with written explanations and calculations.

Assignment Information

  -----------------------------------------------------------------------
  Item                                Details
  ----------------------------------- -----------------------------------
  Course code                         CIP-B101

  Course title                        Basic Computer Skills for Digital
                                      Forensics

  Lab                                 Lab 1B

  Assignment title                    PC System Fundamentals, Disk
                                      Sectors and Partition Analysis

  Student                             Thobile Ngcobo

  Student ID                          C11/26/PCDFIR/17041

  Date completed                      07 November 2026

  Platforms / tools                   Kali Linux, Windows PowerShell,
                                      Windows CMD
  -----------------------------------------------------------------------

Objectives

The assignment demonstrates the ability to:

-   Identify basic computer and storage-system specifications.
-   Recognise digital evidence that may be present across different
    system layers.
-   Inspect running programs and processes and understand process
    identifiers (PIDs).
-   Explain the relationship between programs, processes, and threads.
-   Calculate disk capacity from geometry and sector-size information.
-   Interpret partition entries and calculate partition sizes.
-   Describe the Windows boot process in both legacy BIOS/MBR and
    UEFI/GPT environments.
-   Identify common file systems and explain their relevance to
    digital-forensic investigations.
-   Compare calculated partition information with
    operating-system-reported information.

Assignment Coverage

Part A — System Identification and Evidence Layer Mapping

This section records basic system and storage information and maps
possible evidence sources to different computer-system layers.

Evidence layers considered include:

-   Users: usernames, passwords, and user activity traces.
-   Applications: chat records, browsing activity, and shopping history.
-   Operating system: event logs, registry information, user profiles,
    installed programs, and connected USB devices.
-   Device hardware: serial numbers, boot order, BIOS/UEFI settings, and
    Secure Boot state.
-   Memory (RAM): running programs, process IDs, open network
    connections, and other live information.

The assignment also explains why evidence relating to one activity may
be found across several layers. Cross-checking those traces can help
investigators corroborate findings.

Part B — Program, Process and Thread Investigation

This section examines running processes and records examples with their
process identifiers (PIDs), including:

  -------------------------------------------------------------------------
  Process                                PID recorded Likely purpose
  ---------------------- ---------------------------- ---------------------
  AggregatorHost                                 6928 Windows background
                                                      diagnostics component

  Chrome                                        14980 Web browsing

  ApplicationFrameHost                           5712 Hosts modern Windows
                                                      application windows
  -------------------------------------------------------------------------

It also distinguishes between:

-   Program: a saved set of instructions.
-   Process: a program that has started executing and has an allocated
    process ID and memory space.
-   Thread: a unit of work within a process; a process may contain
    multiple threads.

The section discusses how process information can assist live forensic
triage and malware analysis, including identifying suspicious activity
that may not be apparent from disk information alone.

Part C — Disk Sector and Disk Size Calculation

This section applies sector-based calculations to estimate disk
capacity.

The assignment uses a sector size of 512 bytes and demonstrates
conversions between:

-   Sectors and bytes
-   Bytes and kilobytes
-   Kilobytes and megabytes
-   Megabytes and gigabytes

It also calculates partition sizes using:

Partition size (GB) = Number of sectors × 512 ÷ 1024 ÷ 1024 ÷ 1024

The partition examples documented in the assignment are:

  ---------------------------------------------------------------------------
  Partition   Type            Starting     Number of   Sector size File
                                sector       sectors               system
  ----------- ---------- ------------- ------------- ------------- ----------
  Partition 1 System             2,048       204,800     512 bytes FAT32
              (EFI)                                                

  Partition 2 Reserved         206,848        32,768     512 bytes None
              (MSR)                                                
  ---------------------------------------------------------------------------

The calculated sizes are approximately 0.0977 GB (100 MB) for the EFI
partition and 0.0156 GB (16 MB) for the MSR partition.

Part D — Windows Disk Inspection

The assignment documents Windows-based methods for inspecting disks and
partitions, including:

-   Disk Management
-   Windows PowerShell
-   DiskPart

These tools provide different ways to inspect disk layout, partition
information, and storage configuration.

Part E — MBR, GPT and the Boot Process

This section explains the main stages involved in starting a computer:

1.  Power-on and CPU startup.
2.  Power-On Self-Test (POST).
3.  BIOS/UEFI firmware initialisation.
4.  Identification of a bootable device using the boot order.
5.  Locating the appropriate boot record or EFI boot entry.
6.  Starting the boot loader or boot manager.
7.  Loading the operating-system kernel.
8.  Starting operating-system services and drivers and presenting the
    login screen.

It distinguishes between:

-   Legacy BIOS with MBR: firmware reads the Master Boot Record in
    sector 0 and uses its boot code to locate the active partition.
-   UEFI with GPT: firmware uses a boot entry to load a boot file from
    the EFI System Partition.

The assignment identifies the system used for the practical work as
UEFI-based.

Part F — File-System Identification and Interpretation

The assignment identifies the following file-system usage:

-   NTFS: used by the C: and D: volumes.
-   FAT32: used by the EFI System partition.

It explains the role of a file system in organising raw disk sectors
into files and folders, and discusses forensic relevance, including:

-   File metadata and timestamps.
-   The NTFS Master File Table (MFT).
-   Deleted-file recovery and unallocated space.
-   Slack space and remnants of previous data.
-   Selecting suitable forensic tools and methods for different file
    systems.
-   Using file-system characteristics as clues about a system’s
    configuration.

Part G — Final Comparison

The final section compares partition calculations with information
reported by the operating system.

It notes that the partition-table calculation and operating-system
information agree, while reported volume sizes may differ slightly
because of file-system structures and unit conversion or rounding
conventions.

Tools and Technologies

  Tool / technology    Use in the assignment
  -------------------- ---------------------------------------------------
  Kali Linux           Digital-forensics learning and system environment
  Windows PowerShell   Storage and volume inspection
  Windows CMD          Command-line system inspection
  Disk Management      Graphical disk and partition inspection
  DiskPart             Disk and partition command-line inspection
  NTFS / FAT32         File-system identification and interpretation
  BIOS / UEFI          Firmware and boot-process concepts
  MBR / GPT            Partitioning and boot architecture concepts

Key Learning Outcomes

Through this practical assignment, the following foundational concepts
were explored:

-   A computer can contain useful digital evidence across hardware,
    operating-system, application, user, and memory layers.
-   Running-process information provides a view of system activity at
    the time of examination.
-   Disk capacity and partition sizes can be derived from sector counts
    and sector sizes.
-   MBR and GPT systems use different approaches to locating boot
    information.
-   File systems organise data and preserve metadata that may support
    forensic timelines and recovery work.
-   Comparing command-line output, graphical tools, and manual
    calculations helps validate findings.

Repository Contents

Suggested repository structure:

    CIP-B101-LAB1B/
    ├── README.txt
    └── CIP_B101_LAB1B_THOBILE_NGCOBO_17041.pdf

The PDF is the completed assignment submission. This README provides a
project-style overview of its scope, tools, and learning outcomes.

Author

Thobile Ngcobo
CIP-B101 — Basic Computer Skills for Digital Forensics

------------------------------------------------------------------------

This repository is intended for educational and portfolio purposes. The
practical findings and descriptions are based on the completed Lab 1B
assignment.



