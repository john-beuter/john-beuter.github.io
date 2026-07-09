---
title: What I Learned from My Time as a Pentester with Tenax Solutions
description: An overview of my first real pentesting job and the experience that I gained from it
author: John 
date: 2026-07-08 11:33:00 -0500
categories: [Security]
tags: [Internship]
pin: true
math: true
mermaid: true
image:
  path: assets/img/Tenax.png 
---
### Intro


Over the past ~five months, while completing my Master's degree, I had the opportunity to work part-time as a pentester at Tenax Solutions. During my time at Tenax, I led eight internal and external penetration tests. Throughout these engagements, I consistently challenged myself to learn new attack paths and create new tools.


Leading up to my internship with Tenax I had gained invaluable knowledge through both my education at Iowa State and through my internships with Wellmark Blue Cross and Blue Shield and the Advance Course in Engineering (ACE). This background was instrumental in helping me adapt to the challenges of my role with Tenax. In this blog post, I wanted to take a moment to reflect on some of the biggest lessons that I learned from my first real experiences as a pentester.


### Plan the Work and Work the Plan


The first takeaway I had was the importance of planning. From my first week on the job, I was given real clients and real deadlines. Learning how to plan my time effectively was huge. I am a great time manager, but the past five months have really challenged me as I balanced an internship, grad school, and life. I found that what really worked for me was using the first 10-15 minutes of every day to make a goal list. These goals were pretty basic, like "enumerate web endpoint at X IP", "Revise section X of report Y", or "Research X type of attack or tool", but they broke down larger goals into smaller, more manageable tasks. After I created my list, I chunked out my 8-hour shift and wrote in when I was going to work on each goal. At the end of each shift, I would use the last 10-15 minutes to go over my goal list and notes from the day and assess what I accomplished and how I was progressing my deadlines.


Within learning how to best plan my time, I had to learn how to pentest more efficiently. In a previous blog post, I mentioned the challenges I faced when enumerating a network with thousands of devices. On that engagement, I found myself getting hyper-focused on one or two particular devices. I would dive super deep on one device, realize I couldn't go any further, then restart the process on a new device. It was slow and meticulous. A great strategy for a small network, but not feasible for a larger client. As I grew as a pentester, I had to find a balance between diving all in and only skimming the surface.


### "Seek First to Understand, Then to Be Understood"
#### Technical Application of Quote


I chose the above quote from Stephen F. Covey because it encompasses the philosophy I apply to my professional relationships and also to my approach to engineering/pentesting overall. During my engagements, I realized pretty early on that when I went down these rabbit holes, I wasn't stopping to understand the bigger picture. I was looking for quick pwns, root access, or, in some cases, Domain Admin. While all of those things are great, it meant I was missing out on some of the more complex attack paths. My first Active Directory pentest really exposed this flaw in my approach. After gaining initial access to the domain, I became heavily focused on the BloodHound results I got from scanning as a compromised user. I figured there had been a group permission I wasn't seeing or a device I had access to, etc. It wasn't until I spoke with a coworker, talked through the environment, and learned about Active Directory Certificate Services (ADCS) that I stopped to think, "Huh, maybe a network with this many users would implement ADCS." It was from this moment of pausing to ask "How is this network being used?" and following up with questions like "What services would be important on this network?" that I narrowed my attack scope and understood what would be most important to attack.


#### Interpersonal Application of Quote:


One thing that surprised me about this role was how often I had the chance to work directly with clients as an intern. This was an element that I really loved about my job. I thought it was exciting to talk through my work and recommend changes to help organizations be more secure. Each organization I tested had a different company culture and varying levels of IT/security experience. This meant that one week I could be working on a report for a very tech-savvy organization and the next I could be working with a small business that has never had an IT person, let alone a pentest. These varying client types meant I had to understand each client's unique background before I could explain my findings or methodology. I had to get to know the company and the people. I often did this by reaching out and communicating with the point of contact throughout my engagement, so they could stay in the loop and I could gauge how they responded to different findings. Using these insights, along with information from account managers about the company culture, I would tailor my reports to best suit my intended audience.




### Concluding Remarks
Overall, this role taught me a lot and also highlighted how I can grow. Going forward, I will continue my learning, focusing primarily on Active Directory penetration testing, which I will document on this site. I am incredibly grateful to the team at Tenax for giving me both the freedom and the mentorship to become a better security professional. Having the chance to go hands-on with numerous real-world environments gave me invaluable experience. I can't wait to continue my journey in cybersecurity and expand my understanding of red teaming concepts.
