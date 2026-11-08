# Lab 1B Report: PC System Fundamentals, Disk Sectors and Partition Analysis
 
| | |
|---|---|
| **Course code** | CIP-B101, Basic Computer Skills for Digital Forensics |
| **Lab** | Lab 1B |
| **Student** | Thobile Ngcobo |
| **Date** | 07 November 2026 |
| **Platform used** | Windows 10 (PowerShell, Command Prompt), Kali Linux on WSL |
| **Approach** | Read-only observation, calculation and verification |
 
---
 
## Contents
 
1. [Introduction](#1-introduction)
2. [Tools used](#2-tools-used)
3. [Safety and evidence handling](#3-safety-and-evidence-handling)
4. [Part A: System identification and evidence layers](#4-part-a-system-identification-and-evidence-layer-mapping)
5. [Part B: Program, process and thread](#5-part-b-program-process-and-thread-investigation)
6. [Part C: Disk sector and disk size calculation](#6-part-c-disk-sector-and-disk-size-calculation)
7. [Part D: Safe partition table inspection](#7-part-d-safe-partition-table-inspection)
8. [Part E: Boot process, MBR and GPT](#8-part-e-boot-process-mbr-and-gpt)
9. [Part F: File system identification](#9-part-f-file-system-identification-and-interpretation)
10. [Part G: Final assignment, verifying partitions 1 and 2](#10-part-g-final-practical-assignment)
11. [Conclusion](#11-conclusion)
12. [Appendix](#appendix)
 
---
 
## 1. Introduction
 
This lab prepares an initial system profile of a computer that could later be used in a forensic
investigation, without changing anything on the machine. The aims were to identify the hardware,
operating system and storage, to map where digital evidence can exist across the layers of a
computer, and to observe running processes. Disk and partition sizes were then calculated from sector
information, and the first two partitions were verified by reading the **raw GPT partition table**
directly from the disk. The boot process, the MBR and GPT partition styles, and the file systems
in use were also explained. Every step followed a read-only, repeatable method.
 
---
 
## 2. Tools used
 
| Category | Tool / command |
|----------|----------------|
| Operating system | Microsoft Windows 10 Home (build 19045), Kali Linux on WSL |
| System information | `msinfo32`, `systeminfo` |
| Storage inspection | `Get-Disk`, `Get-Partition`, `Get-Volume`, `wmic logicaldisk`, Disk Management (`diskmgmt.msc`) |
| Process observation | `Get-Process`, `tasklist` |
| Raw table reading | A read-only PowerShell script (Appendix A) |
| Linux inspection | `df -Th`, `lsblk -f`, `blkid` |
| Documentation | Screenshots, manual calculations |
 
---
 
## 3. Safety and evidence handling
 
- Only the machine owned by the student was examined.
- No command that writes to disk was used: no `format`, `delete`, `clean`, `create`, `convert`, `mkfs` or `dd`.
- The physical disk was opened with **read-only** access (`FileAccess.Read`).
- All findings were recorded with screenshots and written calculations.
 
---
 
## 4. Part A: System identification and evidence layer mapping
 
### A1. System specification
 
![A1 System Information](figA1_basic_system_specification.png)
![A1 systeminfo output](screenshots/figA1_system_info.png)
 
| Item | Value |
|------|-------|
| Manufacturer / model | CONNEX / L1430-PRO |
| OS name | Microsoft Windows 10 Home |
| Version | 10.0.19045, build 19045 |
| System type | x64-based PC |
| Processor | Intel Celeron N3350 @ 1.10 GHz, 2 cores, 2 logical processors |
| Installed physical memory | 4.00 GB (about 3.9 GB usable) |
| BIOS | American Megatrends, version LC-BI-14.1-Y116AR910-560-E, dated 10/10/2020 |
| **BIOS mode** | **UEFI** |
| **Secure Boot state** | **Off** |
| Boot device | `\Device\HarddiskVolume6` |
 
**Observation:** The firmware runs in UEFI mode, which matches the GPT partition style found on the disks in Part D.
 
### A2. Storage information
 
![A2 Storage information](screenshots/A2_storage.png)
 
| Disk | Model | Size | Partition style | Logical sector size |
|------|-------|------|-----------------|---------------------|
| 0 | Hitachi HTS545050B9A300 | 465.76 GB | GPT | 512 bytes |
| 1 | Generic 58.5 GB disk | 58.5 GB | GPT | 512 bytes |
 
`wmic logicaldisk` output (drives with a letter only):
 
| Drive | File system | Size (bytes) | Free (bytes) | Volume name |
|-------|-------------|--------------|--------------|-------------|
| C: | NTFS | 62,691,209,216 | 11,727,814,656 | WIN |
| D: | NTFS | 499,410,743,296 | 385,556,779,008 | (none) |
 
### A3. Mapping evidence sources to system layers
 
| Layer / source | Example evidence | Where it may be found | Why it matters |
|----------------|------------------|-----------------------|----------------|
| **User** | Typed usernames and passwords, opened files, searched terms | Browser saved logins and history, Recent Files, e-mail accounts (Gmail, Outlook) | Links a person's actions to activity on the system |
| **Application** | Chats, shopping history, document metadata | WhatsApp and Temu data folders and databases (for example `AppData`), browser cache | Shows which software created or used the evidence |
| **Operating system** | Logon times, installed programs, user accounts, USB devices connected | Windows Event Logs, the registry, user profile folders | Shows system events and account activity |
| **Disk / storage** | Saved and deleted files, file timestamps, partitions | NTFS metadata (`$MFT`), unallocated space, the GPT partition table | Supports recovery and timeline analysis |
| **Memory / process** | Running programs and PIDs, open network connections, passwords in use | RAM capture, Task Manager, `tasklist` | Shows live activity that disappears at power-off |
| **Hardware / firmware** | Serial numbers, boot order, BIOS/UEFI settings, Secure Boot state | BIOS/UEFI setup, `msinfo32`, Device Manager | Identifies the physical machine and can reveal tampering |
 
**Why evidence can exist in more than one layer.** The course formula is *user activities = applications +
operating system + disk/memory*. One action leaves a trace in every layer it passes through. For a WhatsApp chat:
the application stores the message in its database, the disk holds that file and its timestamps, the OS logs
when the application ran, and RAM holds the content while the app is open. Evidence in several layers lets an
investigator **cross-check** findings, and deleting one trace does not remove the others.
 
---
 
## 5. Part B: Program, process and thread investigation
 
### B1. Observed processes
 
![B1 Process list](screenshots/B1_processes.png)
 
Command used: `Get-Process | Select-Object -First 20 Name,Id,CPU,WorkingSet`
 
| Process name | PID | Likely purpose |
|--------------|-----|----------------|
| AggregatorHost | 6928 | Windows background diagnostics component |
| chrome | 14980 | Google Chrome web browser (one of many Chrome processes) |
| ApplicationFrameHost | 5712 | Windows component that hosts the windows of modern apps |
 
PID values are specific to this session and change after a restart. Chrome appears many times in the
list, each with a different PID, which shows one application running as many processes.
 
### B2. Program, process and thread
 
- **Program:** a file of instructions stored on disk. By itself it does nothing.
- **Process:** a program that has been started. The OS loads it into RAM, gives it its own memory space and
  assigns it a process ID (PID). Starting the same program twice creates two processes.
- **Thread:** a smaller unit of work inside a process. One process can contain several threads that run
  concurrently and share the process's memory. The OS scheduler decides which thread uses the CPU next.
 
**Why process information is useful in malware analysis and live triage.** Process information shows
what the computer is doing at that moment. Malware that exists only in RAM leaves no file on disk, so the process
list may be the only place it can be seen, and that evidence is lost at shutdown. The PID identifies one running
instance whose memory, network connections and files can be examined. An unusual name, location or parent
process can reveal malicious activity, and the process leads back to the program file for further analysis.
 
---
 
## 6. Part C: Disk sector and disk size calculation
 
### C1. Slide calculation
 
Given: sectors per track = 400, heads = 12, cylinders = 17,000, bytes per sector = 512.
 
| Item | Formula | Answer |
|------|---------|--------|
| Total sectors | 400 x 12 x 17,000 | **81,600,000 sectors** |
| Total bytes | 81,600,000 x 512 | **41,779,200,000 bytes** |
| Kilobytes | 41,779,200,000 / 1024 | **40,800,000 KB** |
| Megabytes | 40,800,000 / 1024 | **39,843.75 MB** |
| Gigabytes | 39,843.75 / 1024 | **38.91 GB** |
 
**Cross-check:** 81,600,000 / 2,097,152 = 38.91 GB (because 1024 x 1024 x 1024 / 512 = 2,097,152).
The total size of the disk is about **38.91 GB**.
 
*Note:* dividing by 1024 gives GiB, which most tools label "GB". Manufacturers divide by 1000, so
advertised sizes can differ slightly.
 
### C2. Partition size calculation (own disk)
 
![C2 Get-Partition output](screenshots/C2_get_partition.png)
 
Formula: `partition size in GB = sector count x 512 / 1024 / 1024 / 1024`
 
Start sector = offset in bytes / 512 (Windows reports offsets in bytes).
 
| | Partition 1 | Partition 2 |
|---|-------------|-------------|
| Type | System (EFI) | Reserved (MSR) |
| Offset (bytes) | 1,048,576 | 105,906,176 |
| Start sector | 2,048 | 206,848 |
| Number of sectors | 204,800 | 32,768 |
| Sector size | 512 bytes | 512 bytes |
| File system | FAT32 | None |
 
Start sectors:
 
- Partition 1: 1,048,576 / 512 = **2,048**
- Partition 2: 105,906,176 / 512 = **206,848**
 
Sizes:
 
```text
Partition 1:  204,800 x 512 = 104,857,600 bytes
              104,857,600 / 1024 / 1024 / 1024 = 0.0977 GB  (100 MB)
 
Partition 2:  32,768 x 512 = 16,777,216 bytes
              16,777,216 / 1024 / 1024 / 1024 = 0.0156 GB  (16 MB)
```
 
---
 
## 7. Part D: Safe partition table inspection
 
### D1. Windows method (read-only)
 
![D1 Disk Management](screenshots/D1_disk_management.png)
 
Disk Management shows the layout of both disks:
 
| Disk | Layout |
|------|--------|
| Disk 0 (465.75 GB) | 100 MB EFI System partition, 465.11 GB NTFS data volume (D:), 546 MB Recovery partition |
| Disk 1 (58.48 GB) | 100 MB EFI System partition, 58.39 GB NTFS Windows volume (C:) |
 
`Get-Partition` for Disk 0:
 
| Partition | Type | Offset (bytes) | Size (bytes) |
|-----------|------|----------------|--------------|
| 1 | System | 1,048,576 | 104,857,600 |
| 2 | Reserved | 105,906,176 | 16,777,216 |
| 3 | Basic (D:) | 122,683,392 | 499,410,746,368 |
| 4 | Recovery | 499,534,266,368 | 572,522,496 |
 
### Raw GPT partition table (read-only)
 
![D1 Raw GPT read](screenshots/D1_raw_gpt_read.png)
 
Because the lab requires partition-table evidence, the GPT entries were read straight from the physical disk
with a read-only script (Appendix A). Sector 1 begins with the text **`EFI PART`**, which confirms a valid GPT header.
The partition entries start at sector 2 (byte offset 0x400); each entry is 128 bytes.
 
| Entry | First LBA bytes | Last LBA bytes | First LBA | Last LBA |
|-------|-----------------|----------------|-----------|----------|
| 1 | `00 08 00 00 00 00 00 00` | `FF 27 03 00 00 00 00 00` | 2,048 | 206,847 |
| 2 | `00 28 03 00 00 00 00 00` | `FF A7 03 00 00 00 00 00` | 206,848 | 239,615 |
 
Values are stored **little-endian**, so the bytes are reversed before converting. For example
`00 08 00 00` becomes `00 00 08 00` = 0x800 = 2,048, and `FF 27 03 00` becomes `00 03 27 FF` = 0x327FF = 206,847.
 
---
 
## 8. Part E: Boot process, MBR and GPT
 
### E1. Boot sequence
 
1. **Power-on:** electricity reaches the motherboard and the CPU starts running the firmware.
2. **POST (power-on self-test):** the firmware checks the CPU, RAM, keyboard and storage devices.
3. **BIOS/UEFI starts:** it loads its settings from **CMOS** (battery-backed memory holding the boot order,
   date and time). UEFI keeps similar settings in non-volatile memory.
4. **Boot device selection:** the firmware tries devices in the boot order until one can boot.
5. **Boot record:**
   - *Legacy BIOS with MBR:* the firmware reads sector 0 (the MBR), whose boot code finds the active partition.
   - *UEFI with GPT:* the firmware loads a boot file from the **EFI System Partition** (a small FAT32 partition).
6. **Boot loader / manager:** on Windows, `bootmgr` or `bootmgfw.efi` lets the user choose an OS and prepares to load it.
7. **Kernel loading:** the boot manager loads the OS kernel (`ntoskrnl.exe` on Windows) from the boot volume, normally C:.
8. **OS control:** the kernel starts services and drivers and the login screen appears.
 
```text
Power-on -> POST -> BIOS/UEFI (CMOS settings) -> Boot device -> MBR or EFI partition
         -> Boot manager -> Kernel -> Operating system
```
 
| Stage | What happens | Forensic relevance |
|-------|--------------|--------------------|
| Power-on and POST | Hardware is initialised and checked | Hardware failure or tampering may affect evidence access |
| BIOS/UEFI firmware | Boot order and firmware settings are applied | Boot mode explains partition style and boot artifacts |
| MBR/GPT or EFI boot | The bootable partition or EFI entry is located | Boot records may hold evidence of boot configuration |
| Boot loader / manager | OS selection and pre-kernel loading | May show multi-boot or hidden OS behaviour |
| Kernel and OS control | The operating system fully starts | Logs and user activity begin to accumulate |
 
### Boot mode of this system
 
![E1 Boot mode evidence](screenshots/E1_boot_mode.png)
 
**This system uses UEFI.** System Information reports *BIOS Mode: UEFI* (Secure Boot State: Off), and `Get-Disk`
shows both disks as **GPT**, each with a 100 MB EFI System partition formatted FAT32.
 
### E2. MBR compared with GPT
 
| Feature | MBR | GPT |
|---------|-----|-----|
| Age and design | Older scheme from the 1980s | Modern scheme used with UEFI |
| Size limit | About 2 TB (32-bit sector counts) | Very large disks (about 9.4 ZB at 512-byte sectors) |
| Partitions | 4 primary entries | Up to 128 entries by default |
| Where it is stored | Sector 0, with the boot code | Primary header (LBA 1) and entries (LBA 2 to 33), plus a backup at the end of the disk |
| Forensic issue | Damage to sector 0 can make the layout unreadable | The backup copy and checksums help validation and recovery |
 
---
 
## 9. Part F: File system identification and interpretation
 
![F Get-Volume and wmic output](screenshots/F_file_systems.png)
 
| Volume | Name | File system | Size | Role |
|--------|------|-------------|------|------|
| C: | WIN | **NTFS** | 58.39 GB | Windows system drive |
| D: | (none) | **NTFS** | 465.11 GB | Data drive |
| no letter | BOOT | **FAT32** | 96 MB | EFI System partition |
| no letter | (none) | **NTFS** | 546 MB | Recovery partition |
| no letter | (none) | **FAT32** | 96 MB | Second EFI System partition |
 
*Supplementary (Kali Linux on WSL):* `df -Th` shows the Linux root `/` on `/dev/sdd` as **ext4** (a virtual disk
of about 1 TB maximum). The Windows drives C: and D: appear as `9p` mounts, which is only the WSL bridge; their real
file system is NTFS.
 
**Why the file systems differ.** NTFS is the standard Windows file system and provides timestamps, permissions and a
change journal. The EFI partition uses FAT32 because UEFI firmware can only read simple file systems at start-up.
 
### Why a file system is important in digital forensics
 
A file system is the set of rules that turns raw sectors into named files and folders. It decides how files are
stored and found, so an examiner must identify it before any analysis. It records metadata such as creation,
modification and access times (NTFS keeps this in the Master File Table), which is used to build timelines. It also
determines what happens on deletion: the data usually remains until overwritten, so deleted files can often be
recovered. Slack and unallocated space can hold leftovers of older data. Finally, NTFS, FAT32 and ext4 differ, so the
file system guides the tools and methods used and prevents wrong conclusions.
 
---
 
## 10. Part G: Final practical assignment
 
**Task:** verify the size of partitions 1 and 2 using the partition table, showing each step and the size in GB.
 
![G OS comparison](screenshots/G_os_comparison.png)
 
### Step by step
 
| Step | Action | Result |
|------|--------|--------|
| 1 | `Get-Disk` confirms Disk 0 is **GPT** with **512-byte** sectors | Formulas use 512 |
| 2 | Read the raw disk. Sector 1 starts with `EFI PART` | Valid GPT header |
| 3 | Read the entries at byte 0x400 (entry 1) and 0x480 (entry 2) | Raw bytes recorded (Part D) |
| 4 | Convert little-endian bytes to decimal | See table below |
| 5 | Compute sectors = last LBA - first LBA + 1 (GPT stores the last sector) | See table below |
| 6 | Compute bytes = sectors x 512, then divide by 1024 three times | Sizes in GB |
| 7 | Compare with `Get-Partition`, `Get-Volume` and Disk Management | Match (see below) |
 
| Task | Partition 1 | Partition 2 |
|------|-------------|-------------|
| Identification | System (EFI) partition, Disk 0 | Microsoft Reserved partition, Disk 0 |
| Start sector | 2,048 | 206,848 |
| End sector | 206,847 | 239,615 |
| Sector count | 206,847 - 2,048 + 1 = **204,800** | 239,615 - 206,848 + 1 = **32,768** |
| Size in bytes | 204,800 x 512 = 104,857,600 | 32,768 x 512 = 16,777,216 |
| **Size in GB** | 104,857,600 / 1024 / 1024 / 1024 = **0.0977 GB** (100 MB) | 16,777,216 / 1024 / 1024 / 1024 = **0.0156 GB** (16 MB) |
| OS-reported size (`Get-Partition`) | 104,857,600 bytes (100 MB) | 16,777,216 bytes (16 MB) |
| Match | **Exact** | **Exact** |
 
### Explanation of differences
 
There is **no difference** between the partition-table calculation and the OS-reported partition sizes, because both
come from the same table. `Get-Volume` shows the FAT32 volume in partition 1 as 96 MB because the file system uses
part of the partition for its own structures. Dividing by 1024 gives GiB, which Windows labels "GB", and tools also
round their output. The Reserved partition has no file system, so `Get-Volume` does not list it.
 
*Extra check (partition 3, D:):* the table shows 499,410,746,368 bytes = 975,411,614 sectors = 465.11 GB. A small gap of
1,634 sectors lies between this partition and the Recovery partition, so partition sizes must be read from the table
rather than estimated by subtracting start sectors.
 
---
 
## 11. Conclusion
 
The lab showed that a computer can be profiled completely without modifying it. The system runs Windows 10 in
**UEFI** mode on two **GPT** disks. Evidence can be found in the user, application, operating-system, disk,
memory and firmware layers, and the same action usually leaves traces in several of them, which allows findings to
be cross-checked. Disk and partition sizes were calculated from sector counts, and the first two partitions
(100 MB EFI and 16 MB Reserved) were **verified from the raw GPT table and matched the OS exactly**. This matters
in digital forensics because conclusions must come from verifiable evidence, gathered with read-only methods, and
documented so that another examiner can repeat the steps and reach the same result.
 
---
 
## Appendix
 
### Appendix A: read-only script used to read the GPT entries
 
The script only reads. It opens the physical disk with `FileAccess.Read` and writes nothing.
Run in an administrator PowerShell. `PhysicalDrive0` is Disk 0.
 
```powershell
$fs = New-Object System.IO.FileStream("\\.\PhysicalDrive0",[IO.FileMode]::Open,[IO.FileAccess]::Read,[IO.FileShare]::ReadWrite)
$buf = New-Object byte[] 2048
$null = $fs.Read($buf,0,2048)
$fs.Close()
"GPT header text at byte 512: " + [Text.Encoding]::ASCII.GetString($buf,512,8)
foreach($i in 0..1){
  $o = 1024 + 128*$i
  $first = [BitConverter]::ToUInt64($buf,$o+32)
  $last  = [BitConverter]::ToUInt64($buf,$o+40)
  "Entry $($i+1) at byte $o"
  "  first LBA bytes: " + (($buf[($o+32)..($o+39)] | ForEach-Object { $_.ToString('X2') }) -join ' ')
  "  last LBA bytes : " + (($buf[($o+40)..($o+47)] | ForEach-Object { $_.ToString('X2') }) -join ' ')
  "  first LBA = $first ; last LBA = $last ; sectors = " + ($last - $first + 1)
}
```
 
Output obtained:
 
```text
GPT header text at byte 512: EFI PART
Entry 1 at byte 1024
  first LBA bytes: 00 08 00 00 00 00 00 00
  last LBA bytes : FF 27 03 00 00 00 00 00
  first LBA = 2048 ; last LBA = 206847 ; sectors = 204800
Entry 2 at byte 1152
  first LBA bytes: 00 28 03 00 00 00 00 00
  last LBA bytes : FF A7 03 00 00 00 00 00
  first LBA = 206848 ; last LBA = 239615 ; sectors = 32768
```
 
### Appendix B: screenshot checklist
 
| # | Screenshot | Included |
|---|------------|----------|
| 1 | System Information / `systeminfo` | Yes |
| 2 | Disk model and size output | Yes |
| 3 | Logical volume / file-system output | Yes |
| 4 | Running process / PID output | Yes |
| 5 | Disk Management, `Get-Partition` and raw table output | Yes |
| 6 | Partition 1 calculation evidence | Yes |
| 7 | Partition 2 calculation evidence | Yes |
| 8 | Boot mode evidence (UEFI) | Yes |
| 9 | File-system identification evidence | Yes |
| 10 | Report summary / conclusion | Yes |
 
### Appendix C: formula summary
 
| Purpose | Formula |
|---------|---------|
| Total sectors from geometry | Cylinders x Heads x Sectors per track |
| Size in bytes | Sectors x bytes per sector |
| Size in GB (GiB) | Bytes / 1024 / 1024 / 1024, or for 512-byte sectors: sectors / 2,097,152 |
| GPT partition sectors | Last LBA - First LBA + 1 |
| Start sector from a Windows offset | Offset (bytes) / 512 |
 
