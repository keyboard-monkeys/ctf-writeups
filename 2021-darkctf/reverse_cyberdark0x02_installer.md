Looking for where the entered key is read in, there is a function (offset `0x00101ab0`) which is called from a function called in a loop from  `main` and it reads input with `wgetnstr`. Fixing its stack frame to account for the different arrays:

```
undefined8        RAX:8          keyLen
undefined1        SIL:1          badFlag
undefined8        RAX:8          retVal
undefined8        Stack[-0x10]:8 local_10
undefined1[264]   Stack[-0x118   weirdArr
char[14]          Stack[-0x138   input_key  
undefined1[48]    Stack[-0x168   key_no_dashes
```

This function first validates the key format (output from ghidra):

```C
  wgetnstr(stdscr,input_key,0xffffffff);
  keyLen = strlen(input_key);
  badFlag = false;
  sVar1 = 0;
  while (keyLen != sVar1) {
    if ((((int)sVar1 != 4) && ((int)sVar1 != 9)) &&
       (0x19 < (byte)(*(char *)(sVar1 + (long)input_key) + 0xbfU))) {
      badFlag = true;
    }
    sVar1 = sVar1 + 1;
  }
  if (((keyLen != 0xe) || (input_key[4] != '-')) || ((input_key[9] != '-' || (badFlag)))) {
    retVal = 0xffffffff;
  }
```

then copies all the non-dash characters to `key_no_dashes`:

```C
    lVar2 = 0;
    iVar6 = 0;
    do {
      if (((int)lVar2 != 4) && ((int)lVar2 != 9)) {
        lVar5 = (long)iVar6;
        iVar6 = iVar6 + 1;
        key_no_dashes[lVar5] = (int)*(char *)(lVar2 + (long)input_key);
      }
      lVar2 = lVar2 + 1;
    } while (lVar2 != 0xe);
```

