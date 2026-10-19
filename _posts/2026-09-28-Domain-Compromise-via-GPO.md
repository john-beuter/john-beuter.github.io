---
title: Windows AD Attack Lab
author: John
date: 2026-09-28 06:33:00 -0500
categories: [Blogs]
tags: [Pentesting, Active Directory]
pin: true
math: true
mermaid: true
image:
 path: 
---


## Background (Executive Summary)


For this lab I was given three devices within and active directory domain to attack. I was also provided with an initial user access. We will call this user `bob`. The objective of this lab was to compromise each machine and discover corresponding user and admin flags for each device on the network. Working through this lab, I was able to compromise two of the hosts by discovering plaintext credentials, and the third device (the domain controller) I was able to compromise via have WriteOwner permissions on the domain. 

During the compromise of this domain,  I leveraged numerous new tools, discoverd numerous news attack paths, and went down countless rabbit holes. The intent of this blog post is to illustrate the work that was done, what information was found and to practice report writing. 

## Enumeration

Begining my engagement I wanted to make sure I did good enumeartion. In some of fmy previous approached to HTB the challenges or other labs of the sort,  I have noticed that I tend to jump at the first chance at an exploitable service which often causes me to overlook other potential attack pathes. This is a skill I am still working on as in this lab I fell for this is exact pitfall. So let's dive into the enumeration, where I went wrong intially and how I corrected my process. 

During my first initial pass at enumeration I scanned all three devices. Common amongst all of the devices, I saw SMB was running. Using this information I attempted to run enum4linux to enumerate SMB shares on the devices and get operating system information etc. 



During my initial enumeration of the network, I actually didn't know that I had credential accesss to the device. I tried anonymous access via SMB primarily to get access onto the domain but had no luck. When I re-read the scope document/lab instructions I saw that I had access to a user account on a device I'll call device 'A'. 

My initial nmap scan, `nmap -Pn DEVICE_A' revealed the following ports open on the device: 

```
SMB PORTS
WINRM
XXX
XXX
WEB Server
```

Upon seeing winrm open, my mind instantly jumped to using evilwinrm to gain intial access. SCREENSHOT of WINRM. I was able to gain access to the device using evilwinrm and my credentials. From there I upload PowerView, Sharphound, and WinPeas to begin my internal enumeration. From my first attempts to run Sharphound I saw that my results weren't quite complete and the graphed data didn't quite make sense. There appeared to be an issue. I tried running PowerView, but saw that my device was not allowed to run powershell. After some quick powershell work,  I got the ability to run scripts but still no luck running powerview commands. It was at this point that I wanted to run furether internal enumeration of my environment and so I ran WinPeas. From this tool output I got a ton of information about the permissions my user had on the host and other information about the host itself. 

Discovered that I had admin permissions? Discovered the secura CVE. Ran the exploit to get admin permissions and then read the root flag. Used the access that I had to run mimikatz and dump the lsa cache. From this cache I was able to discover plaintext credentials. I then used these plaintext credentials to evilwinrm connect to Target B. On target B, I uploaded Winpeas. Didn't find anything.  Dug around and found Msql running. I enumerated the database and found a creds database file. I cat'd the creds file and found two users, one admin and one for the user charlotte


DO A BLURB ABOUT WINPEAS

- I think that my exploit for secura actually worked, maybe that was how I was able to view the admin information within the file share. 

- Try using bloodhound-python to run enumeration as 'bob'



