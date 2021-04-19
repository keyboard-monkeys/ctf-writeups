We were given a memory dump, a dwarf file and a system map. https://drive.google.com/drive/folders/1c6vdBabGu33edSLXQZY5s8JAeM8au8Uo?usp=sharing

The dwarf and system map can be zipped and put in the folder `volatility/plugins/overlays/linux/` to create a profile. The name of the created profile can be checked with `python2 vol.py --info | grep Profile` and in my case, it was Linuxphillipx64.

We can view the bash hisory using the command `python2 vol.py --profile=Linuxphillipx64 -f ../philip-1.raw linux_bash`, which outputs the following:
```
Pid      Name                 Command Time                   Command
-------- -------------------- ------------------------------ -------
    1534 bash                 2021-04-03 03:18:46 UTC+0000   clear
    1534 bash                 2021-04-03 03:19:19 UTC+0000   scp -i key -P 5001 ./super-secret-flag lubuntu@chals2.umdctf.io:~/
    1534 bash                 2021-04-14 22:11:37 UTC+0000   ssh -i key lubuntu@chals2.umdctf.io -p 5001
```

We can see, that to get the flag, we can ssh, but we need the key file. RSA private keys can be found with memdump.py. https://github.com/Crapworks/pentest/blob/master/memdump.py

The command `python2 memdump.py philip-1.raw` outputs the following:
```
[*] Modul: privatekey [ 14 items ]
 [+] offset: 0x12688000 - length: 1678
 [+] offset: 0x3f4f4740 - length: 1211
 [+] offset: 0x420ef998 - length: 88807105
 [+] offset: 0x49c9f978 - length: 55832087
 [+] offset: 0x49c9f998 - length: 55832055
 [+] offset: 0x475a10d8 - length: 96720567
 [+] offset: 0x09cc4197 - length: 144458999
 [+] offset: 0x3f4f4d20 - length: 46115477
 [+] offset: 0x3602fcf8 - length: 155995907
 [+] offset: 0x09cc5417 - length: 144454263
 [+] offset: 0x3602fb18 - length: 155996387
 [+] offset: 0x475a12b8 - length: 96720087
 [+] offset: 0x09cc41bc - length: 144458962
 [+] offset: 0x420ef958 - length: 88807169
```

These are all possible keys. We can check them with `hexdump -C -s offset -n length philip-1.raw` and the first one shows promise. We can extract it with `dd bs=1 skip=308838400 count=1678 if=philip-1.raw of=key`. (0x12688000 = 308838400)

On most linux machines, we have to change the key file permissions for ssh to allow to use it. That can be accomplished with the command `chmod 600 key`.

Now we can ssh with the command `ssh -i key lubuntu@chals2.umdctf.io -p 5001` and get the file `super-secret-flag` from there. The file contains `VU1EQ1RGLXtHNGxsNGdoM3JfNF8xaWYzfQ==`, which can be decoded from base64 to get the flag `UMDCTF-{G4ll4gh3r_4_1if3}`.
