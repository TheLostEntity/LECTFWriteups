# LFI Inclusion Challenge Writeup (TryHackMe) By LostEntity

## Introduction
This is my writeup of the [Inclusion](https://tryhackme.com/room/inclusion) room on [TryHackMe](https://tryhackme.com). This will be my second pentesting walkthrough, so feel free to provide feedback through my [Twitter](https://twitter.com/LostEntity1929)

Before starting the task, make sure you have your AttackBox/VPN connection and target system running.

## Enumeration
To begin, We'll enumerate the target system starting with nmap, using the following scan to determine what services are running on the target system and determine the exact versions of each service and the operating system running. 

`nmap -A MACHINE IP`

![Nmap Scan](https://i.imgur.com/goD8DFM.png)

The scan reveals that we have an SSH service and a python web server running on the system. For now, I will investigate, the web application running and revisit the SSH service later.

Next, we'll conduct a directory search on the web server using dirsearch. You can use gobuster scan in place of dirsearch if you do not have access to the script.

`dirsearch -u 10.10.184.163`

![Dirsearch Scan](https://i.imgur.com/lD7vC1X.png)

The scan reveals that there is only recognised directory on the web server that being /article which we cannot access as a normal user.

## Exploring the application
With the information gathered in the enumeration phase, it's time to explore the web application deployed on the webserver. Opening the machine IP in Firefox reveals the following page:

![Homepage](https://i.imgur.com/uVLORBs.png)

There is several links on the page. The learn more link does nothing, but the other three links show articles. The article we are interested in is the LFI-attack article. This article contains some information which will useful for exploiting local file inclusion vulnerability on the application.

![LFI Page](https://i.imgur.com/As00vxj.png)

This page breaks how an attacker can exploit an LFI vulnerability to reveal files and directories contained on a system. Note the URL and the parameter used to load the articles. The article mentions the following example on the third to last line:

`http://example.com/?file=../../../../etc/passwd`

Let's try applying this to our machine, modify the above url to the following and refresh the page:

`http://MACHINE-IP/article?name=../../../../etc/passwd`

If the directory is accessable, the passwd file should be loaded in the browser in the following format:

![Passwd file](https://i.imgur.com/Ngq5bWD.png)

The passwd file contains the information on all the user accounts on the system. We can use this file to find the user account on the system. We will be looking for a user with a /home directory. You can use the find/search tool of your browser to find the users with /home directories. You will notice a user called **falconfeast** in the passwd file. The passwd file also reveals the user's password.

## Gaining Access
Using the username and password from the exploration phase, we can login into the SSH server as the falconfeast user and collect the user flag. 

![User Flag](https://i.imgur.com/pbamAyX.png)

## Escalating to root
Now we have access to the server, we can now look to escalate our privileges to root. To begin, we'll use `sudo -l` to see if the falconfeast user can run any programs as sudo. The results of this query reveal that the user can run socat with elevated privileges.

![Sudo -l results](https://i.imgur.com/XVXoqAI.png)

We'll find a method for privilege escalation using [GTFOBins](https://gtfobins.github.io/). GTFOBins has several options for escalating privileges using socat. For this task, we'll use the sudo payload to escalate privileges using the following command in our SSH shell:

`sudo socat stdin exec:/bin/bash`

Note this will not spawn a full shell, but it will partial shell will be enough for our purposes. Spawn the partial shell and confirm you are accessing the system as root and determine what directory we are currently in:

![Root Access](https://i.imgur.com/rDH9u1d.png)

To obtain the root flag, we will need to navigate to the root directory using cd:

`cd ../../root`

Once you are in the root directory, you check for the **root.txt** and use `cat root.txt` to reveal the root flag.

![Root Flag](https://i.imgur.com/rUJ1HkA.png)
