We were given a picture, which was a stereogram.

![original imag](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-UMDCTF/data/magic.png)

The flag can be extracted with the tool stegsolve.jar. https://github.com/eugenekolo/sec-tools/blob/master/stego/stegsolve/stegsolve/stegsolve.jar

Using the tool and selecting Analyse/Stereogram Solver with the offset 72, we get the flag `UMDCTF-{th15_15_b14ck_m4g1k}`.

![flag](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-UMDCTF/data/magic_solve.png)
