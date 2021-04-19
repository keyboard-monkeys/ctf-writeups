This is the second challenge of the advantageous adventures series.

Using the server, username and password obtained from the first challenge, an SSH session can be opened on that server. The user doesn't have too many privileges, but can run `dumpcap`. This is interesting because a WiFi password was also provided. The `eth0` interface hosts the SSH session, but `ip link` shows another interface called `wlo2`. The previously obtained script expects at least 2153 packets, so that's how long `dumpcap` will run on the interface.

Transferring the packet capture to a local machine (using `scp`), it runs through the script obtained from the first challenge to produce a WiFi packet capture. Among the packets is the SSID of the network, which turns out to be the flag `UMDCTF-{sp00ky_sh@rky}`.
