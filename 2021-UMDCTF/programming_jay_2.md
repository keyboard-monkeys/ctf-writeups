This problem is a continuation of "Jay 1".

The given server responds with

```
███████ ████████ ██    ██  ██████ ██   ██ 
██         ██    ██    ██ ██      ██  ██  
███████    ██    ██    ██ ██      █████   
     ██    ██    ██    ██ ██      ██  ██  
███████    ██     ██████   ██████ ██   ██ 

Welp that didn't work. As soon as they took me out of this place, they dragged me
to another room and told me to solve another stupid puzzle. You think you could help
me with this one as well? This was all they gave me:

"You are given a 2-dimensional array of values for an image. Find the
brightest subregion of the image (the bigger the number, the brighter). A subregion (rectangular) can range from one pixel to the whole 
image. You can assume the image has the same width and height."

They also gave me an example for clarification:
|-----|-----|-----|-----|-----|
|     |     |     |     |     |
|  6  | -5  | -7  |  4  | -4  |
|     |     |     |     |     |
|-----|-----|-----|-----|-----|
|     |     |     |     |     |
| -9  |  3  | -6  |  5  |  2  |
|     |     |     |     |     |
|-----|-----|-----|-----|-----|
|     |     |     |     |     |
| -10 |  4  |  7  | -6  |  3  |
|     |     |     |     |     |
|-----|-----|-----|-----|-----|
|     |     |     |     |     |
| -8  |  9  | -3  |  3  | -7  |
|     |     |     |     |     |
|-----|-----|-----|-----|-----|

Format: "sum, row_start, row_end, col_start, col_end" Example: 17, 2, 3, 1, 2

sum = the maximum subregion sum
row_start = row index of top left
row_end = row index of bottom right
col_start = col index of top left
col_end = col index of bottom right

Press Enter to get the 2D array
```

Pressing enter makes the server send a `300` by `300` 2D array of numbers, out of which the rectangle with maximum sum must be found. The problem text hints at an `O(n^3)` solution, which can be achieved by iterating over all the possible start and end rows for the rectangle. Then all that's left to do is to find the columnwise sum (from column prefix sums) to generate a 1D array and to run Kadane's algorithm on that, then find the maximum over all such runs.

```python3
from pwn import *
import time
r = remote("chals3.umdctf.io", 6002)

r.recvuntil("Press")
r.sendline()
r.recvuntil("[")
vals = eval("[" + r.recvuntil("]]").decode())
print(len(vals), len(vals[0]))

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
            best_end = current_end

    return best_sum, best_start, best_end

# copied idea from https://stackoverflow.com/a/21940302
def max_2d(numbers):
    # columnwise prefix sum
    for r in range(1, len(numbers)):
        for c in range(len(numbers[0])):
            numbers[r][c] += numbers[r - 1][c]
    # empty prefix
    numbers.insert(0, [0 for _ in range(len(numbers[0]))])

    best_rs = 0
    best_re = 0
    best_cs = 0
    best_ce = 0
    best = 0
    for rs in range(len(numbers) - 1):
        for re in range(rs + 1, len(numbers)):
            l = [numbers[re][c] - numbers[rs][c] for c in range(len(numbers[0]))]
            s, cs, ce = max_subarray(l)
            if s > best:
                best_rs = rs
                best_re = re - 1
                best_cs = cs
                best_ce = ce
                best = s
    return best, best_rs, best_re, best_cs, best_ce


ansv = max_2d(vals)
ans = ", ".join(map(str, ansv))
print(ans)
r.sendline(ans)
r.interactive()
```

After solving the problem, the server responds with text containing the flag `UMDCTF-{Pr0f3ss0r_Gr3n4nd3r}`.