calls a function with `weirdArr` as a parameter, then maps all the characters in `key_no_dashes` using `weirdArr` (taking the element at the index corresponding to the character's value):

```C
    puVar3 = key_no_dashes;
    do {
      puVar4 = puVar3 + 1;
      *puVar3 = (uint)weirdArr[(int)*puVar3];
      puVar3 = puVar4;
    } while ((uint *)input_key != puVar4);
```

and finally calls yet another function to determine whether the modified key is valid. This function allocates a large array on the stack and copies data from somewhere in memory into it:

```C
memcpy(aiStack8216,&DAT_00103560,0x2008);
```

and then does the actual validation:

```C
  piVar2_rdx = modifiedKey + 1;
  iVar1_rax = aiStack8216[*modifiedKey];
  do {
    if (*piVar2_rdx != aiStack8216[iVar1_rax]) {
      retVal = 0;
      goto LAB_00101a80;
    }
    piVar2_rdx = piVar2_rdx + 1;
    iVar1_rax = aiStack8216[iVar1_rax + 1];
  } while (piVar2_rdx != modifiedKey + 0xc);
  retVal = 1;
```

This first sets `iVar1_rax` from the value obtained from the first character. Then, in a loop, it checks the array at that position against the value obtained from the current character (`*piVar2_rdx`), then sets `iVar1_rax` to the value in the array at `iVar1_rax + 1`.

This means that for every first character (the value obtained from it), there is at most one key that is valid. Thus, the keys can be generated by trying all the possible first characters and seeing whether a valid key sequence results.

The values for the large array in the last function can be dumped from memory in `gdb` after the `memcopy` (`dump memory`), and the same could be done for `weirdArr` in the first function. However, code can be run to generate that array:

```C
#include <stdio.h>

typedef unsigned char byte;
typedef unsigned long ulong;
typedef unsigned int uint;

char arr[270];

int main() {
  byte bVar1;
  uint uVar2;
  ulong uVar3;
  byte bVar4;

  uVar2 = 1;
  uVar3 = 1;
  do {
    bVar4 = (byte)uVar3;
    bVar1 = bVar4 * '\x02' ^ bVar4;
    if ((char)bVar4 < '\0') {
      bVar1 = bVar1 ^ 0x1b;
    }
    uVar3 = (ulong)bVar1;
    uVar2 = uVar2 * 2 ^ uVar2;
    uVar2 = uVar2 ^ uVar2 * 4;
    uVar2 = uVar2 << 4 ^ uVar2;
    if ((char)uVar2 < '\0') {
      uVar2 = uVar2 ^ 9;
    }
    bVar4 = (byte)uVar2;
    arr[uVar3] = (bVar4 << 1 | (char)bVar4 < '\0') ^ (bVar4 << 4 | bVar4 >> 4) ^ bVar4 ^
      (bVar4 << 2 | bVar4 >> 6) ^ (bVar4 << 3 | bVar4 >> 5) ^ 99;
  } while (bVar1 != 1);
  *arr = 99;

  for(int i = 0; i < 270; ++ i) {
    printf("%d ", (uint)(byte)arr[i]);
  }
  printf("\n");
}
```

(Only the first 256 bytes can matter.)

A full keygen (large array dumped in `middump`):

```python 3
bigArr = []
with open("middump", "rb") as f:
    for i in range(0x2008 // 4):
        v = int.from_bytes(f.read(4), "little")
        bigArr.append(v)

# generated array
weirdArr = [99, 124, 119, 123, 242, 107, 111, 197, 48, 1, 103, 43, 254, 215, 171, 118, 202, 130, 201, 125, 250, 89, 71, 240, 173, 212, 162, 175, 156, 164, 114, 192, 183, 253, 147, 38, 54, 63, 247, 204, 52, 165, 229, 241, 113, 216, 49, 21, 4, 199, 35, 195, 24, 150, 5, 154, 7, 18, 128, 226, 235, 39, 178, 117, 9, 131, 44, 26, 27, 110, 90, 160, 82, 59, 214, 179, 41, 227, 47, 132, 83, 209, 0, 237, 32, 252, 177, 91, 106, 203, 190, 57, 74, 76, 88, 207, 208, 239, 170, 251, 67, 77, 51, 133, 69, 249, 2, 127, 80, 60, 159, 168, 81, 163, 64, 143, 146, 157, 56, 245, 188, 182, 218, 33, 16, 255, 243, 210, 205, 12, 19, 236, 95, 151, 68, 23, 196, 167, 126, 61, 100, 93, 25, 115, 96, 129, 79, 220, 34, 42, 144, 136, 70, 238, 184, 20, 222, 94, 11, 219, 224, 50, 58, 10, 73, 6, 36, 92, 194, 211, 172, 98, 145, 149, 228, 121, 231, 200, 55, 109, 141, 213, 78, 169, 108, 86, 244, 234, 101, 122, 174, 8, 186, 120, 37, 46, 28, 166, 180, 198, 232, 221, 116, 31, 75, 189, 139, 138, 112, 62, 181, 102, 72, 3, 246, 14, 97, 53, 87, 185, 134, 193, 29, 158, 225, 248, 152, 17, 105, 217, 142, 148, 155, 30, 135, 233, 206, 85, 40, 223, 140, 161, 137, 13, 191, 230, 66, 104, 65, 153, 45, 15, 176, 84, 187, 22, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

allowedSlice = weirdArr[ord('A'):ord('Z')+1]

revMap = {i: chr(ord('A')+c) for c, i in enumerate(allowedSlice)}

for initc in allowedSlice:
    try:
        key = ""
        key += revMap[initc]
        curri = bigArr[initc]
        for _ in range(11):
            key += revMap[bigArr[curri]]
            curri = bigArr[curri + 1]
        key = list(key)
        key.insert(4,'-')
        key.insert(9,'-')
        print("".join(key))
    except:
        pass
```

which generates precisely 10 keys:

```
ADGR-THFS-SZPF
BNQO-PSBL-SDHD
CROG-IURB-TNDE
IOXT-KDQL-VZFE
JIRX-YPMF-IZLO
QWMA-LSRT-PZKX
SXFH-MICO-PEWZ
WECN-UQWO-PQWX
XBUZ-CHUC-IKLY
YJFS-ZLMU-RVJT
```

Giving these to the server returns the flag `darkCON{DRM_FR33_g4m35_4r3_EZ-T0_cR4ck}`.