![intro](https://github.com/user-attachments/assets/e1d0f3d1-a767-446b-a75f-6b857c6f4137)

This is a walkthrough for the tryhackme CTF Thompson. I will not provide any flags or passwords as this is intended to be used as a guide. 

## Scanning/Reconnaissance

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -sV -sC -A -v $ip -oN
```

Command break down:

-sV

Service Version Detection: This option enables version detection, which attempts to determine the version of the software running on open ports. For example, it might identify an HTTP server as Apache with a specific version.
-sC

Default Scripts: This option runs a collection of default NSE (Nmap Scripting Engine) scripts that are commonly useful. These scripts perform various functions like checking for vulnerabilities, gathering additional information, and identifying open services. They’re a good starting point for gathering basic information about a host.
-A

Aggressive Scan: This option enables several scans at once. It combines OS detection (-O), version detection (-sV), script scanning (-sC), and traceroute (--traceroute). It’s useful for a comprehensive scan but can be intrusive and time-consuming.
-v

Verbose Mode: Enables verbose output, which provides more detailed information about the scan’s progress and results.
$ip

Target IP: This is a placeholder for the target IP address you want to scan. In practice, replace $ip with the actual IP of the machine you are targeting.
-oN

Output in Normal Format: This option saves the scan results in a plain text file format. After -oN, specify a filename where you want to store the output.

The scan reveals three open ports including a simple http web server on port 8080.

![nmap](https://github.com/user-attachments/assets/7dfd6640-2b53-4d6d-bcda-b1e25ecf49b9)

When we visit the ip:8080 we get the default apache tomcat page. Let's run a gobuster scan on $ip:8080.

![buster](https://github.com/user-attachments/assets/0d164a8c-413a-4726-9167-fb3c4ceac7d5)

When we visit the /manager directory a pop up prompts for user and password. I tried some of the usual defaults but no luck. However, when we click cancel it shows us some default credentials, I also found that these are common default credentials:

![creds](https://github.com/user-attachments/assets/6425c364-4cf7-414a-b4fc-b106bce24ce7)

Refresh the page and use those credentials to log into the manger site. 
Here we see a listing of directories plus an upload panel for WAR files. Web Application Archive (WAR) files is a packaged file format used to deploy web applications on Java based servers, like tomcat. Let's use msfvenom to create a war file reverse shell payload:

![msf](https://github.com/user-attachments/assets/4076239d-b668-4fb2-9f6a-103313808642)

Now, we just need to upload the file. Then set up a net cat listener on your choosen port. After that, we just need to click on the war file that is listed above, amoung the other directory listings. And we have a shell as tomcat!

![shell](https://github.com/user-attachments/assets/91b3499b-8fd4-4ab7-9d68-2d5f70d4db42)

When we navigate to the home directory, we see user.txt in jack's directory:

![user txt](https://github.com/user-attachments/assets/6cb6d8ac-dd18-4c75-9819-a28e55739495)

# Privilege Escalation

In jack's home directory we see an id.sh file, and a test.txt file:

![id sh](https://github.com/user-attachments/assets/c37def8b-c2c7-4274-b638-c1eed36db692)

By looking at test.txt we can see that id.sh is being run as root, based on the output of id in test.txt. We can also write to id.sh, so we can use this to escalate to root, when id.sh is being executed by a cronjob every minute.

We will write to id.sh and add the line chmod u+s /bin/bash to set the SUID bit for /bin/bash, which will alow us to run /bin/bash as root:

![exploit](https://github.com/user-attachments/assets/b2a9bcfa-915f-4ebb-9317-77560acbd143)

From there, we just need to head to the root directory and cat our root.txt. 

![root txt](https://github.com/user-attachments/assets/c415d9af-b440-4ded-a306-29e3d2e44555)

I hope you enjoyed this CTF.







