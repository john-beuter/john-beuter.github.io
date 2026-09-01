---
title: Overview of Entra ID Components and Attack Concepts 
author: John
date: 2026-09-14 11:33:00 -0500
categories: [Security]
tags: [Entra]
pin: true
math: true
mermaid: true
image:
 path: assets/img/entra.png
---


This blog post is going to be a first for me. Typically with these posts I like to break down technical projects that I have create or challenge problems that I have solved. With this "mini-post" I wanted to step back a little and instead focus on checking my own understanding of new (to me) concept: Entra ID. In this post I'll be going over the basics of Entra ID. I am not claiming to be an Entra or Microsoft AD pro by any stretch of the means nor do I claim to be the leading source of knowledge on this topic. I merely hope that by learning the basic of Entra ID, and explaining my learning in a blog, that I will be better able to retain what I learned and help other beginners (like me) understand how Entra ID works. 

I feel it is especially important to point out with this post, as with all others, I am open to feedback and critiques of my work. I am always learning! 

## Jumping from on-prem to the cloud

We need to break down the components that not only make up Entra but that define why Entra exists. The majority of my real world experience has been define by working in on-prem environments. The network has a domain controller, users authenticate to it and receive the ability to access different resources, again, all within the network environment. These excahnges that gurantee a users access rely on protocols and services like NTLM, LDAP, and Kerberos. 

All of this is great, but what if the company has numerous other platforms that are off-prem that they want their users to authenticate to? Something like a cloud based service per say? Well On-prem domain based authentication isn't going to be the solution, we need something that can support federated authentication. 

Federation is the process of conveying an identity and access information accross mutlpile network systems (NIST). Federation is how we have things like single sign ons that allow us to sign into one account and have access to numerous sites. At it's core Entra ID is a large scale enabler of federated access.  -- 



## Entra ID: Something cool and completely different from Azure Active Directory

Entra ID is connective authentication tissue that allows a user to authenticate to multiple resources that exist within somethign called an Entra tenant. Think of the Tenant like the enivornment where Entra IDs are trusted. Entra ID don't solely exist for the connection from on-prem to the cloud, it can also be used as a stand alone single auth point. 

If an environment uses on-prem AD and syncs to Entra ID, done via an Entra Id connector, the on-prem AD is deemed as the source of Truth. Alternativly the relationship can be thought of as uni-directional in the sense that a user created within an on-pren environment can have their identity synced to Entra, but a user that only exists in Entra can not have their account synced down to domain. 

### Entra ID connector



## Resources I Found Useful: 
