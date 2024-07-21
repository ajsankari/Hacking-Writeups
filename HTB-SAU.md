
# Details: 
IP :  10.129.66.57

## NMAP SCAN:

![](attachment/8778ed22912712e9d47ea9e3dd464b8a.jpg)]

Ports open:
22/tcp open SSH
80/tcp filtered http 
55555/tcp open unknown


cant get on port 80 as its filtered
lets try port 55555

![](attachment/ab5fb3d765ed048daec7f04c5d77775f.png)


![](attachment/9bcea793bd4359003a0da115b775e2dd.png)

We can now send a request to the basket using curl.

![](attachment/2e0426f5b5247fe6d1e3e23a9be03d85.png)

test with SSTI with **curl http://10.129.229.26:55555/objl6op -A "{{ hello }}"/'hi'**

but the user agent shows the {{ hello }} so it is not vulnerable.

![](attachment/590fdcf4e9033cebfbcd8b37f05bf9d6.png)


# LOOKING ELSEWHERE

![](attachment/fa61551464d79329de00a9857b32f9c9.png)
powered by request-baskets and version


request-baskets cve found on google allows us to use SSRF (Server Side Request Forgery)

CVE : CVE-2023-27163
![](attachment/c2b3d9cdd3564a107dca61352f9eec40.png)
![](attachment/03a0233a7da4482a2303cf99f993dba8.png)

![](attachment/59a9531cdec62775042a2540d3711e0a.png)
Looking on google I find a RCE exploit for Maltrail on github

RCE Exploit https://github.com/spookier/Maltrail-v0.53-Exploit


![](attachment/c4290cc174313a76c39102f27744f23c.png)

We now can spawn a shell with the exploit.

We are now in as user puma and can find the user flag.

**user.txt c40901e3b6c50____________

# Looking at sudo privileges

Using command sudo -l we can see that we are able to run systemctl as sudo

Using GTFO bins we find that we are able to exploit this and get a root shell

![](attachment/371fe3e0f33eda17b7ccff3f9a5870e6.png)

Use priv escalation in sytemctl to get root using !/bin/bash to spawn root shell.

![](attachment/cea3b61dfc214935960185c85b1925a3.png)

We now have root and the box is complete




