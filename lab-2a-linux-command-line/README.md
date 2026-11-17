# Lab 2A: Linux Command Line for Digital Forensics Basics

**Course:** CIP-B101 Basic Computer Skills for Digital Forensics (ICDFA)
**Author:** Thobile Ngcobo
**Environment:** Kali GNU/Linux Rolling 2026.3 on WSL2 (Windows), kernel 6.18.54.1-microsoft-standard-WSL2

## Overview

I built an evidence workspace on the Linux command line and practised the commands a forensic analyst uses every day: navigating the file system, reading file metadata, searching evidence, checking network settings safely, scripting and archiving.

## What I did

| Task | Commands | Result |
|---|---|---|
| 1. Workspace | `mkdir -p`, `cd`, `pwd`, `ls -la` | Created notes, evidence, scripts, outputs, screenshots and archive folders with one command |
| 2. OS and file system info | `ls /`, `cat /etc/os-release`, `uname -a`, `whoami`, `id` | Kali Rolling 2026.3 running on WSL2; /proc and /sys are virtual (size 0) |
| 3. Inodes and disk | `ls -li`, `stat`, `df -h`, `df -i` | Evidence file inode 13742, 55 bytes, permissions 0644; 36,402 of 67,108,864 inodes used on / |
| 4. Paths | `cd`, `pwd`, `readlink -f` | Navigated with absolute and relative paths and resolved the full path of a file |
| 5. File management | `echo`, `cat`, `cp -r`, `mv`, `rm` | Created, copied, renamed and deleted files and folders |
| 6. Log searching | `grep`, `grep -n`, `grep -Ei` | Found a failed admin login at 10:05, followed by a password change at 10:40 |
| 7. File discovery | `tree`, `find -name`, `-iname`, `-type` | Located files by name, case-insensitive name and type |
| 8. Network (localhost only) | `ip addr`, `ip route`, `ping`, `ss -lnt`, `nmap localhost` | Loopback 127.0.0.1; no listening ports; ports 22, 80 and 443 closed |
| 9. Script | `nano`, `chmod +x`, `./script.sh` | Ran a system-summary script that writes date, user, kernel, disk, inode usage and a folder tree to a file |
| 10. PATH | `export PATH`, `cd /tmp` | Ran the script by name from another folder after adding its folder to PATH |
| 11. Archive | `zip -r`, `tar -czvf` | Compressed the workspace (zip 641K) |

## Forensic takeaways

- **Inodes** hold a file's metadata and data pointers but not its name, so a renamed file keeps the same inode number, and its timestamps support timelines.
- **grep** turns a log into a short list of events to examine.
- **PATH** decides which program runs when a command name is typed.
- **Localhost-only testing** keeps scanning on systems I own and am authorised to test.

## Notes

The lab sheet supplied the log content and the summary script. I typed, ran and tested them myself. Only localhost was scanned.
