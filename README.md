# AD-Home-Lab
This lab simulates a small enterprise Active Directory environment where users, organizational units, and group policies are managed using PowerShell. This repository documents the implementation of my Active Directory home lab, further details and screen captures can be found in the screenshots folder. I deployed Windows 10 and Windows Server virtual machines on Oracle VirtualBox using ISO installation images.
<br>

#### The Entra ID integration/extension is still at work and can be found at the bottom of the document for those interested

## Virtual Box Setup
You can install VirtualBox here: https://www.virtualbox.org/wiki/Downloads <br>
The installation is relatively straightforward, just select the exe for your operating system and follow the setup instructions provided.

## Virtual Machine Setup
This home lab will make use of two virtual machines. 
1. **Windows Server 2022** - This is where you will configure your domain controller. The domain controller in an Active Directory environment is the central authority responsible for all administrative tasks such as user account provisioning and policy implementation. The ISO installation image for Windows Server can be found at https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022
2. **Windows 10** - This machine will be used to log into provisioned user accounts. Its primary purpose in this lab will be to verify the connection of a client to the lab domain and that implemented policies are active and functional. The ISO installation image for Windows 10 can be found at https://www.microsoft.com/en-ca/software-download/windows10. Select _Download Now_ under _Create Windows 10 Installation Media_, then follow the installation steps and to create and ISO which will be used to set up our virtual machine.

When VirtualBox and both ISO images are installed you can set up your virtual machines. Select _New_, name your VM, navigate to and select the ISO image, and choose a destination folder for your VM, _ex/ \labvms_. Ensure that Unattended install is **unselected** as it is often cause for bugs when setting up your OS in the VM. When both VMs are created, boot them up, verify that they work, and complete the OS installation for each. When you are installing Windows 10 in VirtualBox **MAKE SURE YOU SELECT WINDOWS 10 PRO** otherwise you will not be able to join any domains.

Starting with the Windows 10 machine. Right click the machine in VirtualBox, navigate to Settings > Network. Under _Attached to_ select _Internal Network_. Name the internal network what you like and leave the rest of the settings as they are. Perform the same with the Windows Server VM, then select Adapter 2 and under _Attached to_ select _NAT_, all other settings can remain default. 
<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/vmsetup1.png" alt="User Creation" width="100%"></td>
    <td><img src="AD-Lab-Screenshots/vmsetup2.png" alt="Policy Definition" width="100%"></td>
  </tr>
</table>

## Useful Reference Images
The majority of this lab's setup will refer to numbers on the SConfig screen. Feel free to refer to this image if you ever get confused when you see an instruction like "Select number 7".

<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/sconfig_screen.png" alt="User Creation" width="100%"></td>
  </tr>
</table>

There are steps throughout the lab that will require you to authenticate via an authorized/administrator account. Particularly during domain controller configuration. Should this occur, select 15 in sconfig and refer to the following image and commmands to reset your administrator/authorized account's password.

<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/AdminList&PassChange.png" alt="User Creation" width="100%"></td>
  </tr>
</table>

Locate your administrator account.
```powershell
Get-ADGroupMember -Identity "Domain Admins" | Select-Object Name, SamAccountName
```
Ensure the following two commands are run in the same session to preserve the $password variable. Set your password to something you will remember.

```powershell
$password = Read-Host "Enter new password" AsSecureString
```
```powershell
Set-ADAccountPassword -Identity "Administrator" -NewPassword $password -Reset
```
## Active Directory Installation
The following command should be used to install Active Directory.
<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/AD1_InstallADServices.png" alt="User Creation" width="100%"></td>
  </tr>
</table>

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

## Domain Controller Configuration
I configured the domain controller on Windows Server as follows: 
  * **_Update Setting: Manual_** - (SConfig - 5) Enter 'M' to set updates to manual. This will prevent Windows Server from updating and rebooting while you are halfway through your configuration.
  * **_Remote Desktop: Enabled_** - (SConfig - 7) Enter 'E' to enable remote desktop. Select the more secure option. Setting this is best practice for 
