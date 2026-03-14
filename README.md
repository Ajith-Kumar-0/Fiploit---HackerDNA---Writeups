# Fiploit---HackerDNA---Writeups
hackerdna - writeups - walkthrough - Fiploit

# FiPloit CTF Writeup

## Lab Information

Platform: HackerDNA  
Lab Name: FiPloit  
Difficulty: Easy  

Lab Link:
https://hackerdna.com/labs/fiploit

This lab focuses on exploiting insecure file operations in a PHP web application.

--------------------------------------------------

## Objective

The goal of the lab was to obtain:

- User Flag
- Root Flag

by exploiting vulnerabilities in the web application.

--------------------------------------------------

## Attack Path

The exploitation process involved the following steps:

1. Service discovery
2. Directory enumeration
3. Information disclosure
4. File upload bypass
5. Remote Code Execution
6. Privilege escalation

--------------------------------------------------

## 1. Service Discovery

The web application was accessible on port 8080.

Example access:

http://TARGET_IP:8080

A typical scan used to identify this would be:

nmap -sC -sV -p- TARGET_IP

--------------------------------------------------

## 2. Directory Enumeration

Using Gobuster to discover hidden files and directories.

Command used:

gobuster dir -u http://TARGET_IP:8080 \
-w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt \
-x txt,php,log,bak

This revealed several resources including:

/notes.txt  
/upload_log_temp4.php

--------------------------------------------------

## 3. Information Disclosure

Accessing:

http://TARGET_IP:8080/notes.txt

revealed internal developer notes referencing a temporary upload endpoint and other configuration reminders.

Example notes included:

Remove /upload_log_temp4.php before prod  
Database creds: root:Password123!  
Disable debug  

This represents Sensitive Information Disclosure.

--------------------------------------------------

## 4. File Upload Bypass

The upload functionality only allowed files with the .txt extension.

Uploading:

shell.php

was blocked.

However the validation only checked whether ".txt" appeared in the filename.

Uploading the following bypassed the restriction:

shell.txt.php

This worked because the application detected ".txt" in the name while the server interpreted the final extension ".php".

--------------------------------------------------

## 5. Remote Code Execution (RCE)

After uploading the file, commands could be executed through the browser.

Example:

http://TARGET_IP:8080/uploads/shell.txt.php?cmd=whoami

The server returned the user running the web service.

At this point Remote Code Execution was achieved.

--------------------------------------------------

## 6. System Enumeration and User Flag

After gaining command execution, basic enumeration commands were used:

id
pwd
ls /home

The user directory contained:

/home/ctf/flag-user.txt

Reading the file revealed the user flag.

--------------------------------------------------

## 7. Privilege Escalation

Checking sudo permissions:

sudo -l

Output revealed:

User ctf may run the following commands without password:
 /usr/bin/php

Allowing a user to execute PHP with sudo is a dangerous configuration.

--------------------------------------------------

## 8. Root Access

Using the PHP CLI with sudo allowed commands to run as root.

Example:

sudo php -r 'system("id");'

This confirmed execution as the root user.

The root flag was then retrieved from the root directory.

--------------------------------------------------

## Full Exploitation Chain

Service discovery on port 8080
→ Directory enumeration
→ Developer notes discovered
→ Upload endpoint identified
→ File upload bypass using double extension
→ Web shell upload
→ Remote Code Execution
→ User enumeration
→ Sudo misconfiguration exploitation
→ Root privilege escalation

--------------------------------------------------

## Lessons Learned

This lab demonstrated several important security concepts:

- Sensitive information disclosure
- Improper file upload validation
- Web shell exploitation
- Privilege escalation via sudo misconfiguration

It shows how multiple small vulnerabilities can combine to allow full system compromise.

--------------------------------------------------

## References

Lab:
https://hackerdna.com/labs/fiploit
