#  Implementing Active Directory (On-Premises) in Azure<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />






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
- Ensure Connection between Client and Domain Controller
- Install/Setup Active Directory and Admin Creation
- Create client users using PowerShell

<h2>Setup Resources in Azure</h2>

<p>
<img src="https://i.imgur.com/qU6Tddb.png" height="50%" width="50%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/1vDEENw.png" height="50%" width="50%" alt="Disk Sanitization Steps"/>
</p>

<p>

- Create the Domain Controller VM (Windows Server 2022) named “DC-1”. Take note of the Resource Group and Virtual Network (Vnet) that get created at this time
- Set Domain Controller’s NIC Private IP address to be static DC-1 > Networking > NIC > IP Configurations
- Create the Client VM (Windows 10) named “Client-1”. Use the same Resource Group and Vnet that was previously created
- Ensure that both VMs are in the same Vnet (you can check the topology with Network Watcher)
</p>
<br />
<h2>Ensure Connectivity between the client and Domain Controller</h2>
<p>
<img src="https://i.imgur.com/Mf3lGTW.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>

- Login to the Domain Controller and enable ICMPv4 in on the local windows Firewall
- Start Menu > Windows Defender Firewall with Advanced Secruity programme > Inbound Rules > Sort by Porotocol >
- Enable ICMPv4 rules

<img src="https://i.imgur.com/gpk21pk.png" height="50%" width="50%" alt="Disk Sanitization Steps"/>

- Login to Client-1 with Remote Desktop and ping DC-1’s private IP address with ping -t (perpetual ping) to verify connectivity
</p>
<br />

<h2>Install Active Directory</h2>

<img src="https://i.imgur.com/ZHQxtlh.png" height="70%" width="70%" alt="Disk Sanitization Steps"/>

- Login to DC-1 and install Active Directory Domain Services
- Install Active Directory Domain Services:
  - In the Server Manager, Select "Add Roles and Features"
  - Continue- Select Next, Next, Next,
  - Select "Active Directory Domain Services"
  - "Add Features"; "Next"; "Next"; "Next"; "Install"; "Close"

</p>
<h2>Setup Active Directory</h2>
<p>
<img src="https://i.imgur.com/FMBsJKz.png" height="40%" width="40%" alt="Disk Sanitization Steps"/>  <img src="https://i.imgur.com/SAO1UbQ.png" height="50%" width="50%" alt="Disk Sanitization Steps"/>

- Click "notification" and select "Promote this server to a Domain Controller"
- Select: "Add a new forest" (mydomain.com or your choice)
- Choose a Password and make note of this
- Complete Installation ("Next"; "Next"; "Next"; "Next" and "Install")
- Allow the server to close, which will disconnect the Remote Desktop
<h2>Restart and then log back into DC-1 as user: mydomain.com\labuser</h2>
</p>


<h2>Create an Admin and Normal User Account in AD</h2>

<img src="https://i.imgur.com/zISVYyl.png" height="50%" width="50%" alt="Disk Sanitization Steps"/> <img src="https://i.imgur.com/KXLABdR.png" height="40%" width="40%" alt="Disk Sanitization Steps"/>


- In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES”
- Create a new OU named “_ADMINS”
- Create a new employee named “Jane Doe” (same password) inside "_ADMINS" with the username of “jane_doe”

<img src="https://i.imgur.com/ZbJhhO4.png" height="50%" width="50%" alt="Disk Sanitization Steps"/>

Add jane_admin to the “Domain Admins” Security Group
- Select the _ADMIN Jane Doe and right click to Select Properties
- Select "Member Of"
- Add Domain Users: "Domain"
- Select "Check Names" to open name options
- Select "Domain Admins" and click "Ok" and "Apply"
- Log out/close the Remote Desktop connection to DC-1
- Log back in as mydomain\jane_doe
- User jane_doe will be your Admin account from now on

<h2>Join Client-1 to your domain (mydomain.com)</h2>

<img src="https://i.imgur.com/Q3RRsrW.png" height="50%" width="50%" alt="Disk Sanitization Steps"/> <img src="https://i.imgur.com/ftdGrtr.png" height="50%" width="50%" alt="Disk Sanitization Steps"/> 



- From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address
- From the Azure Portal, restart Client-1
- Login to Client-1 (Remote Desktop) as the original local admin (labuser) and join it to the domain (computer will restart)
- Login to the Domain Controller (Remote Desktop) and verify Client-1 shows up in Active Directory Users and Computers (ADUC) inside the “Computers” container on the root of the domain
- Create a new OU named “_CLIENTS” and drag Client-1 into there

<h2>Setup Remote Desktop for non-administrative users on Client-1</h2>

<img src="https://i.imgur.com/EpETgR7.png" height="50%" width="50%" alt="Disk Sanitization Steps"/>

- Log into Client-1 as mydomain.com\jane_doe and open system properties
- Click “Remote Desktop”
- Allow “domain users” access to remote desktop
- You can now log into Client-1 as a normal, non-administrative user
- Normally you’d want to do this with Group Policy which allows you to change many systems at once


<h2>Create random additional users and attempt to login to client-1 with one of the users</h2>

<img src="https://i.imgur.com/ob3dtl0.png" height="60%" width="60%" alt="Disk Sanitization Steps"/>

- Login to DC-1 as jane_admin
- Open PowerShell_ise as an administrator
- Create a new File and paste the contents of this [script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1) into it
- Run the script and observe the accounts being created
- Open Active Directory and _EMPLOYEES to see the list of random users being added

<h2>Attempt to log into Client-1 with one of the newly made accounts</h2>

<img src="https://i.imgur.com/gGE4m5l.png" height="60%" width="60%" alt="Disk Sanitization Steps"/> <img src="https://i.imgur.com/xjAgmnm.png" height="30%" width="30%" alt="Disk Sanitization Steps"/>

- Choose a random name, take note of account info
- Log off of Client-1
- Log into Client-1, using new account name to test access
- Congratulations on using Active Directory to create Admins/Users and grant permissions

