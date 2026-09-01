---
title: Creating a Lateral Movement Lab Excercise for an Intro Cyber Security Course
author: John
date: 2026-07-30 11:33:00 -0500
categories: [Security]
tags: [School, Pentesting]
pin: true
math: true
mermaid: true
image:
 path: assets/img/Minotar.jpg
---



During my time at Iowa State, I had the privilege of working as a Teaching Assistant (TA) for numerous courses in the Cyber Security Engineering program. Of the courses that I had the opportunity to TA for, my favorite course was CYBE 2310: Cyber Security Concepts and Tools. In this course, students learn the basics of penetration testing through numerous lab exercises. For my creative component, the final project to complete my Master's program, I elected to create a new lab for the course that focused on lateral movement. Throughout my cyber career up until this point, I had limited professional experience using lateral movement, but I knew how powerful the skill could be when working through a complex maze of a computer network. With this blog post, I am going to talk through the learning goals with this lab and dive into how I found my own way through the labyrinth of teaching lateral movement.




### Background and Goals


Within 2310, the course material largely follows the “Cyber Kill Chain” (CKC) developed by Lockheed Martin. The CKC describes the overall process attackers follow to compromise a digital system. The Kill Chain begins with reconnaissance and gaining access, then migrates towards privilege escalation, lateral movement, and finally exfiltration of gathered intelligence. Within the labs for 2310, students work through almost all elements of the CKC except lateral movement.


#### My main goals for the new lateral movement lab were as follows:


1) Demonstrate the use cases of SSH tunnels


2) Showcase how we can use SSH tunnels to route tool traffic into compromised networks


3) Simulate lateral movement into an internal subnet from an external device using SSH tunnels


By showing students basic lateral movement, with SSH tunneling, I hoped to provide a valuable learning experience that could prove useful as students graduate and move to working in industry.


### Lab Summary


In the course, students are given their own network of devices to attack. The environment that students work with is similar to a pentest where the testers set up their own malicious device on the company's internal network. I set up the lateral movement lab so that students are given reverse shell access to a device that has been moved behind a firewall and onto a separate subnet. Now that the device they are targeting is on a separate network, and they only have access through a reverse shell, the lab challenges students to think through and experiment with how they can route traffic from their host through their reverse shell and move laterally into the separate subnet, thus finding their own gold thread through the maze.


#### 1) Student has reverse shell into the subnet
![Alt text](/assets/img/rev_shel.png)  


In a previous lab, students were able to compromise a running process (status.sh) on a PHP server and create a persistent reverse shell. The lateral movement lab takes the compromised PHP server and moves it behind a firewall. Students use the persistent shell as their initial foothold to subnet behind the firewall.


#### 2) Student uses Proxychains to route tool traffic into the subnet
![Alt text](/assets/img/FullProxy.png)  


At this stage in the lab, students use their initial reverse shell access to the compromised network to build an SSH tunnel. The students create an initial reverse SSH connection back to the attacker VM on port 24680 (remote forward). On the compromised host, students create a SOCKS proxy (dynamic forward) available on port 12345. Students then configure Proxychains on the attacker VM to route traffic through 24680 and to the remote host which then routes traffic via the dynamic forward on port 12345. These tunnels, along with Proxychains, allow student relay network-based tool traffic into the compromised network as shown in the diagram below.


![Alt text](/assets/img/Screenshot_20260731_100955.png)  

From this initial tunnel network, students develop the basic structure to support lateral movement into the subnet. 

#### 3) Student identifies an Apache Tomcat server that they can compromise
![Alt text](/assets/img/Server_Discovery.png)


Once the lab walks students through how SSH tunnels with Proxychains can be used to get a tool like nmap to run within the subnet, students discover another device on the isolated subnet. The lab walks through the enumeration process to identify the version of Tomcat that the server is running. After doing some research on the Tomcat version, students should discover that the server is running a version that is vulnerable to a remote code execution deserialization exploit (CVE-2025-24813). 




