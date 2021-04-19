We are given the website https://azeemsm-umdctf.github.io/ and the fact, that we should look at the first project. The sites source and the projects are uploaded to github. https://github.com/azeemsm-umdctf/azeemsm-umdctf.github.io

The first project used a Rasbperry pi to send an email with the code in program.py. Looking at commit history, we can see, that in the beginning, the program sent a POST request to an IFTTT webhook with the data containing an email address. We can find the key file also from the commit histoy, before it was deleted. We can modify the code a bit to send the email to our own email address.

```python
import requests

my_data = { "value1" : "youremailaddress", "value2" : "test" }
r=requests.post("https://maker.ifttt.com/trigger/ring/with/key/iWEKorMwH5rgGjyHb4jzedP9m9LA7yNkTZ0dvfzDSUO", data=my_data)
print(r.text)
```

The flag `UMDCTF-{g!t_h00k3d}` gets sent to your email.