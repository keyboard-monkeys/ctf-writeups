*Orca watching is an awesome pastime of mine!*

![image](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-hsctf/data/lsblue.png)

We are given a picture and the challenge title, which is a huge hint, that we should look for lsb on the blue plane. Using [stegsolve](https://github.com/eugenekolo/sec-tools/blob/master/stego/stegsolve/stegsolve/stegsolve.jar) we can confirm, that there is something on blue plane 0.

![image](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-hsctf/data/lsblue-1.png)

If we extract it, we get tha flag `flag{0rc45_4r3nt_6lu3_s1lly_4895131}`.