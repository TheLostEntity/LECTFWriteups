# Net Sec Challenge Writeup (TryHackMe) By LostEntity

This writeup will cover the [NetSec Challenge](https://tryhackme.com/room/netsecchallenge) room on [TryHackMe](https://tryhackme.com/)

## Enumeration/Nmap scans
To start, we need to indentify the common ports (ports below 10000) on the system. To get this information we'll do the following nmap scan:
`nmap -sV -sC MACHINE-IP `

This scan will reveal all the open ports below port 10000, along with the services running and their versions. The scan will also use nmap's default script, although specifying the `-sC` flag is optional.

![Nmap Scan 1](https://i.imgur.com/16HF0ot.png)

The results indicate the server is running an openSSH server (port 22), an Apache web server (port 80) and a Node.js service. We'll focus in on the Apache & SSH server information as they are relevant to the task. The scan has revealed the versions for each server and the header information. These headers will contain the flags you need to question 4th and 5th question. You can also obtain these headers using the telnet command.

To find the port above port 1000, we will use the same scan, but add an additional flag which `-p 10000-65535` to direct Nmap to scan the ports within this range. 

![Nmap Scan 2](https://i.imgur.com/F3cd3y9.png)

The results of the scan indicate that there is an FTP server (vsftpd) running on a nonstandard port (port 10021). The scan also indicates the version of vsftpd being used (version 3.0.3). From both scans, we have determined that there are 6 open tcp ports on the server (5 standard and 1 nonstandard).

## Gaining Access
Now we have enumerated all the ports and services, we can now look to gain access into the system. Our best entry point will be FTP server, as we have already obtained the two usernames for that server through social engineering. These usernames are:
- eddie
- quinn

Since we do not have the passwords for either user, we will have to bruteforce them using Hydra and the **rockyou.txt** wordlist. To make things easier, we'll also create a text file called **users.txt** which will contain both of the usernames, so we can bruteforce both users in the same attack. The command will read like this:
`hydra -L users.txt -P rockyou.txt ftp://MACHINE-IP:10021`

Adjust the location of users.txt and rockyou.txt to match the locations for the files on your system. The `-L` option indicates the users you want to attempt to login as and -P indicates the password list you are using to bruteforce the passwords. 

This scan may take awhile depending on the system resources you have allocated. 

![Hydra Bruteforce](https://i.imgur.com/kkFITwL.png)

The scan will reveal the passwords for both users, which you can then use to login into the ftp server as those users. 

With this information we can login into ftp server, first as eddie and see if we can find the flag! Running the `ls` command on the FTP server while logged into eddie's account reveals there are no files stored under eddie's account.

![FTP Eddie's Account](https://i.imgur.com/72H1DBi.png)

This means that the flag must be stored under quinn's account. We now log into the ftp server and quinn and repeat the `ls` command a second time. This reveals a text file called **ftp_flag.txt** which contains the ftp server flag. To reveal the flag, use `less ftp_flag.txt`, this will output the contents of the ftp_flag.txt and reveal the flag.

![FTP Quinn's Account](https://i.imgur.com/YSTgauF.png)

## Final Nmap scan
For the final task, we need to open the web application that is running on the server in a web browser and perform a covert nmap scan to retrieve the flag. Navigate to:
`MACHINE-IP:8080`

In your terminal window, you will want to perform the following nmap scan:
`sudo nmap -sN MACHINE-IP`

To perform this scan you will need root permissions, hence the use of sudo at the start of the command. The **-sN** sets the scan type to NULL scan, meaning that nmap will not set any flags for the scan, which ensures that the packet is not detected by any Intrusion Detection System. I would recommend having the terminal window and the browser window open side-by-side so you can monitor the web application in real-time.

![Covert Nmap Scan](https://i.imgur.com/62GpRmt.png)

The scan will the 5 TCP ports from the first nmap, but this time the state of these ports will be open | filtered instead of just open. If the scan has worked correctly, the percentage shown on the web application should go up and the flag will appear below the **reset packet count** button.
