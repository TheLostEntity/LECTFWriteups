# TryHackMe - Investigating Windows Room Report

## Introduction
This is my writeup of the [Investigating Windows](https://tryhackme.com/room/investigatingwindows) room on [TryHackMe](https://tryhackme.com). This will be my second pentesting walkthrough, so feel free to provide feedback through my [Twitter](https://twitter.com/LostEntity1929)

Before starting the task, make sure you have your AttackBox/VPN connection and target system running.
## Question 1 - What's the version and year of the windows machine?
There are a number of ways you can access this information. The first method is to open the properties of the system through Start Menu. Right click on the start button, select "System" and you'll find the information listed under "Windows editions".

![File Explorer](https://i.imgur.com/CWCogGs.png)

The second method is to open the command prompt (cmd) and enter the command:
`systeminfo`

This will load up various information on the server, including the version and year for the operating system. This will be listed at the very top under "OS Name".

![Systeminfo](https://i.imgur.com/9RsQ7nF.png)

## Question 2 - Which user logged in last?
To find out which user logged on last time, you'll need to access the Event Viewer, navigate to Windows Logs/Security and filter the events using the Logon Event ID: 4624. You will be looking for the most recent logon event. The username will be displayed in the New Logon section under Account Name.

![Event Viewer](https://i.imgur.com/lF0QePw.png)

## Question 3 - When did John log onto the system last?
NOTE: Make sure you follow the format provided: MM/DD/YYYY H:MM:SS AM/PM

To find out the last time John logged onto the server, you'll need to open the command prompt and enter the following command:
`net user john | findstr /B /C:"Last logon"`

This command will grab all the user information for John and then filter out everything except the last logon time and date.

![John Last Logon](https://i.imgur.com/XuKKRU5.png)

## Question 4 - What IP does the system connect to when it first starts?
When you logon to the Windows machine, upon startup a command prompt will appear running "PsExec" a program used to execute processes from a remote system. The IP address that the system connects to is shown in this cmd window.

![First IP](https://i.imgur.com/KtTtZcc.png)

## Question 5 - What two accounts had administrative privileges (other than the Administrator user)?
To find the other administrator accounts, begin by opening the **Computer Management** utility either through the Tools menu in the Server Manager or by right clicking the start menu and selecting **Computer Management**. From there, Select `System Tools/Local Users and Groups/Groups` then from the Group menu, select the Administrators group, this will open a properties box which will contain a list of all the members in that group.

![Admin Users](https://i.imgur.com/9EqBazB.png)

## Question 6 - Whats the name of the scheduled task that is malicous?
To begin, open the **Computer Management** utility. We are looking for scheduled task on the system, so we will navigate to the **Task Scheduler** and open the **Task Scheduler Library**. This menu will all scheduled tasks, there triggers, next run time, last run time and the result of the previous run. There are several tasks in scheduler, but one stands out, look at the each of the tasks and the actions they execute. You will be looking for the task modifies the system.

![Malicious Process](https://i.imgur.com/2OvcCzU.png)

## Question 7 & 8 - What file was the task trying to run daily? & What port did this file listen locally for?
Once you have found the malicious task, you can find the file that the task runs and the port it runs on by opening the **Actions** submenu on the task and the file and port will be lasted in the details for the action. 

![Port and Process](https://i.imgur.com/zGRilnC.png)

## Question 9 - When did Jenny last logon?
To find out the last time Jenny logged onto the server, you'll need to open the command prompt or Powershell and enter the following command:
`net user jenny | findstr /B /C:"Last logon"`

This command will grab all the user information for Jenny and then filter out everything except the last logon time and date.

![Jenny Logon](https://i.imgur.com/LastD1v.png)

## Question 10 - At what date did the compromise take place?
The date of system compromise will be the date in which the malicous task was created, this can be found in the **Task Scheduler Library** under the malicous task entry in the Created column.

![Date of compromise](https://i.imgur.com/WcVFGlK.png)

## Question 11 - At what time did Windows first assign special privileges to a new logon?
To find this we will begin opening the Computer Management utility again and navigate to the Event Viewer, then open `Windows Logs/Security`. To filter out any irrelevant logs we will apply a filter to the logs and specify events with the ID **4672**. This is the ID for special logon events. You will want to sort the entries by Date and Time with the oldest entries appearing first. The question hint will help you narrow down the entries to one.

![Special Logon](https://i.imgur.com/ZKs2LTs.png)

## Question 12 - What tool was used to get Windows passwords?
We can use the Task Scheduler Library to help us find this tool. If we look in the Scheduler, there is another suspicous task other than the previous task. You should be able to spot it by looking at the names of each task. Once you find it, open the actions submenu. This will show the file being executed. Researching for executable file will provide you with the answer to the question.

![Passwords](https://i.imgur.com/jIP2IDJ.png)

## Question 13 - What was the attackers external control and command servers IP?
To find C&C server's IP address, we will have to open the system's host file. This file is typically used to map IP addresses to hostnames on [Windows.](https://en.wikiversity.org/wiki/Hosts_file/Edit) 

This file is located in `C:\Windows\System32\drviers\etc` and is plain text, meaning we can view it in Notepad. You will notice a few normal entries and then a couple of irregular entries that are mapped to commonly used website. It is safe to assume the IPs mapped to this site are for the Control & Command server as a common website like this would not need manual DNS mapping. 

![Hosts File](https://i.imgur.com/ikJpmY5.png)

## Question 14 - What was the extension name of the shell uploaded via the servers website?
To find the shell we first need to locate the directory for website on the system. If we look at the Local Disk, there are several directories. The **inetpub** folder is the one we will focus our attention on as this is the folder used for Microsoft Internet Information Services. Opening this folder reveals a second folder **wwwroot**. This is the web root folder and will contain the files hosted on the webservice, including the shell. When open the folder you will find three files. The shell files will be script files.

![Extension](https://i.imgur.com/VAJF1Q1.png)

## Question 15 - What was the last port the attacker opened?
To determine the port the attacker opened, we will look at the Windows Firewall utility more specifically the Inbound Rules as the attacker would need to modify these rules to allow traffic from the Control and Command (C&C) server they are operating. Looking at the inbound rules, one sticks out. The **Allow outside connections for development** rule at the top of the list allows any program to be executed and the port can be accessed by any local or remote address. That rule is allows anyone to access that specific port from any address and execute any programs they like. This rule has obviously been engineered to allow the C&C server to communicate with this system.

![Firewall](https://i.imgur.com/YmaFZP2.png)

## Question 16 - Check for DNS poisoning, what site was targeted?
Refer to the answer for question 13. The website mapped to the C&C server in that scenario is the victim of manual DNS poisoning by the attacker.
