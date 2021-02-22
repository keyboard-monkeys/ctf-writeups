# OSINT/Lola challenges

Both were easily solved once the hint about the username came out. Plugging it into `https://instantusername.com/` and then checking the accounts made it trivial.



## Lola 1

After finding the reddit account from `about.me/l.beck` by putting the hex value into `https://gchq.github.io/CyberChef` we got r_7bonnie, which by repeating the `instantusername.com` search led to a Reddit account. The account had a gif which contained a QR code, which was the flag.

## Lola 2

The account from 500px led to a `https://pastebin.com/E5ipvv8K` link full of coordinates. Mapping these out using `https://maps.co/` led to the flag.



