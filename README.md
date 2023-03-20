<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in Azure</h1>
This tutorial/lab outlines the implementation of on-premises Active Directory within Azure Virtual Machines. If you haven't seen my prerequisite tutorials first, go ahead and check them first and come back here afterwards, as this lab builds on those ones.<br />

<h2>Prerequisites</h2>

- [Setup a Subscription and a Resource in Azure for Beginner Labs](https://github.com/jacksonmalms/setup-azure-sub-and-resource)
- [Create Virtual Machines Within Azure and Observe The Network Topology](https://github.com/jacksonmalms/create-azure-vm)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup Resources in Azure
- Ensure Connectivity between the client and Domain Controller
- Install Active Directory
- Create an Admin and Normal User Account in AD
- Join Client-1 to your domain
- Setup Remote Desktop for non-administrative users on Client-1
- Create a bunch of additional users and attempt to log into client-1 with one of the users
- Clean-up/delete resources or stop both VMs

<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/mBlYU0j.png"/>
</p>
<p>
First, create the resources, I just named the resource group AD-LAB, the virtual machine DC-1. Select Windows Server 2022 and set the size to be 2 CPUs, then create the VM.
</p>
<br />

<p>
<img src="https://i.imgur.com/iANg4bk.png"/>
</p>
<p>
Then create the Windows 10 VM (Client-1), make sure it's in the same resource group and region, and then select the size to use 2 CPUs as well. And finally make sure it is in the same vnet and subnet as DC-1. 
</p>
<br />

<p>
<img src="https://i.imgur.com/2On7GkF.png"/>
</p>
<p>
Now go into DC-1 and click into the NIC (network interface card) as shown above.
</p>
<br />

<p>
<img src="https://i.imgur.com/iBiPkNw.png"/>
</p>
<p>
Then go into IP configurations and click on DC-1's configuration.
</p>
<br />

<p>
<img src="https://i.imgur.com/5PGTLsH.png"/>
</p>
<p>
Now set the IP to static and click save in the upper left.
</p>
<br />

<p>
<img src="https://i.imgur.com/sCtm2u6.jpg"/>
</p>
<p>
Now login to Client-1 and ping DC-1's private IP with perpetual ping with ping -t, notice the request is being timed out, keep this open for now.
</p>
<br />

<p>
<img src="https://i.imgur.com/d0h4V0Q.jpg"/>
</p>
<p>
Now log into DC-1 and open Windows Defender Firewall with Advanced Security.
</p>
<br />

<p>
<img src="https://i.imgur.com/wJhlDGy.png"/>
</p>
<p>
Now go into inbound rules and enable all ICMPv4 traffic, this is the protocol that the ping command uses.
</p>
<br />

<p>
<img src="https://i.imgur.com/Ig6FsqZ.jpg"/>
</p>
<p>
Go back into Client-1 and notice that we are now getting replies back from DC-1 with the ping -t command. (FYI if you hit Ctrl + C in command prompt it will stop whatever command that is currently being executed)
</p>
<br />

<p>
<img src="https://i.imgur.com/Mb91ghL.png"/>
</p>
<p>
Now we will install AD inside of DC-1, so go back into DC-1 and go into server manager -> Add roles and features -> Server Roles -> select Active Directory Domain Services -> Add Features -> and then click next until you can hit install.
</p>
<br />

<p>
<img src="https://i.imgur.com/wVqaKbn.png"/>
</p>
<p>
Now we need to promote DC-1 as a domain controller, so click on the flag in the top right and click "Promote this server to a domain controller".
</p>
<br />

<p>
<img src="https://i.imgur.com/XFj5Yrf.png"/>
</p>
<p>
Now add a new forest and name it (I just named this one "mydomain.com"), hit next.
</p>
<br />

<p>
<img src="https://i.imgur.com/We2cZjC.png"/>
</p>
<p>
Then set the DSRM password, and then just hit nest until you can hit install. It will restart the VM after installing.
</p>
<br />

<p>
<img src="https://i.imgur.com/5nfxWVn.png"/>
</p>
<p>
When you connect back into DC-1, you will need to use a different account and use mydomain.com\user (make sure it has a backslash and not a forward slash), use what ever username that you set when you created DC-1.
</p>
<br />

<p>
<img src="https://i.imgur.com/RfGRNWT.png"/>
</p>
<p>
Now go to Active Directory Users and Computers (ADUC).
</p>
<br />

<p>
<img src="https://i.imgur.com/Gl98T2W.png"/>
</p>
<p>
Now create two new Organizational Units (OU) one called “_EMPLOYEES” and the other “_ADMINS”. Do that by going to mydomain and right clicking in the empty space.
</p>
<br />

<p>
<img src="https://i.imgur.com/HDPxqTB.png"/>
</p>
<p>
Now create a new admin called "jane_admin" and uncheck "User must change password at next logon".
</p>
<br />

<p>
<img src="https://i.imgur.com/GF10K7x.png"/>
</p>
<p>
Now add Jane to Domain Admin, to do that right click on jane -> Properties -> Member Of -> Add -> type "Domain" in the object box -> Check Names -> Domain Admins -> hit OK and then apply.
  
After you add Jane to the domain admins security group, logout and logon to DC-1 as jane_admin.
</p>
<br />

<p>
<img src="https://i.imgur.com/8OmbFP9.png"/>
</p>
<p>
Now set Client-1's DNS server to the private IP of DC-1, in the case of mine it was 10.0.0.4.
  
To do that, on the Azure portal go into Client-1's networking and click on the NIC, go into DNS servers and add the private IP of DC-1 into the DNS server list, and then hit save. After that you will need to restart Client-1 from the Azure portal and logon to it as the original user set for that VM.
</p>
<br />

<p>
<img src="https://i.imgur.com/I8Zp3vV.png"/>
</p>
<p>
After you log back into Client-1, go to Settings -> About -> Rename this PC -> change domain -> set it as mydomain.com -> and then use mydomain.com\jane_admin to make the changes. The VM will now restart.
  
After it restarts logon to Client-1 with MYDOMIAN\jane_admin instead (mydomain.com\jane_admin won't work since by default Client-1 will consider MYDOMAIN as the actual name of the domain. 
</p>
<br />

<p>
<img src="https://i.imgur.com/WmJykFi.png"/>
</p>
<p>
Now in Client-1 go to remote desktop settings and allow all domain users to connect to Client-1. To do this click on Select users that can remotely access this PC -> Add -> type "domain users" in the object box -> Check names -> hit OK.
</p>
<br />

<p>
<img src="https://i.imgur.com/3UaY6jj.png"/>
</p>
<p>
Then in DC-1 go back to Active Directory Users and Computers and ensure Client-1 is listed.
</p>
<br />

<p>
<img src="https://i.imgur.com/1SQwwYX.png"/>
</p>
<p>
In DC-1 open up powershell ise as admin.
</p>
<br />

<p>
<img src="https://i.imgur.com/iGLgdDi.png"/>
</p>
<p>
Now copy the powershell script linked below, hit the new script icon, paste in the script, and then hit the green run button. This script will create 10,000 users with randomly generated names inside the domain, you can change that number if you would like to, it doesn't matter.

- [Script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)

Now you can chose any of those usernames to sign into Client-1, the password will be "Password1" as that is what powershell script set them as and you can play around with the script and change it if you would like to. 
  
Congrats! That is the end of this lab, hopefully you had fun and learned a thing or two, because I sure did!
</p>
<br />
