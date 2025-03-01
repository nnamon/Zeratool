# Zeratool v2.1
Automatic Exploit Generation (AEG) and remote flag capture for exploitable CTF problems

This tool uses [angr](https://github.com/angr/angr) to concolically analyze binaries by hooking printf and looking for [unconstrained paths](https://github.com/angr/angr-doc/blob/master/docs/examples.md#vulnerability-discovery). These program states are then weaponized for remote code execution through [pwntools](https://github.com/Gallopsled/pwntools) and a series of script tricks. Finally the payload is tested locally then submitted to a remote CTF server to recover the flag.

[![asciicast](https://asciinema.org/a/457964.svg)](https://asciinema.org/a/457964)

## Version 2.1 changes

Zeratool now supports some smart rop chain generation. During a buffer overflow
Zeratool will attempt to leak a libc address and compute the base address and build a execve(/bin/sh,NULL,NULL) chain or system(/bin/sh) chain.

## Installing
Zeratool has been tested on Ubuntu 16.04 through 20.04. Please install [radare2](https://github.com/radareorg/radare2) first

    pip install zeratool
    
## Usage
Zeratool is a python script which accept a binary as an argument and optionally a linked libc library, and a CTF Server connection information

```
[chris:~/Zeratool] zerapwn.py -h
usage: zerapwn.py [-h] [-l LIBC] [-u URL] [-p PORT] [-v] [--force_shellcode]
                  [--format_only] [--overflow_only]
                  file

positional arguments:
  file                  File to analyze

optional arguments:
  -h, --help            show this help message and exit
  -l LIBC, --libc LIBC  libc to use
  -u URL, --url URL     Remote URL to pwn
  -p PORT, --port PORT  Remote port to pwn
  -v, --verbose         Verbose mode
  --force_shellcode     Set overflow pwn mode to point to shellcode
  --format_only         Only run format strings check
  --overflow_only       Only run overflow check
```

## Exploit Types
Zeratool is designed around weaponizing buffer overflows and format string vulnerabilities and currently supports a couple types:

 * Buffer Overflow
   * Point program counter to win function
   * Point program counter to shellcode
   * Point program counter to rop chain
     * Rop chains will attempt to leak a libc function
     * Rop chains will then execve(/bin/sh) or system(/bin/sh)
 * Format String
   * Point GOT entry to win function
   * Point GOT entry to shellcode

## Examples
Checkout the samples.sh file. The file contains several examples of Zeratool automatically solving exploitable CTF problems.


```
#!/bin/bash
# Buffer Overflows with win functions
zerapwn.py tests/bin/bof_win_32
zerapwn.py tests/bin/bof_win_64
# Buffer Overflows with ropping
zerapwn.py tests/bin/bof_nx_32
zerapwn.py tests/bin/bof_nx_64
# Buffer Overflow with ropping and libc leak
zerapwn.py tests/bin/bof_nx_64 -l tests/bin/libc.so.6_amd64

#Format string leak
zerapwn.py tests/bin/read_stack_32
zerapwn.py tests/bin/read_stack_64
#Format string point to win function
zerapwn.py challenges/medium_format
#Format string point to shellcode
#zerapwn.py challenges/hard_format #This one sometimes needs to be run twice

# Buffer overflow point to shellcode
# Turn off aslr 
# echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
zerapwn.py tests/bin/bof_32 --force_shellcode
zerapwn.py tests/bin/bof_64 --force_shellcode
```

[Long Asciinema with Three Solves](https://asciinema.org/a/188001)

## Run the tests!
Tox and Pytest are used to verify that Zeratool is working correctly.
```
tox .
```

## FAQ
Q. Why doesn't Zeratool work against my simple exploitable?

A. Zeratool is held together by scotch tape and dreams. 
