The server provides multiple mathematical expressions, some defining intermediate variables and others defining the end `RESULT`. The problems are presented as strings consisting of leetspeakified words describing numbers and operations. The main problem is parsing this to some evaluatable form, which can be done using replacements and the `number_parser` module (on un-leetspeakified text):

```python3
from pwn import *
from number_parser import parse as nparse

r = remote("chals3.umdctf.io", 6003)

# start
r.sendline("yes")

replace = {
    '0': "o",
    '1': "il",
    '3': "e",
    '4': "a",
    '5': "s",
    '7': "tf",
    '9': "g",
    '$': "s"
}

accept = ["one", "two", "three", "four", "five", "six", "seven", "eight", "nine", "zero", "thousand", "hundred", "million", "billion", "plus", "minus", "times", "or", "and", "hundreed", "ten", "twenty", "thirty", "fourty", "forty", "fifty", "sixty", "seventy", "eighty", "ninety", "eleven", "twelve", "thirteen", "fourteen", "fifteen", "sixteen", "seventeen", "eighteen", "nineteen", "mod", "divided", "by"]

replW = {"hundreed": "hundred", "fourty": "forty"}


# make a regular word if possible
def interpret(word):
    for w in accept:
        if len(w) == len(word):
            for ca, cb in zip(word, w):
                if ca != cb and cb not in replace.get(ca, ""):
                    break
            else:
                # some words are misspelled
                return replW.get(w, w)
    return word


while True:
    r.recvuntil(":")
    r.recvuntil("\n")

    line = r.recvline().decode().lower()
    vl = {}
    while True:
        var, _, *l = line.split()
        # un-leetspeak
        l = list(map(interpret, l))
        # parse numbers and add operators
        ls = nparse(" ".join(l)).replace("plus", "+").replace("minus", "-").replace("times", "*").replace("divided by", "//").replace("mod", "%").replace("and", "&").replace("or", "|")
        # replace any variables with their values
        for k in vl:
            ls = ls.replace(k, str(vl[k]))
        print(var, ls)
        # calculate the result
        res = eval(ls)
        print(res)
        # add the result to the dictionary (for variables)
        vl[var] = res
        # final result
        if var == "result":
            break
        line = r.recvline().decode().lower()
    
    r.sendline(str(vl["result"]))
    l = r.recvline().decode()
    print(l)
    # continue if there are any more queries
    if "0 more" in l:
        break
r.interactive()

```

The script *occasionally* fails for some odd inputs, but works most of the time. After answering all the required queries, the server returns the flag `UMDCTF-{h4h4_w311_4t_134$7_th4ts_f1n4lly_0v3r_r1gh7?}`.