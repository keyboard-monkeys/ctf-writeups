# OSINT/Find the location

Description:

```
I got important intel about 10-0052 at 01-01-2020. Can anyone find the flight where it traveled (from country to country).
```

------

Searching up `10-0052` on google shows that it is a military aircraft.  Searching google for `military flight radar` yields a link to `https://www.ads-b.nl/`, which allows historical searches for free. Searching for movements on 01/01/2020 yields a handy map, which shows the plane started in the UK and landed in France.

After verifying the flag format with the admin, we got

`darkCON{United_Kingdom_to_France}`