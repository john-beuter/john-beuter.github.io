---
title: Attacking ADCS and Exploiting ESC 8 
description: An overview of how I acheive Domain Admin in my first active directory internal pentest
author: John
date: 2025-06-24 18:30:00 -0500
categories: [Internship]
tags: [Active Directory]
pin: true
math: true
mermaid: true
image:
  path: assets/img/amusement.jpg 
---


### Mistakes from the Past: The Return to AD


As some of my previous blog posts have highlighted, expanding my knowledge of Active Directory (AD) has been a goal I've had in mind for about a year. My initial attempts at learning AD were all over the place. I started learning AD by building out my domain controller, creating different users, and joining computers to it. I am going to be honest, while I learned a lot, I don't feel like the concepts really stuck.


Over time, as I grew as both an engineer and a student of cybersecurity, I went down numerous other (non-AD) learning rabbit holes and gradually lost touch with what I had learned. It wasn't until May of 2026, over a year after I first dipped my toes into the world of AD, that I had the opportunity to expand my understanding of AD yet again.




### The Environment and Gaining Initial Access


Throughout my time as a pentester up until this engagement, my experience had been limited to a smaller corporate environment that did not utilize AD. During these smaller engagements, I could run basic enumeration with nmap or Nessus and manually test all of my findings. This primitive strategy worked when the network had 10-15 devices, but was entirely infeasible for the network I found myself in, which had over 1,000 devices.


My initial position on the network was a Kali device plugged into the client's network. My user/device was not domain-joined, and I was not provided with a user account. At the start of my enumeration, I was dead set on authenticating to the domain. Starting out, I tried the basics, like SMB null session enumeration and anonymous LDAP channel binding, on the Windows devices I found in the domain. Nope. No luck. I started Responder and hoped to get an account's hash. I did eventually capture a hash, but was ultimately unable to crack it.


I shifted my focus to other devices on the network. My initial enumeration revealed numerous printers and security cameras that were running older operating systems with known vulnerabilities. Bingo. If I could exploit one of these devices and gain access to it, I might get access to the domain IF the device is domain-joined. After doing some poking around online, I found a remote code execution exploit for one of the cameras on the network. I tested the code and was able to gain access to the device as a superuser. Awesome. Unfortunately, the device was not domain-joined and contained no credentials for domain users. Not awesome.


After these initial, quick exploit attempts, I was at an impasse. I didn't know what to do. Fortunately for me, I did have one factor working in my favor. The organization I was testing had a publicly available staff directory. Using this staff directory, I was able to create a list of potential users on the domain. I verified this list of potential users on the domain using the enumeration/brute-forcing tool Kerbrute. After I identified which users were valid, I adjusted my list and began password spraying using various commonly used password conventions. Not longer after I started my password spraying, I got a hit. It was a basic user account, but it gave me a foothold to claw my way through the domain. And so it began.






### The Path to My First Domain Admin Compromise


After gaining initial access to the domain, I wanted to see what permissions and privileges were allotted to my compromised user. My main method of enumeration was using the Bloodhound-Python tool, which uses LDAP queries to collect information about a domain. The results from Bloodhound-Python are then passed to Bloodhound to map relationships between users and groups in the domain. Looking at the output was very overwhelming. I used built-in queries within Bloodhound to map potential vectors for privilege escalation. It was at this point in the engagement that I discovered, or rather re-discovered, the art of kerberoasting.


#### The Kerberos Amusement Park


