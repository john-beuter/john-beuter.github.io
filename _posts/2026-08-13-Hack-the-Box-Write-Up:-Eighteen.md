---
title: Pwning HTB Challenge Eighteen 
author: John
date: 2026-08-10 11:33:00 -0500
categories: [Security]
tags: [Pentesting]
pin: true
math: true
mermaid: true
---


As a part of my learning journey as a Cyber Security Engineer, I bought myself a Hack the Box (HTB) labs subscription. Using HTB, I aim to practice my technical pentesting skills and my written/reporting skills by documenting how I was able to exploit a machine. This month, I worked on the lab "eighteen" which was an Active Directory based lab that allow me to experiment and learn more about a privledge escalation vector via delegated managed service accounts (dMSAs) called BadSuccessor.

  
  

### Recon and Background

  

With this lab, I was given a username and credentials to begin testing. For my enumeration I began with an nmap scan. When I am performing my intial enumeration I like to perform pretty standard scans. My favorite is ```nmap -sV -Pn HOST``` . I find that it gives me enough information to start digging; however, if I am not worried about stealth, I'll throw in a "-A" to perform more aggresive OS and service detection as well as running NSE scripts. This initial scan helps me find if there are any easy to pop vulnerabilities like eternal blue on the device.

  

![Alt text](/assets/img/recon_nmap.png)

  

From my scan, I saw that the host was running four services: Microsoft (MS) IIS httpd (80), MS SQL Server (1433), and MS HTTPAPI (5985). My attention was immeditill drawn to WinRM (port 5985). I tried connecting to the target using EvilWinRM and the provided credentials but had no luck. I repeated the process the process except this time I targeted the MS SQL service. Using the impacket module mssqlclient I was able to authenticate to the service as the provided user Kevin.

  

### Finding the User Flag

  

Using enumeration commands built into mssqlclient, I enumerated my position on the SQL database. I started by running enum_db to see what databases I had available. After identifying these databases, I enumerated the SQL users, and the users that I could impersonate. From these commands I discovered that the user Kevin had permissions that allows them to impersonate the appdev user.

  

![Alt text](/assets/img/impersonate_kevin.png)

  

After becoming the user appdev, wanted to access the financial app database. I re-ran the command USE financial_app to change into the app database. Once inside the database I wanted to view all of the schema that the database was comprised of. Within the schema, I saw that there was a field for users. I then ran SQL command to view all users shown below:

  

![Alt text](/assets/img/sql_commands.png)

  
  

In the output above, I discovered a user hash in pbkdf2 format. I tried running the hash through both John and Hashcat with different options set. Each time I was disappointed when the tools failed to crack the hash. I knew there was something missing. I was stuck, plain and simple. It was at this point that turned to the published walk through to help get me back on track. Getting stuck and working through the problem is part of the learning process, but it is also important to recognize when you need some help to get moving again. From the write-up I learned that the hash was encoded with base64 and required a script to properly decode it into a form that can be cracked. After I cracked password hash I wanted to authenticate, using the cracked password, as different users on the device. Before I could use the password, I had to identify what users existed on the device.

  

In windows you have Security Identifiers (SID) and Relative Identifiers (RID). SIDs correspond to objects in Active Directory. For domain level accounts, the SID is created by concatenating the SID of the domain with an RID value that represents the account. As an authenticated user, I can use tools (like netexec) to enumerate RID values and find users on the device.

  

![Alt text](/assets/img/users.png)

  
  

After finding a list of users, I then created a list of users that I was able to pass into a netexec winrm enumeration module. This module enumerated the users and tried their username + the password I found in the cracked hash. This led to me discovering that the user adam.scott what the account we had access to. After identifying the correct user, via netexec, I authenticated to the target. As the compromised user, I then restarted my enumeration and recon from this new foothold.

  
  

### LOST

  

After gaining access, I then enumerated the environment using powershell commands. During this enumeration process I wanted to determine what type of device I was on (domain controller? workstation? a server?) and what permissions my user had.

  

At this point in the lab, I got bogged down. I was manually running different powershell commands to try and best piece together a picture of the permissions that this user had. After getting nowhere, I took a break and realized I was overlooking a major tool in my toolbox: Bloodhound. Instead of comming up with my own queries and running each one, I could upload a static binary of sharphound, the bloodhound collector, and run it in the environment.

  

After I ran sharphound, I downloaded the JSON output and interpretted the results in Bloodhound. From the permissions of the user adam scott, I saw that I was a part of the IT group. This was the first rabit whole that I went down as I thought that I might be able to abuse this group membership. From this group I saw that I had PS Remote access to the domain controller. Upon furether reasoning I determined that these permissions would really just grant me the ability to remotely run powershell as my current user. Nothing new. Back to the drawing board.

  

