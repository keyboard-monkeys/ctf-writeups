The provided executable offers to both compress and decompress files using some compression technique, but doesn't actually have decompressing functionality. Analyzing the compresssion functionality (ghidra), the first compression function is called with the input filename and a password as parameters. That function opens the input file, generates the output filename (appends `.shitty` to the original filename) and then passes them along with the password to the second compression function. This function performs the actual compression (variables renamed):

```C
  char *data;
  char *dataPtr;
  FILE *__s;
  int blockCount;
  uint lenMinus1;
  ulong __size;
  ulong uVar1;
  char origCh;
  char currCh;
  
  puts("[+] Compression goes brrr...");
  data = (char *)malloc((ulong)(in_len * 3));
  if (in_len == 0) {
    __size = 0;
  }
  else {
    lenMinus1 = in_len - 1;
    dataPtr = in_data + 1;
    blockCount = 1;
    __size = 0;
    uVar1 = 0;
    origCh = *in_data;
    do {
      while( true ) {
        in_len = (uint)uVar1;
        currCh = *dataPtr;
        if (currCh != origCh) break;
        dataPtr = dataPtr + 1;
        blockCount = blockCount + 1;
        if (in_data + (ulong)lenMinus1 + 2 == dataPtr) goto LAB_001014ac;
      }
      data[__size] = origCh;
      dataPtr = dataPtr + 1;
      data[in_len + 1] = (char)((uint)blockCount >> 8);
      data[in_len + 2] = (char)blockCount;
      in_len = in_len + 3;
      __size = (ulong)in_len;
      blockCount = 1;
      uVar1 = __size;
      origCh = currCh;
    } while (in_data + (ulong)lenMinus1 + 2 != dataPtr);
  }
```

This essentially reads consecutive blocks of the same character, then encodes them in three bytes: the character and a two-byte value for how many repetitions of it there were.

However, following the compression there is encryption, which calls another function, passing the compressed data buffer, the length of the data and the password as parameters:

```C
  size_t passLen;
  ulong i;
  
  passLen = strlen(password);
  if ((int)(size & 0xffffffff) != 0) {
    i = 0;
    do {
      data[i] = (byte)(((uint)((byte)data[i] >> 4) ^
                       (int)(password[(i & 0xffffffff) % (passLen & 0xffffffff)] >> 4)) << 4) |
                (password[(i & 0xffffffff) % (passLen & 0xffffffff)] ^ data[i]) & 0xfU;
      i = i + 1;
    } while ((size & 0xffffffff) != i);
  }
```

which is just XOR with the password cyclically. There is a password in `main` that could be the one used. Reversing the encryption:

```python 3
with open("Installer.shitty", "rb") as f:
    data = f.read()

password = "ShimtyPasmword"
passLen = len(password)
with open("decrypted", "wb") as f:
    for i, c in enumerate(data):
        d = ord(password[i % passLen]) ^ c
        f.write(bytes([d]))
```

and the compression:

```python 3
with open("decrypted", "rb") as inFile:
    with open("decompressed", "wb") as outFile:
        try:
            while True:
                ch, hi, lo = inFile.read(3)
                outFile.write(bytes([ch for _ in range((hi << 8) | lo)]))
        except:
            pass
```
However, the resulting file seems like garbage, not an executable file. There are a lot of repeated characters at the beginning instead of an `ELF` header, so the password is likely wrong. Reading the description again, V does say an odd string:

> V: CyberDarkIsACoolGameAndIWannaPlayIt

which turns out to be the correct password. Unencrypting and decompressing the file again, the output is a working ELF file (missing some zero bytes at the end, which makes ghidra give up), which prints the flag `darkCON{c0MpR3553d_g4M3_1N5T4Ll3R_w1th_5h1mTTY_p455W0RD}` on screen.
