In this challenge we were given a picture.

![picture](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-UMDCTF/data/testudos_message.png)

in the picture, there are three columns of shapes, each with seven coloured stripes. Red stipes correspond to 1, white to 0. While addin the 8-th bite 0 to the beginning and reading the bites from top left down each column to bottom right. The binary we get is `01101000 00100001 01100100 01100100 00110011 01101110 01011111 00100100 01101000 00110011 01101100 01101100 01110011`, which translates to ascii `h!dd3n_$h3lls`, which is the flag.