![Alt text](/assets/img/Adam_Scott_Groups.png)

![Alt text](/assets/img/scott_perms.png)

  

Taking a step back from Bloodhound, I decided to upload the PowerView powershell script to my target. PowerView is basically a beefed up powershell module designed to do recon within a Windows domain. Using PowerView, I was able to reveal different ACLs (Access Control Lists) related to my current user by running the command "Get-DomainObjectAcl -Identity "adam.scott"". Think of an ACL as a list of who can access what resource and with what rights. From the ACLs for my user "adam.scott" I discovered that I had "CreateChild" permissions over the "Staff" organizational unit (OU).

  

![Alt text](/assets/img/createChild.png)

  

### Found? - The Resource Based Constrained Delegation Goose Chase

  

Let's rewind a bit here. I see that I have this permission to create childern on the OU "Staff". Ok, what does that mean I can do? Well that means I have the ability to create a new user or computer within the OU of "Staff". Cool. How can I abuse that permission? Well, what if I can give the child I created more permissions than my intial user (adam.scott) has? This would be the perfect avenue for privledge escalation!

  

To a more techinical audience, the attack path I was going for here was essentially a form of Resource Based Constraint Delegation or RBCD. RBCD relies on having a compromised account with "msDS-AllowedToActOnBehalfOfOtherIdentity" permissions (on our target i.e. domain controller) and a machine account quota (MQA) that would allow for the attacker to create another device. If both of these privledges exist, an attacker can create device and then overwrite the "msDS-AllowedToActOnBehalfOfOtherIdentity" attribute on the target so that it delegates to the attacker created device. After delegation is granted to the attacker device, the attacker can request service tickets as the target via S4U2Self (allows a service to obtain a ticket to iteself on behalf of user) and S4U2Proxy (allows service to request a ticket from another service). Using these two settings, an attacker can get a service ticket for the service running on the target machine which leads to initial access. While I thought somethign like RBCD was the right attack path, it didn't work out because I didn't have "msDS-AllowedToActOnBehalfOfOtherIdentity" permission on my target (the domain controller).

  

After this attack failed, I took a step back and did some Googling. From a quick searching using some of the information I found during my recon, I got a hit for an attack path I hadn't heard of: Bad Successor.

  
  
  

![Alt text](/assets/img/google_search.png)

  
  
  

### Actually Found: Hello BadSuccessor

  

After using powerview. I found that my user had access to the CreateChild permission. From this permission, I found that I could perform the exploit "BadSuccessor". BadSuccessor uses the delegated Managed Service Account (dMSA) setting found in Windows Server 2025.

  

dMSA is a feature within Windows Server 2025. Using this feature legacy accounts can migrate their permissions to a dMSA. A dMSA itself is a machine account whose authentication is linked to a device identity (object in AD). Microsoft designed dMSA's in this way so only designated machine identities can access the account. This small distinction means that unlike a service account who's kerberos ticket can be requested by any user, a dMSA has a limited list of identities that it can be accessed by. Long story short, an attacker can't the kerberoast the dMSA unless their account is linked to the dMSA. Well that all sounds great! But looking deeper at the migration process reveals some potential flaws that an attacker can leverage.


During the dMSA migration process, a superseded process can still authenticate to the Domain Controller (DC). The key distribution center (KDC) recognizes the legacy account and adds it to the list of principals that are allowed to retrieve the dMSA's password. This permission enables every account that was dependent on the previous legacy account to have the ability to authenticate to the new dMSA. 

Once the migration is complete, the dMSA has all of the permissions of the superseded account. If a client tries requests a ticket using the previous service account, the domain controller responds with an error indicating that the service account has been superseded by the dMSA (sends _KRB-ERROR_ will contain the _KERB-SUPERSEDED-BY-USER_ field) . The client then automatically retries using the new dMSA account instead, and if the account is listed in the _msDS-GroupMSAMembership_ attribute of dMSA then the client is granted the ticket for the dMSA. 


In this attack path, we simulate the migration process. Instead of directly trying to perform a dMSA migration on a pre-built object, which requires Domain Admin permissions, an attacker needs to create a dMSA object and overwrite the preceded account target to any account that we want (done by modifying the _msDS-ManagedAccountPrecededByLink_). Additionally we can modify the _msDS-DelegatedMSAState_ value on the dMSA that we created to show that the migration process was completed even though no migration occcured. This simulation of a migration is what allows us to trick the KDC into thinking our attacker dMSA supersceded a targeted domain controller. This means that the KDC will now grant anyone requesting a service ticket as the dMSA the same permissions the target user if they listed as member of the dMSA.

  

