We are given a sound file, which is Coldplay's song called Flags.

In the end of the file, there are zip archives, which can be extracted with the command `binwalk -e flags.wav`. One of them is unencrypted and contains the file `hint.txt`. In the file was the following text:
```
Assume all characters are lowercase
password: (1:49)_(1:01)_(0:16)_(0:56)_(0:17)_(1:33)_a_(1:34)?
```

I found the words in the song at the specified locations in the hint. If there were multiple, I chose the ones, which made the most sense together. The password to the other zip is `can_tchaikovsky_talk_to_skeletons_by_a_ouija?` and in it is the file `flag.txt`. The file `flag.txt` contains the flag `UMDCTF-{PY07r_11Y1CH_7CH41K0V5KY}`.