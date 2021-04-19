This challenge was purely guesswork.
We were only given the website `http://chals5.umdctf.io:4002` and no other information. 

The flag is hidden in `http://chals5.umdctf.io:4002/<number>`, where \<number\> is a number from 0 to 20. Each of the pages has a character, which corresponds to the character in the number's place in the flag.

I wrote a python script to extract the flag.

```python
import requests

ans = ''

for i in range(21):
    r=requests.get("http://chals5.umdctf.io:4002/"+str(i))
    ans += r.text

print(ans)

```

The flag is UMDCTF-{r0b0t5_43v3r}.