system administration. Enabling Remote Desktop allows you to log into the domain controller and perform administrative tasks from Windows client machines.
  * **_Domain name - lab.local_** <br>

<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/AD_Domain_Naming_and_Updated_Packages.png" alt="User Creation" width="100%"></td>
  </tr>
</table>

```powershell
Install-ADDSForest -DomainName "lab.local" -ForestMode WinThreshold -DomainMode WinThreshold -InstallDNS:$true -Force
```

This command is used to configure your domain as a forest with lab.local as the root domain. Your machine is no longer viewed as a standalone server, it is promoted to be the domain controller in a hierarchy of servers. 

This foundation allows for the future expansion into child domains such as dev.lab.local or marketing.lab.local to create a multi-domain tree. WinThreshold is not strictly necessary. It sets the domain's functional level to Windows Server 2016 ensuring that no feature incompatible with Windows Server 2016 or newer will be used. InstallDNS:$true installs the DNS server role, allowing you to configure your domain controller as a DNS server.



You can now select 1 on the SConfig screen, and join the created domain as follows:
<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/domain_change.png" alt="User Creation" width="100%"></td>
</tr>
</table>
  
**_Computer name - ITDomainCont_**
Select 2 on SConfig and change the computer name, you can choose any fitting name, for example *ITDomainCont*

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/image.png" alt="User Creation" width="100%"></td>
</tr>
</table>
  
**_Network Adapter Address & Preferred DNS Server_**
Select 8 on SConfig to configure the network addresses. There should be one network adapter present in the Network Settings, type 1 to select this adapter.
<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/network_settings1.png" alt="User Creation" width="100%"></td>
</tr>
</table>

The preferred DNS server will have been set to the loopback address during the installation of ADDSForest. This is what we want, as our domain controller acts as the DNS server for our entire network.

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/network_adapter_settings.png" alt="User Creation" width="100%"></td>
</tr>
</table>

Select 1 on the Network Adapter Settings to set the Network adapter address.


<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/network_adapter_settings_filled2.png" alt="User Creation" width="100%"></td>
</tr>
</table>

First select Static IP address. Since this is the domain controller it is required that it be at a fixed IP address since it is a point of reference for the entire network. It is recommended that you use 172.16.0.1 for your static IP address. This is a private IPv4 address that is commonly used in internal LAN environments. You may leave the subnet mask and the default gateway blank. Leaving the subnet mask blank defaults to 255.255.255.0, defining the boundary of your network to 172.16.0.X

I have found that the new IP address often does not stick, and that it is better to manually change it via PowerShell. If this is the case for you, you can change the static IP manually via PowerShell with the following commands:

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/ipmanual.png" alt="User Creation" width="100%"></td>
</tr>
</table>


First get the correct Interface Alias, in my case this is *Ethernet* 
```powershell
Get-NetAdapter
```

