Simple utility to add line numbers to `strace -k` output.

# Why?

`strace -k` prints a stack trace for each syscall.

It can parse binary symbols (if your binary is compiled with ex. `-ggdb`) and show function names + offset inside the function.

The trace is in the following format:
```
brk(0x55e6a1477000)                     = 0x55e6a1477000
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(brk+0xb) [0x11428b]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(__sbrk+0x91) [0x114371]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(__default_morecore+0xd) [0x9cb8d]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x27c5) [0x97575]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x39e3) [0x98793]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x3bcb) [0x9897b]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x4e5e) [0x99c0e]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_file_doallocate+0x94) [0x81d04]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_doallocbuf+0x50) [0x91ed0]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_file_overflow+0x1b0) [0x90f30]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_file_xsputn+0xe5) [0x8f6b5]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(psiginfo+0x13512) [0x76972]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_printf+0xaf) [0x61d3f]
 > /home/user/lt/hello(print_hello+0x25) [0x116e]
 > /home/user/lt/hello(main+0x12) [0x119f]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(__libc_start_main+0xf3) [0x24083]
 > /home/user/lt/hello(_start+0x2e) [0x108e]
```

Unfortunately, if you have access to source code, `strace` does not support translating addresses to line numbers.

This can be done manually using `addr2line`:
```
$ addr2line -e hello 0x116e
/home/user/lt/hello.c:5
```

`linetrace` just does this automatically for every line that matches a user-defined regex filter.
The filter is needed because we have symbols (and want line numbers) only for certain binaries.

# Installation
Install `python3`, `addr2line`. then:

```
sudo cp linetrace /usr/local/bin
sudo chmod +x /usr/local/bin/linetrace
```

# Example

```
$ strace -k ./hello 2>&1 | linetrace hello

...

brk(0x55a2b2425000)                     = 0x55a2b2425000
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(brk+0xb) [0x11428b]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(__sbrk+0x91) [0x114371]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(__default_morecore+0xd) [0x9cb8d]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x27c5) [0x97575]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x39e3) [0x98793]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x3bcb) [0x9897b]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(pthread_attr_setschedparam+0x4e5e) [0x99c0e]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_file_doallocate+0x94) [0x81d04]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_doallocbuf+0x50) [0x91ed0]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_file_overflow+0x1b0) [0x90f30]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_file_xsputn+0xe5) [0x8f6b5]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(psiginfo+0x13512) [0x76972]
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(_IO_printf+0xaf) [0x61d3f]
 > /home/user/lt/hello(print_hello+0x25) [0x116e]
/home/user/lt/hello.c:5
 > /home/user/lt/hello(main+0x12) [0x119f]
/home/user/lt/hello.c:13
 > /usr/lib/x86_64-linux-gnu/libc-2.31.so(__libc_start_main+0xf3) [0x24083]
 > /home/user/lt/hello(_start+0x2e) [0x108e]
```
