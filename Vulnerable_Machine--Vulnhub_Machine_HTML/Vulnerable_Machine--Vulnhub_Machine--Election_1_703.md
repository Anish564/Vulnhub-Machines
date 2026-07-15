::::::::::: page
# Election: 1 {#election-1 .title}

\

## 

## Election: 1

- **[Election: 1]{style="color:#237522;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/election-1,503/>

![](images/703-1.png)

- Now extract the file .
- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#9141ac;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.31.0/24
:::

![](images/703-2.png)

- Find available port in the machine :

::: codebox
    nmap -v -p- 192.168.31.60
:::

![](images/703-3.png)

::: codebox
    nmap -sC -sV -A 192.168.31.60
:::

![](images/703-4.png)

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.31.60
:::

![](images/703-5.png)

1.  [Web Enumeration]{style="color:#9141ac;"} :

- IP visit in browser : <http://192.168.31.60/>
  <http://192.168.31.60/phpinfo.php> <http://192.168.31.60/phpmyadmin/>
  <http://192.168.31.60/robots.txt>

![](images/703-6.png)

- In election parameter is available : <http://192.168.31.60/election/>

![](images/703-7.png)

- Now run the gobuster for directory brute force :

::: codebox
    gobuster dir -u http://192.168.31.60/election/ -w /usr/share/wordlists/dirb/common.txt
:::

![](images/703-8.png)

- Visit the parameter : <http://192.168.31.60/election/data/>
  <http://192.168.31.60/election/admin/>

![](images/703-9.png)

- Again brute force in /admin parameter :

::: codebox
    gobuster dir -u http://192.168.31.60/election/admin/ -w /usr/share/wordlists/dirb/common.txt
:::

![](images/703-10.png)

- Visit found parameter : <http://192.168.31.60/election/admin/logs/>

![](images/703-11.png)

- Open system file :
  <http://192.168.31.60/election/admin/logs/system.log>

![](images/703-12.png)

1.  [Get the SSH connection on this machine]{style="color:#9141ac;"} :

::: codebox
    ssh love@192.168.31.60
:::

::: codebox
    user: love 
    Password: P@$$w0rd@123
:::

![](images/703-13.png)

- [Key Features]{style="color:#9141ac;"} :

<!-- -->

- Discovered open ports 80 (HTTP) and 22 (SSH) using Nmap
- Enumerated web app and found credentials in logs (love /
  P@\$\$w0rd@123)
- Login panel was misleading, so used credentials for SSH access
- Gained initial shell as user love

<!-- -->

- [Summary]{style="color:#9141ac;"} :

<!-- -->

- Used exposed credentials to gain SSH access through system
  misconfiguration .
:::::::::::
