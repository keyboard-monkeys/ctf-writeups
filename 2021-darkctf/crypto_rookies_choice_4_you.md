This is a classic stream cipher nonce reuse attack. RC4 is a stream cipher: given a key, it generates secure pseudorandom bytes (called the keystream) which you can xor with your message to encrypt it. Then to decrypt you simply xor with the keystream again. However, if you have a known plaintext-ciphertext pair, you can xor those together to recover the keystream. Then you can use the keystream to decrypt any other message encrypted with that keystream.

```py
from pwn import xor  # pip install pwntools
# from out.txt
pt1 = b"0000000000000000000000000000000000000000000000000000000"
ct1 = bytes.fromhex("6c0fd74818a4542dd8d35d5126fbb044218b4ceaebcf4a8e6895e431f36890a17f8c7ecef5d6554e706727eeafa062b58119068d8e15b3")
enc_flag = bytes.fromhex("385e95136bdb2a66baa0593e27b8df03228f1785ea9925c768d08b74b06bffe27bd17da1aed51c21342026bdacb173f8")

keystream = xor(pt1, ct1)
flag = xor(enc_flag, keystream)[:len(enc_flag)]
print(flag)
# b'darkCON{RC4_1s_w34k_1f_y0u_us3_s4m3_k3y_tw1c3!!}' 
```
