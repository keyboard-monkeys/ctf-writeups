In this challenge we are given one RSA private key and 5 public keys. However, the modulus for all of the private keys is the same. This means that if we manage to factor the modulus, we can compute the private exponents of all keys. This can be done with an algorithm such as [this one](https://crypto.stackexchange.com/a/62487), however it is also already implemented in PyCryptodome.

```py
from Crypto.PublicKey import RSA

# from alice_secret.txt
n = 134...707
e = 65537
d = 195...593

key = RSA.construct((n,e,d))
# PyCryptodome computes p and q for us
phi = (key.p-1)*(key.q-1)

# from all_publickeys.txt
exponents = [65537, 68719476737, 4294967297, 1048577, 16777217]
encrypted_msgs = []

with open("encrypted_chats.txt") as f:
    for line in f:
        if ': ' in line:
            encrypted_msgs.append(eval(line.split(": ")[1]))

for i in range(5):
    d = pow(exponents[i], -1, phi)
    ct = int.from_bytes(encrypted_msgs[i], 'big')
    msg = pow(ct, d, n).to_bytes(128, 'big').lstrip(b'\x00')
    print(msg)

# b'Hi everyone! Are you enjoing darkCON?\n'
# b'Hell yeah! Charlie, do you have the secret thing?\n'
# b'Got it! darkCON{4m_I_n0t_supp0sed_t0_g1v3_d???}\n'
# b'[Sush]...[Sussshhh]... Eve Eve...Eve is there!\n'
# b"Nah...She doesn't even have our Private Keys"
```

