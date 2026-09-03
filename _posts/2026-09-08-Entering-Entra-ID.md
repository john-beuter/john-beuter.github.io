---
title: Overview of Entra ID Components and Attack Concepts 
author: John
date: 2026-09-14 11:33:00 -0500
categories: [Blogs]
tags: [Entra, Active Directory]
pin: true
math: true
mermaid: true
image:
 path: assets/img/entra.png
---


-- I need to talk about what a tenant is --
Make sure consistent wording between entra and entra ID 

## A Preface From the Author

This blog post is going to be a first for me. Typically with these posts I like to break down technical projects that I have created or challenge problems that I have solved. With this "mini-post" I wanted to step back a little and instead focus on checking my own understanding of a new (to me) concept: Entra ID. 

In this post, I'm going to do a high level overview of Entra ID covering what it is, what components make it up, and how/why it's used. I am not claiming to be an Entra or Microsoft AD pro by any stretch of the means. I merely hope that by learning the basic of Entra ID, and explaining my learning in a blog, I'll gain a deeper understanding of Entra ID. 

I feel it is especially important to point out with this post, as with all others, I am open to feedback and critiques of my work. I am always learning! With that being said, I hope you enjoy! 


## Background

Entra ID is something I have been meaning to dive into for awhile now, but was always a little intimidated by. For too long I simply wrote of Entra ID off with "Isn't it just AD but in the cloud? Well, no, not really. 

The majority of my real world networking pentesting been in on-prem environments. The network has a domain controller. The domain controllers is using LDAP, NTLM, and Kerberos to manage and facilitate what users are accessing etc. That's great, but what if a service that user needs exists outside of the domain? What if the company has on-prem infrastructure and infrastructure as a service products like cloud computing? Kerberos isn't going to work for this usecase so what else can we do? Well, we need some way to carry over the identity and access management policies (IAM), which are the policies that define who can do what and when, into an off-prem environment. This is where we rely on something called federated authentication.  


## Entra ID Oversimplified

Let's introduce Entra ID through the broad lense of federation. Federation is a process that allows for the conveyance of identity and authentication information across a set of networked systems. Federated authentication is the process of conveying user information to authenticate to one identity provider and be granted access to numerous other platforms that are trusted by the provider. It's this idea that is at the root of Entra ID. 

In a nutshell, Entra ID is like getting passport. When you get a passport, you have to present some identifying documents to the government of your country to proove your identity then you are granted a passport. Think of this like authenticating via single sign on. Once you have prooved your identity and the government has had a chance to verify it, you are granted a passport. With this passport you can travel to any country in the world as long as they trust your nations government (accept passports from that government). 

### Technical Breakdown

It is this bridge, between the on-prem domain and the off-prem cloud, that sparks my interest as an attacker. If Entra ID can sync a domain user to the cloud for authentication other apps and services, could an attacker use this to move their attacks into the cloud? 




### Microsoft Entra Connect
Now that we have a high-level understanding of federation let's look at how Entra connects on-prem Active Directory to off-prem services. Within the Entra ID environment there exists Entra Connect. 



If an environment uses on-prem AD and syncs to Entra ID, done via an Entra Id connector, the on-prem AD is deemed as the source of Truth. Alternativly the relationship can be thought of as uni-directional in the sense that a user created within an on-pren environment can have their identity synced to Entra, but a user that only exists in Entra can not have their account synced down to domain. 




## Resources I Found Useful: 

Federation:
https://www.windows-active-directory.com/azure-ad-federation-basics.html
https://www.windows-active-directory.com/federation-strategies-using-entra.html

I need to rewatch the John Savill video. 