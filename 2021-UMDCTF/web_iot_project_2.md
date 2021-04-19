This is a continuation to IoT Project.

https://azeemsm-umdctf.github.io/
The source is in github. https://github.com/azeemsm-umdctf/azeemsm-umdctf.github.io

The second project is described on the website as a tweeting toaster. From the commit history for `robots.txt`, we can find the following lines:
```
IOT PROJECT 2 IDEA: Use maker event "tweet" so that my toaster can POST to twitter on my secret account when it is ready in the morning. That way I'll know when to get off Twitter and eat my breakfast!
    TODO: setup IFTTT

oh wait that was super easy to setup my second project... value1=normal text, value2=hashtag
```

From that we know, which POST request we should send and where to send it to.
We create the request with the command `curl -X POST -F 'value1=Keyboard_Monkeys_yee' -F 'value2=Keyboard_Monkeys_yee' https://maker.ifttt.com/trigger/tweet/with/key/iWEKorMwH5rgGjyHb4jzedP9m9LA7yNkTZ0dvfzDSUO`. With this, we can search twitter for Keyboard_Monkeys_yee for the Twitter account.

The account is `https://twitter.com/r31qrfwear23231` and in the description is the hex `554d444354462d7b7477336574216e675f346e645f7430407374316e677d`, which can be translated to ascii `UMDCTF-{tw3et!ng_4nd_t0@st1ng}`.


During the competition the webhook didn't work and the author manually tweeted, if we DM-d the correct POST request.