*It doesn't really look like gold to me...*

![image](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-hsctf/data/pallets-of-gold.png)

We are given an image and using [stegsolve](https://github.com/eugenekolo/sec-tools/blob/master/stego/stegsolve/stegsolve/stegsolve.jar), we can see that the flag is hidden behind the noise in some colour planes. It is most clear using modes Red plane 0 or Gray bits.

![image](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-hsctf/data/pallets-of-gold-1.png)

![image](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-hsctf/data/pallets-of-gold-2.png)

From there we can read the flag `flag{plte_chunks_remind_me_of_gifs`.