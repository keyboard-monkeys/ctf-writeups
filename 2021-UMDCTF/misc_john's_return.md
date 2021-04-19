We get a pcapng file with 802.11 association request, wpa2 handshake and encrypted traffic.

From the association request, we can get the SSID `linksys`. I converted the pcapng to pcap wiht https://pcapng.com/ to use with aircrack-ng and downloaded the rockyou password list from https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt.
Then I used the command `aircrack-ng -a2 -w rockyou.txt received.s0i0.pcap` to crack the password, which was `chocolate`.

Using the SSID and password, we can decyrpt the traffic using wireshark. The encrypted traffic had only one packet repeated multiple times. In the packet there was hex `554d444354462d7b77683372335f6a30686e7d`, which can be translated to ascii `UMDCTF-{wh3r3_j0hn}`.