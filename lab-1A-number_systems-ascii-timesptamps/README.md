# Lab 1A: Number Systems, ASCII, Timestamps and Data Representation

**Course:** CIP-B101 Basic Computer Skills for Digital Forensics (ICDFA)
**Author:** Thobile Ngcobo
**Date completed:** 7 November 2026
**Environment:** Kali Linux (WSL2) with bash, Python 3, `xxd` and `bc`

## Overview

Digital evidence is stored as bytes, and what those bytes mean depends on how they are interpreted. In this lab I read the same data as numbers, text, timestamps and hashes, and checked every manual calculation with command-line tools.

## What I did

### 1. Number conversions

Each conversion was worked out by hand first, then verified with the shell and Python.

| Value | Conversion | Result | Verified with |
|---|---|---|---|
| 101010 (binary) | to decimal | 42 | bash arithmetic (`2#101010`), Python `int(..., 2)` |
| AB2 (hex) | to decimal | 2738 | bash arithmetic (`16#AB2`), Python `int(..., 16)` |
| 126 (decimal) | to binary | 1111110 | `bc` with `obase=2`, Python `bin()` |
| 2738 (decimal) | to hex | AB2 | `printf "%X"`, Python `hex()` |
| 101010110010 (binary) | to hex | AB2 | `bc` with `ibase=2` and `obase=16`, Python `hex(int(...))` |

Grouping the last binary value in fours from the right gives `1010 1011 0010`, which is `A B 2`.

### 2. Text as bytes (ASCII and hex)

- Created `Hello` with `echo -n` and viewed its bytes with `xxd`: `48 65 6c 6c 6f`.
- Converted hex back to text with `xxd -r -p`, `printf` and Python's `binascii`.
- `echo` without `-n` adds a newline byte, `0a`, which `xxd` shows as a dot in the text column.
- Hexadecimal is only a notation for bytes. `48 65 6c 6c 6f` decodes to "Hello", but `89 50 4e 47` is the signature of a PNG image and means nothing as text. An examiner has to identify the data type before interpreting it.

### 3. Timestamps

| Value | Format | Converted (UTC) |
|---|---|---|
| 1643643491 | Unix seconds | 2022-01-31 15:38:11 |
| 1675008832 | Unix seconds | 2023-01-29 16:13:52 |
| 725846400 | Mac Absolute Time | 2024-01-02 00:00:00 |

Unix time counts from 1 January 1970 and Mac Absolute Time from 1 January 2001, so a Mac value needs the 978,307,200-second offset added before `date -u -d @...` can read it. Treating it as a Unix value would place the event about 31 years too early. I also compared local time (SAST, UTC+2) with UTC using `date` and `date -u`.

### 4. Hashing and a single byte

I created two files that look identical on screen, one with `echo` and one with `echo -n`.

| | With newline | Without newline |
|---|---|---|
| Bytes | `48 65 6c 6c 6f 0a` (6 bytes) | `48 65 6c 6c 6f` (5 bytes) |
| MD5 | `09f7e02f1290be211da707a266f153b3` | `8b1a9953c4611296a827abf8c47804d7` |

The SHA-256 values differ as well. Hash functions process every byte, so one extra byte changes the result completely. To hash exactly the visible characters, use `echo -n` or `printf`.

### 5. Byte order (endianness)

| Raw bytes | Little-endian | Big-endian |
|---|---|---|
| 01000000 | 1 | 16,777,216 |
| FF000000 | 255 | 4,278,190,080 |
| 00000100 | 65,536 | 256 |
| 78563412 | 305,419,896 | 2,018,915,346 |

The same four bytes give different numbers depending on the order they are read in, so an examiner must know which order the source format uses.

### 6. Mini forensic case

Five evidence items, each interpreted and verified with a command:

| Item | Value | Interpretation |
|---|---|---|
| EVID-001 | `48656c6c6f204943444641` | Hex-encoded ASCII: "Hello ICDFA" |
| EVID-002 | `01001000 01101001` | Two bytes (72, 105): "Hi" |
| EVID-003 | 1675008832 | Unix timestamp: 2023-01-29 16:13:52 UTC |
| EVID-004 | 725846400 | Mac Absolute Time: 2024-01-02 00:00:00 UTC |
| EVID-005 | 78563412 | 305,419,896 little-endian or 2,018,915,346 big-endian |

## Takeaways

- The same bytes can be text, a number or a timestamp depending on context.
- Using the wrong epoch, or mixing UTC and local time, can put an event decades or hours away from when it really happened.
- A single byte, like a trailing newline, changes a hash and can break an integrity check.

## Notes

Educational portfolio. The lab sheet supplied the exercises and sample evidence values; the working, commands and explanations are my own.
