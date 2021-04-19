The given server responds with

```
██   ██ ███████ ██      ██████  
██   ██ ██      ██      ██   ██  
███████ █████   ██      ██████   
██   ██ ██      ██      ██       
██   ██ ███████ ███████ ██       

I've been stuck in this room for some time now. They locked me in here and told
me the only way I can leave is if I solve their problem within five seconds. 
I've tried so much, but my algorithm keeps going over the time. Can you help me?

What I have to do is find the maximum contiguous sub-array within this array of 
numbers. They keep telling me my algorithm isn't efficient enough! If you can 
send me the indices of this array, I might live to see another day.

Format: "sum, i, j" Example: 103, 345, 455

sum = the maximum contiguous sum
i = start index of sub-array
j = end index of sub-array

Press Enter to get the arr
```

Pressing enter makes the server send a list of `20000` numbers, out of which the subarray with maximum sum must be found within `5` seconds. This can be solved easily using Kadane's algorithm:

```python3
from pwn import *
import time
r = remote("chals3.umdctf.io", 6001)

r.sendline()
r.recvuntil("[")
vals = list(map(int, r.recvuntil("]")[:-1].decode().split(", ")))
print(len(vals))

# yank from wikipedia
def max_subarray(numbers):
    """Find a contiguous subarray with the largest sum."""
    best_sum = 0  # or: float('-inf')
    best_start = best_end = 0  # or: None
    current_sum = 0
    for current_end, x in enumerate(numbers):
        if current_sum <= 0:
            # Start a new sequence at the current element
            current_start = current_end
            current_sum = x
        else:
            # Extend the existing sequence with the current element
            current_sum += x

        if current_sum > best_sum:
            best_sum = current_sum
            best_start = current_start
            best_end = current_end + 1  # the +1 is to make 'best_end' exclusive

    return best_sum, best_start, best_end

ansv = max_subarray(vals)
ans = f"{ansv[0]}, {ansv[1]}, {ansv[2]-1}"
print(ans)
print(sum(vals[ansv[1]:ansv[2]]))
r.sendline(ans)
r.interactive()
```

After solving the problem, the server responds with text containing the flag `UMDCTF-{K4d4n35_41g0r1thm}`.
