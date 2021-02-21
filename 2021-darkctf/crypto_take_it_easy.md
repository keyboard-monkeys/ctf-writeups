The first part of this challenge is a broken RSA implementation. We are given 3 numbers, `p`, `ct` and `e`, which correspond to the RSA modulus, ciphertext, and public exponent. However, the modulus is prime, so we can easily compute phi(p) = p-1, and then compute the private exponent `d` using `d = e^-1 (mod phi(p))`. Then we can simply decrypt the ciphertext to get the zip password.

```py
d = pow(e, -1, p-1)
pt = pow(ct, d, p)
password = pt.to_bytes(10, 'big')
print(password)
# b'Ju5t_@_K3Y'
```

Inside the zip file, we find a simple xor-based cipher. Each block of the ciphertext is 2 blocks of the flag xor'ed together. Since xor is its own inverse, if we know one of the blocks that went into xor, we can get the other block by simply applying xor again. We know the first 2 blocks because we know the flag format.

```py
# bytewise xor
xor = lambda a,b: bytes(x^y for x,y in zip(a,b))

blocks = [b'\nQ&4', b"\x17'\x0e\x0f", b'1X5\r', b'072E', b'\x18\x00\x15/']
output = [b"dark", b"CON{"]
for i,ct in enumerate(blocks):
    output.append(xor(ct, output[i]))
print(b"".join(output))
# b'darkCON{n0T_Th@t_haRd_r1Ght}'
```