I recall attending a talk where the speaker described the inner workings of Kerberos as an amusement park. At this amusement park, once guests pay their admission fee, they receive a receipt. Before a park-goer can ride a rollercoaster, they have to return to the ticket stand, show their receipt, and purchase a ticket to ride. Mapping this example back to Kerberos, to use different services in an AD environment, a user needs a service ticket (ST). An ST is like the tickets for the rollercoaster. If a user wants to be granted an ST, they need to provide some form of proof that they are a valid user with access to the service (they need to present their receipt). Validation comes in the form of a ticket called a ticket-granting ticket. Users receive a TGT after authenticating the key distribution center (KDC) on the network. Think of the KDC like the ticket booth at the park. Once a user has a TGT, they can request service tickets for various network resources. Each time a user wants to access a resource, they must first present the KDC with their TGT and the service principal name (SPN) of the service they want to access. An SPN is an identifier for a specific service account. If the TGT and service name are valid, the user is granted an ST. The ST itself is encrypted with the NT hash of the SPN, meaning that an attacker can view the hash, once the ticket has been acquired, and attempt to crack it.


While enumerating the domain using Bloodhound's built-in queries, I discovered two SPNs with Tier 0 (high-privilege) access that could be kerberoasted. I used Impackets GetUserSPN.py script to acquire the hashes of each SPN. After hours of waiting for the hashes to crack, hashcat ultimately gave me nothing. No good. Back to the drawing board. 


#### Certificates for All!


At this point in the engagement, I stepped back and asked for help. This was my first AD pentest, and I was bound to have missed some attack paths. You don't know what you don't know. It was at this point that one of my co-workers graciously gave me an impromptu lecture on the basics of Active Directory Certificate Services (ADCS). ADCS is a way to implement public key infrastructure and manage digital certificates in Active Directory. Compromising ADCS grants attackers access to password-equivalent credentials and certificates that they can abuse on the network. My coworker talked me through how, during one of their recent engagements, they used certipy-ad to exploit numerous escalation-of-privilege abuse cases (ESC) to achieve domain admin. At this point, it was game on! I had a new direction to head and a new tool to explore.


Using certipy, I scanned the network to identify the network's certificate authority. From the results of my certipy enumeration, I discovered that the ADCS implementation on the network was flagged as vulnerable to ESC8. To exploit ESC8, an attacker needs access to a network where ADCS has its web interface enabled and accepts NTLM-based authentication. With this attack path, an attacker coerces authentication from a domain controller (DC) and relays it to the ADCS web endpoint to request a certificate on the DC's behalf. Once the authentication is accepted, the attacker is granted a certificate that essentially identifies them as a DC. The attacker can then present that certificate to the KDC to retrieve a TGT for a domain controller account. With the  TGT, they can dump hashes from the DC.


To set up this attack, I used two terminals. One terminal was running Petite Potam, which required authentication from the DC, and another terminal was running certipy-ad with the relay option enabled. After running both terminals in tandem, I successfully relayed NTLM authentication to the ADCS endpoint and retrieved a DC TGT. With this ticket I then used Impacket's secrets dump to dump NT/NTLM hashes and other secrets for the domain. 


### Lessons Learned and Conclusions


In this engagement, I was really able to explore and learn deeply. The more I got stuck in ruts and failed, the more I learned and understood how AD works. From this experience, I have created a laundry list of attack paths that I want to explore more. Even though my internship is wrapping up, I am excited to continue learning and exploring AD in my free time.


#### Lessons


Keep an organized terminal and note sheet.
 - Consistent folder structure on the attacker device to document what attacks were attempted was super convenient and really helped me write the report.
 - On a similar note, taking detailed notes on each attack path I used or tried helped reinforce my learning.


Enumeration and understanding of the system as a whole is key
 - The more time I spend understanding how AD works and how systems interact, the better I get at being able to find attack paths (shocker, I know)


AWK, Sed, and Grep all the way
 - Using these commands, I could strip tool output and format it for my reports. This saved me time and effort on such a large network.


Check LDAP signing and channel binding. If they are disabled, run responder with ntlm-relay to capture and relay output.


Check the machine account quota as soon as you get access to the domain.


#### Attacks I Want to Test in the Future


  - LazarusWakeUp by Nikos Vourdas src: https://github.com/nickvourd/LazarusWakeUp
  - RBCD
  - Shadow Credentials
  - PrintNightmare

