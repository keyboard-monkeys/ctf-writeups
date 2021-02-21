This challenge lives up to its description - 200 bytes of data are read in, following which each byte is run through a *separate* function. However, the functions are very similar and are luckily arranged in the correct order in the binary. Running the binary through ghidra and dumping the result as C, the result is

```C
...

uint f122323(char param_1)

{
  uint uVar1;
  
  uVar1 = (int)(param_1 >> 4) + ((int)param_1 & 0xfU) * 0x10;
  return uVar1 & 0xffffff00 | (uint)(uVar1 == 0x46);
}

uint f7093176(char param_1)

{
  uint uVar1;
  
  uVar1 = (int)(param_1 >> 4) + ((int)param_1 & 0xfU) * 0x10;
  return uVar1 & 0xffffff00 | (uint)(uVar1 == 0x16);
}

...
```

The returned value needs to be nonzero, so the second condition comparing `uVar1` should be true. The values being compared to are the only values that differ across the functions and can be extracted with a regex:

```
uVar1 == (?:0x)?([a-f0-9]{1,2})
```

(Ghidra dumps smaller values in decimal.) Then all that's left to do is to get back the parameter values, which `z3` can easily solve (the previously extracted hexadecimal values saved in `all`):

```python 3
from z3 import *

def do_solve(x):
    sol = z3.Solver()
    v = z3.BitVec("v", 32)
    sol.add(v > 0)
    sol.add(v < 256)
    v2 = (v >> 4) + (v & 0xf) * 0x10
    sol.add(v2 == x)
    sol.check()
    s = sol.model()
    return s[v].as_long()

j = ""
with open("all") as f:
    for l in f:
        if l.strip() != "":
            v = int(l.strip(), 16)
            s = do_solve(v)
            print(v, s, chr(s))
            j += chr(s)

print(j)

# darkCON{4r3_y0u_r34lly_th1nk1n9_th4t_y0u_c4n_try_th15_m4nu4lly???_Ok_I_th1nk_y0u_b3tt3r_us3_s0m3_aut0m4t3d_t00ls_l1k3_4n9r_0r_Z3_t0_m4k3_y0ur_l1f3_much_e4s13r.C0ngr4ts_f0r_s0lv1in9_th3_e4sy_ch4ll3ng3}
```
