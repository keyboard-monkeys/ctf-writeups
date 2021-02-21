This was more of a guessing challenge than a crypto challenge. We are given 10 seemingly random integers and asked to predict the next one. The numbers are all less than 2^31, so it is most likely a 31-bit PRNG. The simplest and most well-known class of PRNGs is Linear Congruential Generators, or LCGs for short. LCGs usually have 3 parameters: the modulus, the multiplier, and the increment, or `m`, `a` and `c` for short. All LCGs produce their next output with the following formula:

```
next = (a * prev + c) % m
```

Now, if we know `m`, we can compute the parameters of the LCG by observing 3 consecutive outputs from it, like this:

```
y = a*x + c
z = a*y + c
z-y = a*y+c - a*x - c = a*(y-x)
a = (z-y) / (y-x)
c = z - a*y
```

Note that all of these are done modulo `m`. We can divide by multiplying with the modular multiplicative inverse.

This will give an answer for every set of 3 outputs. We can verify it by also computing the parameters for a different set of 3 outputs and checking that we get the same multiplier and increment.

Now, all that remains is to find `m`. Since all input numbers are between 0 and around 2 billion, it's reasonable to assume that `m` is around `2^31`. However, if we try exactly `2^31`, we find that we get a different multiplier for each set of 3 outputs, so this isn't the correct modulus. LCGs often have a prime modulus though (it helps with increasing the period), so we can also try `2^31-1`. With that, we find `a = 16807` and `c = 0`. These are consistent for every set of 3 outputs, so they are the correct parameters for the LCG.

```py
x = 1875578086
y = 2075920736
z = 1980480790

m = 2**31-1

a = (z-y) * pow(y-x, -1, m) % m
c = (z - a*y) % m
print("next output after z: ", (a*z+c) % m)
# in this example, the next output is 2091592677
```

Entering this gives us the flag.

Also, these parameters happen to match the LCG used in C++'s `minstd_rand0` algorithm, so you might have solved it by trying random PRNGs until one matched the sequence in the challenge.
