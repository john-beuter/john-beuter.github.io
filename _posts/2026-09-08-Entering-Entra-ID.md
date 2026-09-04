---
title: Overview of Entra ID 
author: John
date: 2026-09-11 11:33:00 -0500
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
Make a diagram to show the token exchange process
Add sources
Where does ADFS exist in this process??? 

## A Preface From the Author

This blog post is going to be a first for me. Typically with these posts I like to break down technical projects that I have created or challenge problems that I have solved. With this "mini-post" I wanted to step back a little and instead focus on checking my own understanding of a new (to me) concept: Entra ID. 

In this post, I'm going to do a high level overview of Entra ID covering what it is, what components make it up, and how/why it's used. I am not claiming to be an Entra ID or Microsoft AD pro by any stretch of the means. I merely hope that by learning the basic of Entra ID, and explaining my learning in a blog, I'll gain a deeper understanding of Entra ID. 


## Background

Entra ID is something I have been meaning to dive into for awhile now, but was always a little intimidated by. For too long I simply wrote of Entra ID off with "Isn't it just AD but in the cloud? Well, no, not really. 

The majority of my real world networking pentesting been in on-prem environments. The network has a domain controller. The domain controllers is using LDAP, NTLM, and Kerberos to manage and facilitate what users are accessing etc. That's great, but what if a service that user needs exists outside of the domain? What if the company has on-prem infrastructure and infrastructure as a service products like cloud computing? Kerberos isn't going to work for this usecase so what else can we do? Well, we need some way to carry over the identity and access management policies (IAM), which are the policies that define who can do what and when, into an off-prem environment. Here is where we need a cloud based identity provider and federated authentication. Here is where we need to implement Entra ID. 


## The Goal of Entra ID

So what is an identity provider, what is federated authentication, and how do these components make up Entra ID? 

Let's start by looking at Entra ID through the lense of federation. Federation is a process that allows for the conveyance of identity and authentication information across a set of networked systems. Federated authentication is the process of conveying this information about a users identity from an identity provider to then be granted permissions to access another system/resource. 

Ok, let's piece it together: 
Entra ID is a cloud based IdP that supports federated authentication that allows for things like single sign on. 


### How does the Single Sign On (SSO) process work? 

Now that we defined the goal of Entra ID, let's get into the how of how this SSO process works. A couple paragraphs back, I talked about how on-prem AD uses Kerberos, Ldap, and NTLM convey IAM information and that Entra ID can't rely on those same protocols to convey IAM information. Entra ID instead use different protocols that focus on token based authentication. So when our example user goes to authenticate to a SSO, they present their username and password to the IdP (Entra ID) and if the information provided by the user, matches a user profile within Entra ID, then the user is granted a security token. Using this token, our user can navigate to server or resource and authenticate by presenting their token. 

#### Tokens

When a server receives a token it will look at different pieces of information called claims that define the tokens attributes. These attributes can detail the tokens subject, which is information the identifies who the token was issued for, the issued at and expiration time of a token, and the audience of a token. The audience field is the tokens way of specifiying who is the intended target for this token. If a server receives a token and the audience field is blank or does not specify the server, the receiving server discard the token entirly. If the token contains the correct claims information and the receiving server has an established trust with the IdP, then the user can authenticate. If no trust relationship exists, the receiving server has no way of understanding whether the token is valid or not. 

Pause. At this point in my learning adventure with Entra ID, I had a lot of questions. Namely, is the token the user granted the same for all resources? Having it be one token that granted a user access to all resources didn't seem like a great idea, security wise since if an attacker compromised one token they would have access to everything. So how does Entra ID create an new token for every service without forcing re-authentication each time? Well during authentication through a web based SSO, Entra ID can set cookie on the users browser to show the session that was created during authentication. Now, if the user requests a resource, the corresponding server sends a request to the IdP which can use the cookie to authenticate the user and assign a new token. Pretty neat! 


### Identities 

Let's look at how Entra ID creates users. One common way is to link users that exist in an on-prem instance of AD to Entra ID. The process of linking these users is possible via the Entra ID connect tool that runs within an on-prem environment. Now Entra ID connect has some important quirk, namely it syncs unidirectionally with any account that is created within the on-prem being synced into Entra ID; however, if an account only exists in Entra ID, it cannot be synced down to the on-prem AD. Additionally, when a user is created on Entra ID from a on-prem domain user, equivalent objects are created. This means that as user, I mantain the same permissions I had locally in the cloud. Interesting. 

Copying from on-prem to Entra ID isn't the only way that businesses can create users in Entra ID. Users can be directly imported into Entra ID from other services or users can be added from other Entra ID instances. In the later example, B2B collaboration, Entra ID can be used to connect two businesses with shared services. Through this connection, companies can create things like guest accounts that would allow a member of different organization to have access to your companies Entra ID environment. It was at this point in my Entra ID learning that I start to think about what attack pathes are possible in Entra ID.


### Closing Thoughts: Moving from understanding to weaponizing

My understanding of Entra ID is still a work in progress. But by creating this post I noticed that I started to understand Entra ID more and throught that understanding I have been able to think more critically about attacking Entra ID. At this point in my Entra ID journey I have so many questions like, If I compromised a domain locally, accomplished domain admin, how could I abuse the Entra ID connect process?" or "How could domain compromise impact the security of Entra ID tokens?" or "Could I forge tokens as other users even without DA permissions?" These questions are great for helping me define where my understading of Entra ID and attacking Entra ID can expand and grow. In the future, I hope to get hands on experience with Entra ID in a controlled lab environment and start experimenting to discover the answer to some of these questions. Until then, I hope you have enjoyed this post and learned something new.

I feel it is especially important to point out with this post, as with all others, I am open to feedback and critiques of my work. I know within this overview I may have missed some details or components that make up Entra ID. Please educate me. I am always looking to learn and grow. 

## Resources: 

Federation:
https://www.windows-active-directory.com/azure-ad-federation-basics.html
https://www.windows-active-directory.com/federation-strategies-using-entra.html
