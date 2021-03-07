The challenge is a flag validator, performing multiple syscall `777`s with blocks of the input as parameters and checking whether `errno` was set after any such call.

Looking through the initialization functions (before `main`), one of the functions copies data to the stack and then calls `prctl` for `SET_SECCOMP` with mode `SECCOMP_MODE_FILTER`. This loads a Berkeley Packet Filter, which filters any system calls performed. The instructions for the filter can be dumped directly from the memory. The filter can then be disassembled (I wrote my own disassembler):

```python3
class instr:
    def __init__(self, a):
        self.inst = a[0] + a[1] * 0x100
        self.jt = a[2]
        self.jf = a[3]
        self.k = a[4] + a[5] * 0x100 + a[6] * 0x10000 + a[7] * 0x1000000

    def toLine(self, line):
        instclasses = ["LD", "LDX", "ST", "STX", "ALU", "JMP", "RET", "MISC"]
        ic = self.inst & 0x07
        s = instclasses[ic]
        srcm = {0x0: "K", 0x8: "X", 0x10: "A"}
        if ic in (0, 1): #LD(X)
            size = self.inst & 0x18
            sm = {0x0: "W", 0x8: "H", 0x10: "B"}
            s += sm[size]

            mode = self.inst & 0xe0
            mm = {0x0: "IMM", 0x20: "ABS", 0x40: "IND", 0x60: "MEM", 0x80: "LEN", 0xa0: "MSH"}
            mv = mm[mode]
            if mv == "IMM":
                mv = hex(self.k)
            elif mv == "ABS":
                mv = "D[" + hex(self.k) + "]"
            elif mv == "IND":
                mv = "D[" + hex(self.k) + " + x]"
            elif mv == "MEM":
                mv = "M[" + hex(self.k) + "]"
            
            s += " " + mv
        elif ic in (2, 3): # ST
            s += " M[" + hex(self.k) + "]"
        elif ic == 4: #ALU
            opm = ["ADD", "SUB", "MUL", "DIV", "OR", "AND", "LSH", "RSH", "NEG", "MOD", "XOR"]
            op = self.inst & 0xf0
            s = opm[op >> 4]
            src = self.inst & 0x08
            sv = srcm[src]
            if sv == "K":
                sv = hex(self.k)
            s += " " + sv
        elif ic == 5: # JMP
            jm = ["JA", "JEQ", "JGT", "JGE", "JSET"]
            jmode = self.inst & 0xf0
            jv = jm[jmode >> 4]
            s = jv
            js = self.inst & 0x08
            if jv not in ("JA",):
                jsv = srcm[js]
                if jsv == "K":
                    jsv = hex(self.k)
                s += " " + jsv
            s += " ->" + hex(self.jt + line + 1)
            if self.jf != 0:
                s += " !->" + hex(self.jf + line + 1)
        elif ic == 6: # RET
            src = self.inst & 0x18
            sv = srcm[src]
            if sv == "K":
                sv = hex(self.k)
                retm = {0x7fff0000: "ALLOW", 0x00050000: "ERRNO 0", 0x00050001: "ERRNO 1"}
                if self.k in retm:
                    sv += " " + retm[self.k]
            s += " " + sv
            
        return hex(line)[2:].rjust(2, '0') + " " + s
        
# "data" contains the dumped memory in the format
# 00 00 00 00 00 00 00 00
# for each instruction
with open("data") as f:
    l = [instr([int(x, 16) for x in li.split()]) for li in f.readlines()]
    
for i, ins in enumerate(l):
    print(ins.toLine(i))
```

The output (with comments `#`):