##### 3.1) CVE-2025-24813 Overview


CVE-2025-24813 is a vulnerability found in Apache Tomcat servers running versions 11.0.0-M1 to 11.0.1, 10.1.0-M1 to 10.1.34, and 9.0.0.M1 to 9.0.98. If the Tomcat server is configured to allow for partial PUT requests (that's a big if), then an unauthenticated attacker can trigger remote code execution by sending a malicious session file via a partial PUT request that contains a serialized Java payload. Tomcat handles the partially uploaded file and stores it in a location that a user can access. The attacker can send a GET request to the endpoint that stores the session file and trigger the deserialization of the payload, leading to remote code execution. See diagram below.


![Alt text](/assets/img/full_attack.png)




### My Experience Building the Lab


The saying goes, "To teach is to learn twice," and I found that saying to be incredibly accurate. At each design decision point in the creation of this lab, I was challenged to not only deepen my own technical understanding of the concepts but also my ability to effectively communicate my understanding to an audience of beginners.


### Making a CVE Come to Life for a Beginner Audience


For the final part of the lab, where students exploit the Tomcat server, I wanted to show off an attack path that students haven't had a chance to work with before, but also be aware of the students' experience level. I wanted the exploit to be challenging, but make sense in the context of the course. CVE-2025-24813 was a relatively new vulnerability with well-documented proof of concept code that I thought would be the perfect cap to the lab. The exploit allows for remote code execution, allowing students to gain access to the remote device and the exploit itself had enough nuance that it didn't feel like a copy and paste of every other remote code execution performed in previous labs.


While implementing the CVE seemed like a pretty cut-and-dry process. There were numerous articles about how the attack worked, and I thought I would be able to easily implement and test it based on published research. I was wrong. The article provided contradictory information at times and were purposly vague in some of the more technical areas. From a technical perspective, I loved the challenge of reading reports and trying to reverse engineer an exploit based on what was released to the public. Through this process, I got to learn and go hands-on with serialization exploits using the tool YSOSERIAL and understand at the client-server level how this attack was working.


#### Recognizing and Improving my Own Understanding of Lateral Movement


As I started the process of creating the network that would eventually become the lateral movement lab, I quickly realized how limited my own understanding of lateral movement was. My initial idea for the lab was to just cover lateral movement with Chisel. As I sat down to create the first iteration of the lab using Chisel I realized I didn't understand tunneling with Chisel well enough to accurately teach it. This realization made me reflect on my own learning process. When I initially learned about Chisel, and lateral movement for that matter, I watched a couple of YouTube videos and read some write-ups about using Chisel and didn't really dive into the weeds on the subject as much as I gained a surface-level understanding.


It wasn't until after I did some more reading, got hands-on practice, and dove into the technical weeds of how tunneling with something like SSH works that I started to understand lateral movement. By first working through forward, reverse, and dynamic SSH tunnels and understanding the use cases of each one, I could then easily use that knowledge to help me understand how to pick up a tool like Chisel or Lignolo-ng and run with it. That process of starting with the foundational elements first is the goal of the 2310 course. We, TAs and teaching faculty, are not trying to make students experts in any one tool or portion of the CKC; we are trying to teach students the foundational concepts that they can then apply to any tool. So, as I was creating my lab, I realized that I really didn't want to focus on the fancy tools; I wanted to focus first on the fundamentals. This is how I ended up using SSH tunnels for lateral movement in the lab.


#### Wrapping Up


I am incredibly grateful for the opportunity to be a TA at Iowa State and to have a chance to shape the curriculum for the program that I care so much about. Being able to create this lab not only enhanced my understanding of lateral movement, but it also challenged me to a better teacher and studet of cyber security. I hope that in the future, as this lab is worked into the course, students will gain experience and confidence that they can directly apply in their future careers.