#### Attempt One: Manually Exploiting via Powershell Commands

  

Ok, let's circle back. Our compromised user, adam.scott, has CreateChild permissions in the OU staff. Using these permissions, we can create a dMSA. With our dMSA we will fake a migration process and set our target user to be a domain admin. The KDC will think that a successfull migration has taken place, allowing for us to request a dMSA ticket and be granted the permissions of our target user. So, let's try it out!

  

When I first attempted this exploit, I directly followed the white paper published by the researcher Yuval XXX at Akamai. In that paper the author runs a series of powershell commands to create the dMSA and manually write to the PrecededByLink and the DelegateMSAState. I started by first creating a test account called "first_dmsa". After creating this account I then verified the settings for the DelegateMSAState setting by running the powershell command ```Get-ADServiceAccount -Identity "first_dmsa" -Properties * `|` Format-List```. Which generated the output shown below:

  

![Alt text](/assets/img/manual_delegated.png)

  

From the output we can see that the delegation state of the account shows up as 0. From the research done by Akamai, the state being set to '0' is an unknown value that could indicate that the dMSA is disabled. An interesting thing that I noticed at this point was that the PrecededByLink field was missing from the properties of the dMSA. I manually tried to set the attribute, but received a permission denied error.

  

![Alt text](/assets/img/failed_pre.png)

  

This behavior didn't make sense to me. Maybe it was the fact that the migration process was in an unknown state (0) instead of a completed migration state that caused my permissions update to fail or maybe it was acutally a permissions issue for my user.

During my enumeration with PowerView, I saw that adam.scott had the "WriteDACL" permission over the OU "Staff". WriteDACL is the permission that should allow my user to modify the discretionary access control list (DACL) that is associated with a specific object. The DACL specifies which users or groups can access the object and with what permissions. I assumed then that if this permission is enabled, then my user can create an object within the Staff OU and modify the permissions as I see fit. There had to be something I wasn't seeing or something I wasn't understanding about how permissions worked for this user, but more importantly, I was forgetting how the migration process permissions worked. 

I a couple paragraphs back, when describing how the dMSA migration process worked, I stated "Instead of directly trying to perform a dMSA migration on a pre-built object, which requires Domain Admin permissions, an attacker needs to create a dMSA object and overwrite the preceded account target to any account that we want (done by modifying the _msDS-ManagedAccountPrecededByLink_)." Well, there is my problem. By first creating the object on the domain, then going back and trying to overwrite the permissions built into the object at it's creation, I was trying to use permissions only accessible as a Domain Admin. What I needed to do was set all of the permissions on my dMSA object then create it on the domain, not create it then try to reconfigure.  
  

#### Attempt Two: Automated Exploitation via SharpSucessor

After some research into how I could create my evil dMSA object, I came across the tool SharpSucessor by Logan Goins (insert link to blog).  Below is an excerpt from their code: 

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

Reading through the code, Logan is setting the attributes of the dMSA that we need (namely the PrecededByLink which links the dMSA to the Administrator account) and uses CommitChanges() to save the object on domain. Using this code I can create a vulnerable dMSA object and use it priv esc.

##### Running SharpSuccessor

After upload SharpSuccessor to the environment via evil-winrm, I ran the following command to create my dMSA object:

```
SharpSuccessor.exe add /impersonate:Administrator /path:"OU=Staff,DC=eighteen,DC=htb" /account:adam.scott /name:john_dmsa

```

![Alt text](/assets/img/obj_cre.png)

Once I had my dMSA object, I need to get a Kerberos ticket for that account. Since my user (adam.scott), should be listed in the msDS-GroupMSAMembership attribute of the dMSA, I should have the necessary permissions to retrieve  a ticket on behalf of the dMSA. Let's check this permission using Powershell and PowerView:


![Alt text](/assets/img/adam_msD.png)



Now that we have confirmed adam.scott is listed in the msDS-GroupMSAMembership attribute we can start the process of getting a ticket for the dMSA. Before we can start messing with tickets we first need to change our Powershell logon type to 9 which allows us to perform Kerberos ticket operation. To do this I am going to run a basic reverse shell using Invoke-RunAs. 

On target:
![Alt text](/assets/img/first_shell.png)

Receiving connection on attacker machine:
![Alt text](/assets/img/second_shell.png)


First things first, we need to request a ticket granting ticket (TGT). Once our user has a TGT, I can now request a TGS  (Ticket Granting Service) ticket which will allow me to access dMSA. To perform our ticket operations I am going to use the tool "Rubeus" which I uploaded to the environment as "Rubu.exe." 

Note: For a more in-depth breakdown of the Kerberos tickets and this process please see my blog "Attacking ADCS and Exploiting ESC 8."

To get our TGT
