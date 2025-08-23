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

![Events showing Deans account being compromised](/images/pic1.png)