```
# allow if the system call isn't 777
00 LDW D[0x0]
01 JEQ 0x309 ->0x2 !->0xca

# check that all the bytes in the first parameter
# are less than 0x80
02 LDW D[0x10]
03 AND 0xff
04 JGE 0x80 ->0xcc
05 LDW D[0x10]
06 RSH 0x8
07 AND 0xff
08 JGE 0x80 ->0xcc
09 LDW D[0x10]
0a RSH 0x10
0b AND 0xff
0c JGE 0x80 ->0xcc
0d LDW D[0x10]
0e RSH 0x18
0f AND 0xff
10 JGE 0x80 ->0xcc

# M[0] = args[0]
11 LDW D[0x10]
12 ST M[0x0]
# M[1] = args[1] ^ M[0]
13 LDW D[0x18]
14 LDXW M[0x0]
15 XOR X
16 ST M[0x1]
# M[2] = args[1] ^ M[0]
17 LDW D[0x20]
18 LDXW M[0x1]
19 XOR X
1a ST M[0x2]
# M[3] = args[2] ^ M[1]
1b LDW D[0x28]
1c LDXW M[0x2]
1d XOR X
1e ST M[0x3]

# M[4] = M[0] + M[1] + M[2] + M[3]
1f LDW M[0x0]
20 LDXW M[0x1]
21 ADD X
22 LDXW M[0x2]
23 ADD X
24 LDXW M[0x3]
25 ADD X
26 ST M[0x4]
# M[5] = M[0] - M[1] + M[2] - M[3]
27 LDW M[0x0]
28 LDXW M[0x1]
29 SUB X
2a LDXW M[0x2]
2b ADD X
2c LDXW M[0x3]
2d SUB X
2e ST M[0x5]
# M[6] = M[0] + M[1] - M[2] - M[3]
2f LDW M[0x0]
30 LDXW M[0x1]
31 ADD X
32 LDXW M[0x2]
33 SUB X
34 LDXW M[0x3]
35 SUB X
36 ST M[0x6]
# M[7] = M[0] - M[1] - M[2] + M[3]
37 LDW M[0x0]
38 LDXW M[0x1]
39 SUB X
3a LDXW M[0x2]
3b SUB X
3c LDXW M[0x3]
3d ADD X
3e ST M[0x7]

# M[8] = (M[4] | M[5]) ^ (M[6] & M[7])
3f LDW M[0x4]
40 LDXW M[0x5]
41 OR X
42 ST M[0x8]
43 LDW M[0x6]
44 LDXW M[0x7]
45 AND X
46 LDXW M[0x8]
47 XOR X
48 ST M[0x8]
# M[9] = (M[5] | M[6]) ^ (M[7] & M[4])
49 LDW M[0x5]
4a LDXW M[0x6]
4b OR X
4c ST M[0x9]
4d LDW M[0x7]
4e LDXW M[0x4]
4f AND X
50 LDXW M[0x9]
51 XOR X
52 ST M[0x9]
# M[10] = (M[6] | M[7]) ^ (M[4] & M[5])
53 LDW M[0x6]
54 LDXW M[0x7]
55 OR X
56 ST M[0xa]
57 LDW M[0x4]
58 LDXW M[0x5]
59 AND X
5a LDXW M[0xa]
5b XOR X
5c ST M[0xa]
# M[11] = (M[7] | M[4]) ^ (M[5] & M[6])
5d LDW M[0x7]
5e LDXW M[0x4]
5f OR X
60 ST M[0xb]
61 LDW M[0x5]
62 LDXW M[0x6]
63 AND X
64 LDXW M[0xb]
65 XOR X
66 ST M[0xb]

# various jumps based on M[8]..M[11]
67 LDW M[0x8]
68 JEQ 0xf5ffc1f6 ->0x8e
69 JEQ 0x7344aeee ->0x7e
6a JEQ 0xfda6effe ->0x8a
6b JEQ 0x638f7ca2 ->0x76
6c JEQ 0xa2285400 ->0x7c
6d JEQ 0x8990fefe ->0x88
6e JEQ 0x9f576dd4 ->0x8c
6f JEQ 0xf6b9ebe2 ->0x7a
70 JEQ 0xf9e28bee ->0x80
71 JEQ 0x1f9b8fb4 ->0x86
72 JEQ 0xefec86de ->0x90
73 JEQ 0xdf60093a ->0x82
74 JEQ 0xb8af3fbe ->0x78
75 JEQ 0x7f01bbcc ->0x84
76 LDW M[0x9]
77 JEQ 0xf1cf5c2e ->0xaa !->0xcc
78 LDW M[0x9]
79 JEQ 0xb6af7dbe ->0x9a !->0xcc
7a LDW M[0x9]
7b JEQ 0xd6b9bde2 ->0x94 !->0xcc
7c LDW M[0x9]
7d JEQ 0x60fad508 ->0xa0 !->0xcc
7e LDW M[0x9]
7f JEQ 0x77600ede ->0xa2 !->0xcc
80 LDW M[0x9]
81 JEQ 0xf3b68ece ->0x9e !->0xcc
82 LDW M[0x9]
83 JEQ 0x4fe90926 ->0xa8 !->0xcc
84 LDW M[0x9]
85 JEQ 0x7e1933ac ->0x92 !->0xcc
86 LDW M[0x9]
87 JEQ 0x1f9b8fb4 ->0xac !->0xcc
88 LDW M[0x9]
89 JEQ 0xcb94e7da ->0xa6 !->0xcc
8a LDW M[0x9]
8b JEQ 0xb9c2adfe ->0x96 !->0xcc
8c LDW M[0x9]
8d JEQ 0xf01b94c ->0x9c !->0xcc
8e LDW M[0x9]
8f JEQ 0xf5efe5f6 ->0xa4 !->0xcc
90 LDW M[0x9]
91 JEQ 0xa7ad8d4e ->0x98 !->0xcc
92 LDW M[0xa]
93 JEQ 0x7efd33a4 ->0xc2 !->0xcc
94 LDW M[0xa]
95 JEQ 0xd6f33dda ->0xba !->0xcc
96 LDW M[0xa]
97 JEQ 0xbbdaa5e6 ->0xbe !->0xcc
98 LDW M[0xa]
99 JEQ 0x24a7ad2e ->0xbc !->0xcc
9a LDW M[0xa]
9b JEQ 0xb7fdfcbe ->0xc6 !->0xcc
9c LDW M[0xa]
9d JEQ 0xf01b94c ->0xae !->0xcc
9e LDW M[0xa]
9f JEQ 0xb3bdaed6 ->0xb2 !->0xcc
a0 LDW M[0xa]
a1 JEQ 0x60ffd7bc ->0xc4 !->0xcc
a2 LDW M[0xa]
a3 JEQ 0x5f785fd2 ->0xb0 !->0xcc
a4 LDW M[0xa]
a5 JEQ 0x27aeff3e ->0xb8 !->0xcc
a6 LDW M[0xa]
a7 JEQ 0xc39dc1ca ->0xb6 !->0xcc
a8 LDW M[0xa]
a9 JEQ 0x4d8f1f86 ->0xc8 !->0xcc
aa LDW M[0xa]
ab JEQ 0x99ff4c6e ->0xc0 !->0xcc
ac LDW M[0xa]
ad JEQ 0xe97d7d54 ->0xb4 !->0xcc
ae LDW M[0xb]
af JEQ 0x9f576dd4 ->0xcb !->0xcc
b0 LDW M[0xb]
b1 JEQ 0x5b5cffe2 ->0xcb !->0xcc
b2 LDW M[0xb]
b3 JEQ 0xb9e9abf6 ->0xcb !->0xcc
b4 LDW M[0xb]
b5 JEQ 0xe97d7d54 ->0xcb !->0xcc
b6 LDW M[0xb]
b7 JEQ 0x8199d8ee ->0xcb !->0xcc
b8 LDW M[0xb]
b9 JEQ 0x27bedb3e ->0xcb !->0xcc
ba LDW M[0xb]
bb JEQ 0xf6f36bda ->0xcb !->0xcc
bc LDW M[0xb]
bd JEQ 0x6ce6a6be ->0xcb !->0xcc
be LDW M[0xb]
bf JEQ 0xffbee7e6 ->0xcb !->0xcc
c0 LDW M[0xb]
c1 JEQ 0xbbf6ce2 ->0xcb !->0xcc
c2 LDW M[0xb]
c3 JEQ 0x7fe5bbc4 ->0xcb !->0xcc
c4 LDW M[0xb]
c5 JEQ 0xa22d56b4 ->0xcb !->0xcc
c6 LDW M[0xb]
c7 JEQ 0xb9fdbebe ->0xcb !->0xcc
c8 LDW M[0xb]
c9 JEQ 0xdd061f9a ->0xcb !->0xcc
# final conditions
ca RET 0x7fff0000 ALLOW # other system calls
cb RET 0x50000 ERRNO 0 # correct
cc RET 0x50001 ERRNO 1 # incorrect
```

