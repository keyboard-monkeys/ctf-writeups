This challenge involves a Diffie-Hellman key exchange between 2 parties, Alice and Bob. The intended solution involves establishing different keys with Alice and Bob, however the values are so small (around 32 bits) that we can simply solve the discrete logarithm problem (the hardness of which Diffie-Hellman's security depends on) with brute force.

```py
# pip install pwntools sympy pycryptodome
import pwn
from sympy.ntheory import discrete_log
from Crypto.Cipher import AES

# from MITM.c
primes = [1697841911,1438810907,666397859,941857673]
g = 13061880230110805485346525688018595113271880103717720219673350299083396780730251766148414377512386061643807530751287373200960399392170617293251618992497053

conn_a = pwn.remote("13.233.166.242", 49154)
conn_b = pwn.remote("13.233.166.242", 49155)

# read the 4 public values from alice
a_pub = []
conn_a.recvline()
for i in range(4):
    a_pub.append(int(conn_a.recvline()))
conn_a.recvline()

# and from bob
b_pub = []
conn_b.recvline()
for i in range(4):
    b_pub.append(int(conn_b.recvline()))
conn_b.recvline()

# send alice's public values to bob, and vice versa
conn_a.sendline('\n'.join(map(str, b_pub)))
conn_b.sendline('\n'.join(map(str, a_pub)))

# read alice's ciphertext
ct_a = conn_a.recvline()
ct_a = bytes.fromhex(ct_a.split(b" : ")[1].replace(b"0x", b"").decode())

# and send it to bob
conn_b.recvline()
conn_b.send(ct_a)

# and then read bob's ciphertext
ct_b = conn_b.recv()
ct_b = bytes.fromhex(ct_b.split(b" : ")[1].replace(b"0x", b"").decode())

# Now, compute the private values of both alice and bob by solving the
# discrete logarithm problem on their public values (since g^priv = pub).
# These aren't the actual values, but rather the values mod each of the 4
# primes used, but that is good enough since we always operate mod those primes
priv_a_res = [int(discrete_log(primes[i], a_pub[i], g)) for i in range(4)]
priv_b_res = [int(discrete_log(primes[i], b_pub[i], g)) for i in range(4)]

# Now that we know the private values, compute the AES keys
key_a = [pow(b_pub[i], priv_a_res[i], primes[i]) for i in range(4)]
key_b = [pow(a_pub[i], priv_b_res[i], primes[i]) for i in range(4)]

# convert from integers to byte sequences
key_a = b''.join(x.to_bytes(4, 'little') for x in key_a)
key_b = b''.join(x.to_bytes(4, 'little') for x in key_b)

# and decrypt the ciphertexts
pt_a = AES.new(key_a, AES.MODE_CBC, iv=b'\x00'*16).decrypt(ct_a)
pt_b = AES.new(key_b, AES.MODE_CBC, iv=b'\x00'*16).decrypt(ct_b)

print(pt_a + pt_b)
# b'darkCON{d1ff13_h3llm4n_1s_vuln3r4bl3_t0_m4n_1n_th3_m1ddl3_1789}\x00'
```
