---
title: Building an Active Directory Testbed Environment
description: Small introduction to my learning experiences with Windows Active Directory
author: John
date: 2025-05-27 11:33:00 +0800
categories: [Projects, Security]
tags: [Active Directory]
pin: true
math: true
mermaid: true
image:
  path: assets/img/Active Directory.jpg 
---

## The long awaited return 

This post was originally published in May of 2025 as an introduction and overview of some of my Active Directory (AD) experimentation. Like all good side projects, this one was pushed to the back burner over time. In the time since I started this blog a lot has changed. I have gained a lot more experience as both an engineer and a pentester. With this post, I aim to not only revisit and redocument my first iteration of an AD lab, but also expand on the orignial goals using the knowledge of AD that I have since gained.  


## Project Overview 

 In the Spring of my Senior year (Spring of 2025), I wanted to challenge myself to learn more Active Directory (AD). So I decide to spin up a Windows server VM and join a couple of Windows VMs to the domain. After this rough lab structure was created I defined the following Red/Blue/Purple team goals for myself:
 
### Goals: 

1. Create an Active Directory server and join a Windows client to the server [Blue team]
  - Include numerous user accounts with varying levels of privledges (both local and domain accounts)
2. Implement various attack on AD [Red team]
  - Eternal blue with mimikatz for lateral movement [Stumped here in 2025]
  - Golden Ticket
  - AS-REP Roasting
3. Understand potential indicators of compromise in an AD environment as it is being exploited[Purple team]
 
 
 Looking back, these goals are great. Shoutout to past John! However; during this first exploration into the world of AD, I really had no real understanding of Active Directory. I didn't have a good understanding of the different permissions within a Windows environment, Kerberos, and a host of other things. I had to step back and do a lot of learning and come to point of understanding first before I could keep cobbling exploits together. It was the failure to recognize this need to step back and gather more information that led me to burnout my first attempt at building this lab. 

## Getting the environment back up and running

 To revist this lab project after almost a year and half of downtime I realized I needed to start from scratch. This part really was the epitome of "do it right or do it twice" and you know what, I am glad I am doing it again. So the last time I built this lab, I had three devices. I had one domain controller running the latest version of Windows server, a Windows workstation running a 2016 version of Windows, and Windows workstation running a version of Windows that should have been vulnerable to Eternal Blue. 
 
 Eternal Blue was defining attack path of my first build. Honestly the first iteration of the AD lab wasn't so much an AD lab as it was an experiment to see if I could exploit Eternal Blue on a domain joined device and perform an LSASS dump that would get me access to Domain Admin (DA) credentials. To make EB work, I had to have a prepatched version of Windows running and a vulnerable version of SMB protocol. Assuring that these two conditions were met turned out to be a major hurtle that really slowed my roll back in 2025. On my current iteration I wanted to keep things as simple as possible. My goal with this lab is to learn AD related attack paths. So I am going to focus on building a simple Domain that allows me to experiment with Kerberos, Machine Account Quotas, Resource Based Constrained Delegation, and the lot of AD attacks. 

 With this in mind, I opted to create a Domain with one Domain Controller and one workstation, both running the most up to date versions of Windows OS. Within the same network as these devices, I also set up a Kali VM to be my attacker box. This set up created a simple, attacker on network scenario that I have used in previous internal network pentests. 

#### Setting up a Domain:

As I mentioned, I am going to use Windows server and Windows desktop edition. My domain controller will be running Windows Server 2025 and my desktop will use Windows 10 Pro. For the installation of the Domain Controller I elected to use the windows GUI. This was just the most intuitive process for me. 

Sources: 
https://www.virtualgyanis.com/post/step-by-step-how-to-install-and-configure-domain-controller-on-windows-server-2019
https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-#perform-a-staged-rodc-installation-using-the-graphical-user-interface
https://learn.microsoft.com/en-us/windows-server/administration/server-manager/add-remove-roles-features?tabs=gui