This can then be solved using `z3`:

```python3
from z3 import *

s = Solver()

flag = [BitVec(f"f{i}", 32) for i in range(56)]

# start is known
for i, c in enumerate("zer0pts{"):
    flag[i] = BitVecVal(ord(c), 32)

# bytes must be below 0x80
for i in range(8, 56):
    s.add(flag[i] <= 0x7e)

# flag bytes also need to be > 0x20 due to scanf
# however, the flag is shorter than the allowed 55 bytes
# the end value was found experimentally
for i in range(8, 52):
    s.add(flag[i] > 0x20)

# blocks of 4 bytes
blocks = []

for i in range(0, 56, 4):
    # little-endian
    blocks.append(flag[i] | (flag[i + 1] << 8) |
                  (flag[i + 2] << 16) | (flag[i + 3] << 24))

# parse the jump conditions here
cond = """67 LDW M[0x8]
68 JEQ 0xf5ffc1f6 ->0x8e
69 JEQ 0x7344aeee ->0x7e
6a JEQ 0xfda6effe ->0x8a
6b JEQ 0x638f7ca2 ->0x76
6c JEQ 0xa2285400 ->0x7c
6d JEQ 0x8990fefe ->0x88
6e JEQ 0x9f576dd4 ->0x8c
6f JEQ 0xf6b9ebe2 ->0x7a
70 JEQ 0xf9e28bee ->0x80
71 JEQ 0x1f9b8fb4 ->0x86
72 JEQ 0xefec86de ->0x90
73 JEQ 0xdf60093a ->0x82
74 JEQ 0xb8af3fbe ->0x78
75 JEQ 0x7f01bbcc ->0x84
76 LDW M[0x9]
77 JEQ 0xf1cf5c2e ->0xaa !->0xcc
78 LDW M[0x9]
79 JEQ 0xb6af7dbe ->0x9a !->0xcc
7a LDW M[0x9]
7b JEQ 0xd6b9bde2 ->0x94 !->0xcc
7c LDW M[0x9]
7d JEQ 0x60fad508 ->0xa0 !->0xcc
7e LDW M[0x9]
7f JEQ 0x77600ede ->0xa2 !->0xcc
80 LDW M[0x9]
81 JEQ 0xf3b68ece ->0x9e !->0xcc
82 LDW M[0x9]
83 JEQ 0x4fe90926 ->0xa8 !->0xcc
84 LDW M[0x9]
85 JEQ 0x7e1933ac ->0x92 !->0xcc
86 LDW M[0x9]
87 JEQ 0x1f9b8fb4 ->0xac !->0xcc
88 LDW M[0x9]
89 JEQ 0xcb94e7da ->0xa6 !->0xcc
8a LDW M[0x9]
8b JEQ 0xb9c2adfe ->0x96 !->0xcc
8c LDW M[0x9]
8d JEQ 0x0f01b94c ->0x9c !->0xcc
8e LDW M[0x9]
8f JEQ 0xf5efe5f6 ->0xa4 !->0xcc
90 LDW M[0x9]
91 JEQ 0xa7ad8d4e ->0x98 !->0xcc
92 LDW M[0xa]
93 JEQ 0x7efd33a4 ->0xc2 !->0xcc
94 LDW M[0xa]
95 JEQ 0xd6f33dda ->0xba !->0xcc
96 LDW M[0xa]
97 JEQ 0xbbdaa5e6 ->0xbe !->0xcc
98 LDW M[0xa]
99 JEQ 0x24a7ad2e ->0xbc !->0xcc
9a LDW M[0xa]
9b JEQ 0xb7fdfcbe ->0xc6 !->0xcc
9c LDW M[0xa]
9d JEQ 0xf01b94c ->0xae !->0xcc
9e LDW M[0xa]
9f JEQ 0xb3bdaed6 ->0xb2 !->0xcc
a0 LDW M[0xa]
a1 JEQ 0x60ffd7bc ->0xc4 !->0xcc
a2 LDW M[0xa]
a3 JEQ 0x5f785fd2 ->0xb0 !->0xcc
a4 LDW M[0xa]
a5 JEQ 0x27aeff3e ->0xb8 !->0xcc
a6 LDW M[0xa]
a7 JEQ 0xc39dc1ca ->0xb6 !->0xcc
a8 LDW M[0xa]
a9 JEQ 0x4d8f1f86 ->0xc8 !->0xcc
aa LDW M[0xa]
ab JEQ 0x99ff4c6e ->0xc0 !->0xcc
ac LDW M[0xa]
ad JEQ 0xe97d7d54 ->0xb4 !->0xcc
ae LDW M[0xb]
af JEQ 0x9f576dd4 ->0xcb !->0xcc
b0 LDW M[0xb]
b1 JEQ 0x5b5cffe2 ->0xcb !->0xcc
b2 LDW M[0xb]
b3 JEQ 0xb9e9abf6 ->0xcb !->0xcc
b4 LDW M[0xb]
b5 JEQ 0xe97d7d54 ->0xcb !->0xcc
b6 LDW M[0xb]
b7 JEQ 0x8199d8ee ->0xcb !->0xcc
b8 LDW M[0xb]
b9 JEQ 0x27bedb3e ->0xcb !->0xcc
ba LDW M[0xb]
bb JEQ 0xf6f36bda ->0xcb !->0xcc
bc LDW M[0xb]
bd JEQ 0x6ce6a6be ->0xcb !->0xcc
be LDW M[0xb]
bf JEQ 0xffbee7e6 ->0xcb !->0xcc
c0 LDW M[0xb]
c1 JEQ 0xbbf6ce2 ->0xcb !->0xcc
c2 LDW M[0xb]
c3 JEQ 0x7fe5bbc4 ->0xcb !->0xcc
c4 LDW M[0xb]
c5 JEQ 0xa22d56b4 ->0xcb !->0xcc
c6 LDW M[0xb]
c7 JEQ 0xb9fdbebe ->0xcb !->0xcc
c8 LDW M[0xb]
c9 JEQ 0xdd061f9a ->0xcb !->0xcc"""

# one system call
def constrain(D):
    M = [None for _ in range(12)]
    M[0] = D[0]
    M[1] = M[0] ^ D[1]
    M[2] = M[1] ^ D[2]
    M[3] = M[2] ^ D[3]

    M[4] = M[0] + M[1] + M[2] + M[3]
    M[5] = M[0] - M[1] + M[2] - M[3]
    M[6] = M[0] + M[1] - M[2] - M[3]
    M[7] = M[0] - M[1] - M[2] + M[3]

    M[8]  = (M[4] | M[5]) ^ (M[6] & M[7])
    M[9]  = (M[5] | M[6]) ^ (M[7] & M[4])
    M[10] = (M[6] | M[7]) ^ (M[4] & M[5])
    M[11] = (M[7] | M[4]) ^ (M[5] & M[6])

    # list all jumps
    jlist = []
    rmap = {}
    curri = 0
    for l in cond.split('\n'):
        addr, *inst = l.split()
        addr = int(addr, 16)
        if inst[0] == "LDW":
            curri = int(inst[1][-2], 16)
            rmap[addr] = addr + 1
        else:
            jlist.append((addr, curri, inst))

    # construct conditions by iterating over the jumps
    # from bottom to top
    jmap = {0xcc: False, 0xcb: True, 0xca: False}
    for j in jlist[::-1]:
        v = int(j[2][1][2:], 16)
        ci = j[1]
        addr = j[0]
        tc = int(j[2][2][4:], 16)
        if tc in rmap:
            tc = rmap[tc]
        tc = And(M[j[1]] == v, jmap[tc])
        if len(j[2]) == 4:
            fc = int(j[2][3][5:], 16)
        else:
            fc = addr + 1
        if fc in rmap:
            fc = rmap[fc]
        fc = And(M[j[1]] != v, jmap[fc])
        jmap[addr] = Or(tc, fc)
    # starting address
    co = simplify(jmap[0x68])
    s.add(co)

# 14 iterations of system calls
for i in range(1, 15):
    constrain([blocks[i - 1],
               blocks[i % 0xe],
               blocks[(i + 1) % 0xe],
               blocks[(i + 2) % 0xe]])

# construct flag
s.check()
m = s.model()
o = ""
for v in flag:
    nv = m[v]
    if nv is None: # literal value
        nv = v
    v = nv.as_long()
    if v == 0: # zero byte, end
        break
    o += chr(v)
print(o) # zer0pts{B3rk3l3y_P4ck3t_F1lt3r:Y3t_4n0th3r_4ss3mbly}
```
