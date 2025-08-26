---
layout: default
---

[Back to main page](https://ornaka.github.io)

Writeup about TryHackMe's room Volt Typhoon

Link to the room:

[https://tryhackme.com/room/volttyphoon](https://tryhackme.com/room/volttyphoon)

# Scenario

Scenario: The SOC has detected suspicious activity indicative of an advanced persistent threat (APT) group known as Volt Typhoon, notorious for targeting high-value organizations. Assume the role of a security analyst and investigate the intrusion by retracing the attacker's steps. You have been provided with various log types from a two-week time frame during which the suspected attack occurred. Your ability to research the suspected APT and understand how they maneuver through targeted networks will prove to be just as important as your Splunk skills. 

Remember to swap time frame to all time from the top right corner of Splunk.

## Task 1 - Initial Access

Volt Typhoon often gains initial access to target networks by exploiting vulnerabilities in enterprise software. In recent incidents, Volt Typhoon has been observed leveraging vulnerabilities in Zoho ManageEngine ADSelfService Plus, a popular self-service password management solution used by organizations.

### Question 1 

Comb through the ADSelfService Plus logs to begin retracing the attacker’s steps. At what time (ISO 8601 format) was Dean's password changed and their account taken over by the attacker?

### Question 2

Shortly after Dean's account was compromised, the attacker created a new administrator account. What is the name of the new account that was created?

I used the following query, which showed events containing answer to both question 1 and 2 

index=main "AD*" username="dean-admin" "Password" 
| sort -_time desc

![Events showing Deans account being compromised](/images/pic1.png)

## Task 2 - Execution

Volt Typhoon is known to exploit Windows Management Instrumentation Command-line (WMIC) for a range of execution techniques. They leverage WMIC for tasks such as gathering information and dumping valuable databases, allowing them to infiltrate and exploit target networks. By using "living off the land" binaries (LOLBins), they blend in with legitimate system activity, making detection more challenging.

### Question 3 

In an information gathering attempt, what command does the attacker run to find information about local drives on server01 & server02?

Query I used: index=main sourcetype="wmic" "*server02*" OR "*server01*"

![Result of the query](/images/pic2.png)

### Question 4 

The attacker uses ntdsutil to create a copy of the AD database. After moving the file to a web server, the attacker compresses the database. What password does the attacker set on the archive?

At first I queried ntdsutil but it didn’t give the answer right away. However the events shown revealed name of the database dump, which I searched and found the answer

![Result](/images/pic3.png)

## Task 3 - Persistence

Our target APT frequently employs web shells as a persistence mechanism to maintain a foothold. They disguise these web shells as legitimate files, enabling remote control over the server and allowing them to execute commands undetected.

### Question 5 

To establish persistence on the compromised server, the attacker created a web shell using base64 encoded text. In which directory was the web shell placed?

For this one had to check Volt Typhoon’s MITRE ATT&CK-page at [https://attack.mitre.org/groups/G1017/](https://attack.mitre.org/groups/G1017/)

One of the references had the answer

![Screenshot from a reference](/images/pic4.png)

## Task 5 - Defence Evasion

Volt Typhoon utilizes advanced defense evasion techniques to significantly reduce the risk of detection. These methods encompass regular file purging, eliminating logs, and conducting thorough reconnaissance of their operational environment.

### Question 6 

In an attempt to begin covering their tracks, the attackers remove evidence of the compromise. They first start by wiping RDP records. What PowerShell cmdlet does the attacker use to remove the “Most Recently Used” record?

Since cmdlets for removing files start with remove used “remove*” as query. MRU in the Windows logs stands for Most Recently Used, which was also mentioned in the question.

![Query result](/images/pic5.png)

### Question 7 

The APT continues to cover their tracks by renaming and changing the extension of the previously created archive. What is the file name (with extension) created by the attackers?

I saw this file earlier while looking for the password for the database, so I used 7z as query since it was the archiving tool apt used to pack database dump.

![Event](/images/pic6.png)

Under what regedit path does the attacker check for evidence of a virtualized environment?

Used a query "virtual" which got me a hit

![Regedit path](/images/pic7.png)

## Task 6 - Credential Access

Volt Typhoon often combs through target networks to uncover and extract credentials from a range of programs. Additionally, they are known to access hashed credentials directly from system memory.

### Question 8 

Using reg query, Volt Typhoon hunts for opportunities to find useful credentials. What three pieces of software do they investigate? Answer Format: Alphabetical order separated by a comma and space.

Queried "reg query*" 

![Event containing answer](/images/pic8.png)

What is the full decoded command the attacker uses to download and run mimikatz?

This was a hard one for me. I used index=main sourcetype=”powershell”| sort asc as a query and scrolled through the results. Eventually found the right answer. As a note, there was one encoded command beforehand which I actually found out to be the webshell download asked in question 4. After finding the right one decoded the string with CyberChef

![Decoded string in CyberChef](/images/pic9.png)

## Task 7 - Discovery & Lateral Movement

Discovery
Volt Typhoon uses enumeration techniques to gather additional information about network architecture, logging mechanisms, successful logins, and software configurations, enhancing their understanding of the target environment for strategic purposes.

Lateral Movement
The APT has been observed moving previously created web shells to different servers as part of their lateral movement strategy. This technique facilitates their ability to traverse through networks and maintain access across multiple systems.

### Question 9 

The attacker uses wevtutil, a log retrieval tool, to enumerate Windows logs. What event IDs does the attacker search for? Answer Format: Increasing order separated by a space. (Don’t be like me and ignore the separated by space part and wonder why answer isn’t working with commas) 

Used "wevtutil" as query and checked the EventID-field from fields-bar on the left to get the ID's

![Three ID's](/images/pic10.png)

### Question 10 

Moving laterally to server-02, the attacker copies over the original web shell. What is the name of the new web shell that was created?

I used "server-02" "copy" as query and found only one event, which contained correct answer.

![Event](/images/pic11.png)

## Task 8 - Collection

During the collection phase, Volt Typhoon extracts various types of data, such as local web browser information and valuable assets discovered within the target environment.

### Question 11

The attacker is able to locate some valuable financial information during the collection phase. What three files does Volt Typhoon make copies of using PowerShell?
Answer Format: Increasing order separated by a space.

Queried "PowerShell" "copy" and found an event containing copying of one of the files. Added "*.file-extension" to the query and found the three files.

![](/images/pic12.png)

## Task 9 - C2 & Cleanup

C2
Volt Typhoon utilizes publicly available tools as well as compromised devices to establish discreet command and control (C2) channels.

Cleanup
To cover their tracks, the APT has been observed deleting event logs and selectively removing other traces and artifacts of their malicious activities.

### Question 12 

The attacker uses netsh to create a proxy for C2 communications. What connect address and port does the attacker use when setting up the proxy? Answer Format: IP Port

Queried only "netsh" and found an event containing the IP address and the port number

![](/images/pic13.png)

### Question 13

To conceal their activities, what are the four types of event logs the attacker clears on the compromised system?

Googled wevtutil and found out it can be used to clear logs with a cl-switch, so queried "wevtutil cl*" and found the exact event I was looking for.

![](/images/pic14.png)
