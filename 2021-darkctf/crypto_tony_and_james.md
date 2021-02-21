This challenge has a custom encrypt function based on RNG. Each ciphertext block is the xor of a random 512-bit integer, one byte of plaintext, and a `raw` value which is derived from the seed. In the info PDF, we are given the first random value.

Looking at the `get_seed` code, `raw[0]` is almost exactly the value of `rand`, except it's shifted to the right by one bit. This means that if we know `raw[0]`, we only have 2 possible values for `rand`, and knowing `rand` allows us to compute both `raw` and `seed`.

Since we know `r0`, we can obtain `raw[0] ^ m[0]` by simply computing `r0 ^ ct_blocks[0]`. There are only 256 possible values for `m[0]`, so we can just try them all. All in all, this leaves 512 candidates for the `rand` in `get_seed`, which we can brute-force. Once we know the correct `raw` and `seed`, we can pretty much just run the encryption again to get the original message.

```py
import random
ct_blocks = []
with open("encrypted.txt") as f:
    for line in f:
        ct_blocks.append(int(line.split()[1], 16))

# from info.pdf
r0 = 1251602129774106047963344349716052246200810608622833524786816688818258541877890956410282953590226589114551287285264273581561051261152783001366229253687592

# from src.py
def get_seed(l, rand):
    seed = 0
    raw = []
    while rand > 0:
        rand = rand >> 1
        seed += rand
        raw.append(rand)
    return raw, seed

tmp = r0 ^ ct_blocks[0]
for i in range(512):
    rand = (tmp << 1) ^ i
    raw, seed = get_seed(64, rand)
    random.seed(seed)
    if random.randint(1, 2**512) == r0:
        break

out = []
random.seed(seed)
for i in range(64):
    r = random.randint(1, 2**512)
    pt = r ^ raw[i] ^ ct_blocks[i]
    out.append(pt)
print(bytes(out))
# b'darkCON{user_W4rm4ch1ne68_pass_W4RM4CH1N3R0X_t0ny_h4cked_4g41n!}'
```