This command will update to the desired IP:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 172.16.0.1 -PrefixLength 24
```

This change should now be reflected in the Network Settings (8 on SConfig)

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/network_confirmation.png" alt="User Creation" width="100%"></td>
</tr>
</table>

## Connecting your Client Machine to your Domain Controller

Before we attemotNow that we have configured our domain controller, we need to join the domain we created with our Windows client. Log into the Windows 10 machine, navigate to _Network Status_ > _Change adapter options_ > _Ethernet_ > _Properties_. Select _Internet Protocol Version 4 (TCP/IPv4)_ and properties. Then configure _IP Address, Subnet Mask, and Preferred DNS Server_ as follows: 

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/settingDNSforclient.png" alt="User Creation" width="100%"></td>
</tr>
</table>
  
  > IP address can be set to any IP in your range other than that of your DNS server, another option might be 172.16.0.102.
  
  > The preferred DNS server must point directly to the DC since it is the DNS server for the entire network.

Search for This PC and select properties. Scroll down and select _Rename this PC (advanced)_. Select _change_ and set the domain to the name of the domain you previously created via the domain controller (ex/ lab.local). With this you should have successfully joined the correct domain. You will be prompted to restart the computer to apply these changes.

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/joindomainfinally.png" alt="User Creation" width="100%"></td>
</tr>
</table>

We will further validate that our joining of the domain was successful in the following section.

## User Account Creation
User accounts were created in PowerShell. Passwords were stored as secure strings.

    $securePass = Read-Host "Enter a secure password:" -AsSecureString

Example user creation command:

    New-ADUser 
    -Name "LabUser1" 
    -GivenName "Lab" 
    -Surname "User1" 
    -SamAccountName "LabUser1" 
    -UserPrincipalName "LabUser1@lab.local" 
    -Enabled $true 
    -AccountPassword $securePass 
    -Path "OU=Users,DC=lab,DC=local"

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/pass%26usercreation.PNG" alt="User Creation" width="100%"></td>
</tr>
</table>

Passwords can be created and assigned to a user in the single user creation command.

    New-ADUser
    -Name "John Smith"
    -GivenName "John"
    -Surname "Smith"
    -SamAccountName "jsmith"
    -UserPrincipalName "jsmith@lab.local"
    -Enabled $true
    -AccountPassword (Read-Host -AsSecureString "Enter a password")
    -Path "OU=Engineering,OU=Departments,DC=lab,DC=local"

If your join of your lab domain was successful, then you can log into your client machine with the credentials of any account provisioned.

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/GPOUserLoginClient.png" alt="User Creation" width="100%"></td>
</tr>
</table>

## Bulk User Creation

In situations where wants to create a large number of user accounts, say during employee onboarding, manually creating every user account with AD would be very time consuming. In AD you can make use of bulk user creation by writing account info to CSV format. 

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/writingtocsv1.png" alt="User Creation" width="100%"></td>
</tr>
</table>


While I tend to prefer writing to strings and then copying those strings to csv files, it is much easier to create a csv and then just write to it in notepad. Use the following command with your csv's path to open it up in notepad.

```powershell
notepad replace-with-your-file-path
```

The format of the csv's lines is set in the first line. Each comma separated item within a line can be treated as a separate parameter. One line in users.csv for example contains a user's first name, last name, OU, etc, all as separate referenceable parameters. We can put these to use with a simple **for loop** to create multiple users in one command. We import the contents of the csv file into a string and then run our loop.


<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/BulkUserCreation.png" alt="User Creation" width="100%"></td>
</tr>
</table>


To make bulk user creation even more efficient, we can use a bit of **scripting**. We will write the commands we need to create a set of users to a PowerShell Script file (ps1). This will allow us to automate the bulk creation process. Anytime we want to create a number of new user accounts via a csv we just need to execute this script. 

I would again recommend typing up your script in notepad to verify that your syntax is correct, as it is easy to mess up when trying to write commands to a file through PowerShell, as you can see by the error message behind the notepad window lol.


<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/editInNotepad.png" alt="User Creation" width="100%"></td>
</tr>
</table>


As you can see the above script allows you to enter the path to your users csv, then a secure default password. It then creates a user for every line of the csv, setting their password to the default, and requiring a password change on logon. Once you have verified that your syntax is correct go ahead and create a fresh user's csv with new user for whom you want to create the account. Run the script and enter your user csv path and your choice of a default password.

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/successfulbulkscript.png" alt="User Creation" width="100%"></td>
</tr>
</table>

The users I created in newusers.csv were *user1, user2, & user3*. Should the script have been successful, they should appear listed upon running this command.

```powershell
Get-ADUser -Filter *
```

<table style="width:100%">
<tr>
  <td><img src="AD-Lab-Screenshots/newusersfrombulk.png" alt="User Creation" width="100%"></td>
</tr>
</table>

## Organizational Unit (OU) Creation
Upon configuring the domain controller, I created an OU to hold all user accounts for users in the domain:

    New-ADOrganizationalUnit 
    -Name "Users" 
    -Path "DC=lab,DC=local"

You can place OUs in other OUs by prepending *OU=* to the -Path string. For example you might want to place an *Engineering* OU in a *Departments* OU.

```powershell
New-ADOrganizationalUnit
-Name "Departments"
-Path "DC=lab,DC=local"
```

```powershell
New-ADOrganizationalUnit
-Name "Engineering"
-Path "OU=Departments,DC=lab,DC=local"
```

## Security Grouping and Group Policy Implementation:
Group Policy Objects (GPOs) can be linked to OUs to control user behaviour.

**Security Groups** are used to assign permissions and apply policies to specific collections of users across OUs.

1. Create the GPO:

       New-GPO 
       -Name "IT-Admins-Policy" 
       -Domain "lab.local"
   
> ![policycreation](AD-Lab-Screenshots/GroupPolicyCreation.png)
       
2. Define the GPO:
   
       Set-GPRegistryValue 
       -Name "IT-Admins-Policy" 
       -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" 
       -ValueName "NoControlPanel" 
       -Type DWord -Value 1

> ![policydefinition](AD-Lab-Screenshots/GroupPolicyDefinition.png)

3. Link the GPO to the desired OU:

       New-GPLink 
       -Name "IT-Admins-Policy" 
       -Target "OU=IT,DC=lab,DC=local"
       
> ![policylinktoOU](AD-Lab-Screenshots/GroupPolicyLinkToOU.png)

4. Adding a generic user to this OU enforces the policy on their account:
         
       Move-ADObject 
       -Identity "CN=LabUser4,CN=Users,DC=lab,DC=local" 
       -TargetPath "OU=IT,DC=lab,DC=local"

> ![policylinktoOU](AD-Lab-Screenshots/MoveUserOU.PNG)

When a user is placed inside an OU, any GPOs associated to it are automatically applied to the user account. 

<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/PolicyActive1.png" alt="User Creation" width="100%"></td>
    <td><img src="AD-Lab-Screenshots/PolicyActive2.png" alt="Policy Definition" width="100%"></td>
  </tr>
</table>

Applying GPOs to OUs is useful for enforcing general restrictions at the department level. However, these restrictions can sometimes be too broad.
For example, an Accounting OU might restrict users from changing their wallpaper. While appropriate for most employees, this restriction may not be suitable for senior staff.Instead of removing the policy entirely, Security Groups can be used to refine its scope.

OUs and Security Groups serve different purposes. A user belongs to one OU at a given level of the hierarchy, but can belong to multiple Security Groups.
This allows policies to be applied broadly via OUs, then refined using Security Groups based on role or responsibilities.

Together, OUs and Security Groups allow administrators to adjust access control without modifying the underlying policies applied to the organization.

The effectiveness of Security Groups can be demonstrated with three accounts:
- Accounting Intern
- Engineering Intern
- Engineering Executive

Assume a GPO is applied to the Engineering OU that prevents users from changing their wallpaper. By default, this affects both the intern and the executive.

> ![policylinktoOU](AD-Lab-Screenshots/securitygroupaccs.png)
> Account Creation: The GPO will first be applied to engineering, then will be applied only to an interns security group to demonstrate the power of security grouping.

> ![policylinktoOU](AD-Lab-Screenshots/createenggpo.png)
> ![policylinktoOU](AD-Lab-Screenshots/setenggpo.png)
> ![policylinktoOU](AD-Lab-Screenshots/linkenggpo.png)
> As before: Create the GPO, Define it, then link it to the OU

<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/englogin.png" alt="User Creation" width="100%"></td>
    <td><img src="AD-Lab-Screenshots/wallpaper_restriction.png" alt="Policy Definition" width="100%"></td>
  </tr>
</table>

The policy applies to Engineers regardless of position. If we only want this GPO to apply to interns organization wide, we can create an _Interns Security Group_ and shift the policy from all Authenticated users in engineering, to all interns in our domain. Applying GPOs to OUs is still valuable for some foundational restrictions, but security groups are very good since a user can be in several Security Groups.

1. Let's start by removing the policy's link to the Engineering OU and applying it globally. At this point the GPO applies to every user in the domain.

    ```powershell
    Remove-GPLink -Name "Engineering-No-Wallpaper" -Target "OU=Engineering,DC=lab,DC=local"
    ```

    ```powershell
    New-GPLink -Name "Engineering-No-Wallpaper" -Target "DC=lab,DC=local"
    ```
    
1. We will create the Interns Security Group and add the engineering and accounting interns to it.

   ```powershell
   New-ADGroup -Name "Interns" -GroupCategory Security -GroupScope Global -Path "DC=lab,DC=local"
   ```

   ```powershell
   Add-ADGroupMember -Identity "Interns" -Members "engintern", "accintern"
   ```

2. We can mute the policy for all authenticated users such that it no longer affects any accounts domain-wide.

   ```powershell
   Set-GPPermissions -Name "Engineering-No-Wallpaper" -TargetName "Authenticated Users" -TargetType User -PermissionLevel None -Replace 
   ```
   
3. We then apply the policy to the Interns security group, and voila, both interns regardless of OU are affected, and non-intern members of their OUs are not.

   ```powershell
   Set-GPPermissions -Name "Engineering-No-Wallpaper" -TargetName "Interns" -TargetType Group -PermissionLevel GpoApply
   ```

> ![policylinktoOU](AD-Lab-Screenshots/accinterngpo.png)
> Policy active for accounting intern
> ![policylinktoOU](AD-Lab-Screenshots/enginterngpo.png)
> Policy active for engineering intern
> ![policylinktoOU](AD-Lab-Screenshots/engexecgpo.png)
> Policy filtered out and inactive for non interns

## Common Administrative tasks

### Password Reset
A common help desk task in an Active Directory environment is resetting a user's password. The following is an example of this process:

    $securePass = Read-Host "Enter a secure password:" -AsSecureString
    Set-ADAccountPassword 
    -Identity "LabUser1" 
    -Reset 
    -NewPassword $securePass

### Password Rotation
In enterprise environments, it is best practice to enforce password resets after a set period of time (ex/30 days, 90 days, 1 year). In AD an admin can enforce a password reset upon a user's next login.

> ![policylinktoOU](AD-Lab-Screenshots/passwordRotation0.png)

    Set-ADUser
      -Identity "jsmith"
      -ChangePasswordAtLogon $true

While it is possible for an admin to manually reset a password from a DC, this is not best practice since it would involve the admin being aware of the user's password. As such the previous command can be preceded by a standard password reset command in the case that a user forgets their password. This would allow the user a temporary password with which to authenticate before changing their password at login, since the user must know their current password to go through the reset.

```powershell
Set-ADAccountPassword
-Identity "jsmith"
-Reset
-NewPassword (Read-Host -AsSecureString "Enter a temporary password")
```

```powershell
Set-ADUser
-Identity "jsmith"
-ChangePasswordAtLogon $true
```

Now the user will be prompted to change their account the next time they want to log in.

<table style="width:100%">
  <tr>
    <td width="50%"><img src="AD-Lab-Screenshots/passwordRotation1.png" alt="User Creation"></td>
    <td width="50%"><img src="AD-Lab-Screenshots/passwordRotation2.png" alt="Policy Definition"></td>
  </tr>
  <tr>
    <td width="50%"><img src="AD-Lab-Screenshots/passwordRotation3.png" alt="User Creation"></td>
    <td width="50%"><img src="AD-Lab-Screenshots/passwordRotation4.png" alt="Policy Definition"></td>
  </tr>
</table>

## Entra ID Integration

### Tenant Creation and Azure AD Syncing

1. Created a new tenant in Microsoft Entra Admin Center.
   - Domain Name: adlabextension
   - Resource Group: ad-lab-rg
   - Make sure it is internal
2. Installed Entra Connect to sync on prem AD environment with Entra
   - You will have the option to download cloud sync or connect sync. I chose connect sync to manage the lab on prem.
