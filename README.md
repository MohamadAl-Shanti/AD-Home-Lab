# AD-Home-Lab
This lab simulates a small enterprise Active Directory environment where users, organizational units, and group policies are managed using PowerShell. This details the implementation of my Active Directory home lab, further details and screen captures can be found in the screenshots folder. I Deployed Windows 10 and Windows Server virtual machines on Oracle VirtualBox using ISO installation images.
<br>

## Virtual Machine Setup
You can install VirtualBox here: https://www.virtualbox.org/wiki/Downloads
Install Windows Server and Windows 10 iso files online lol. When VirtualBox is configured; open VirtualBox, select the new option, select your iso file and destination folder, name your machine, and select finish:

<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/vmsetup1.png" alt="User Creation" width="100%"></td>
    <td><img src="AD-Lab-Screenshots/vmsetup2.png" alt="Policy Definition" width="100%"></td>
  </tr>
</table>

## Easy Password Reset and SConfig screen Reference
The majority of this lab's setup will refer to numbers on the SConfig screen, and there are steps throughout that will require you to authenticate via an authorized/administrator account. Feel free to refer to these images to make your life easier.
> ![usercreation](AD-Lab-Screenshots/sconfig_screen.png)
> ![usercreation](AD-Lab-Screenshots/AdminList&PassChange.png) <br> Three commands; List authorized accounts, create secure string for new password, set administrator account with the new password.

## Active Directory Installation
The following command should be used to install Active Directory and its services.
> ![usercreation](AD-Lab-Screenshots/AD1_InstallADServices.png)

## Domain Controller Configuration
I configured the domain controller on Windows Server as follows: 
  * **_Update Setting: Manual_** - (SConfig - 5) Enter 'M' to set updates to manual. This will prevent Windows Server from updating and rebooting while you are halfway through your configuration.
  * **_Remote Desktop: Enabled_** - (SConfig - 7) Enter 'E' to enable remote desktop. Select the more secure option. Setting this is best practice for 
system administration. Enabling Remote Desktop allows you to log into the domain controller and perform administrative tasks from Windows client machines.
  * **_Domain name - lab.local_** <br>
  > ![usercreation](AD-Lab-Screenshots/AD_Domain_Naming_and_Updated_Packages.png) <br> This command is used to configure your domain as a forest. Your machine is no longer viewed as a standalone server but rather the domain controller in a hierarchy of servers.

  You can now select 1 on the SConfig screen, and join the created domain as follows:
  > ![usercreation](AD-Lab-Screenshots/domain_change.png)
  * **_Computer name - ITDomainCont_**
  Select 2 on SConfig and change the computer name as follows:
  > ![usercreation](AD-Lab-Screenshots/image.png)
  * **_Network Adapter Address & Preferred DNS Server_**
    Select 8 on SConfig to configure the network addresses. There should be one network adapter present in the Network Settings, type 1 to select this adapter.
  > ![usercreation](AD-Lab-Screenshots/network_settings1.png) 

The preferred DNS server will have been set to the loopback address during the installation of ADDSForest. This is what we want, as our domain controller acts as the DNS server for our entire network.

  > ![usercreation](AD-Lab-Screenshots/network_adapter_settings.png)

Select 1 on the Network Adapter Settings to set the Network adapter address.

  > ![usercreation](AD-Lab-Screenshots/network_adapter_settings_filled2.png)

First select Static IP address. Since this is the domain controller it is required that it be at a fixed IP address since it is a point of reference for the entire network. It is recommended that you use 172.16.0.1 for your static IP address. This is a private IPv4 address that is commonly used in internal LAN environments. You may leave the subnet mask and the default gateway blank. Leaving the subnet mask blank defaults to 255.255.255.0, defining the boundary of your network to 172.16.0.1

I have found that the new IP address often does not stick, and that it is better to manually change it via PowerShell. If this is the case for you, refer to this commands to update it:

  > ![usercreation](AD-Lab-Screenshots/ipmanual.png)

This change should now be reflected in the Network Settings (8 on SConfig)

  > ![usercreation](AD-Lab-Screenshots/network_confirmation.png)

## Organizational Unit Creation
Upon configuring the domain controller, I created an organizational unit to organize accounts:

    New-ADOrganizationalUnit 
    -Name "Users" 
    -Path "DC=lab,DC=local"

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

> ![usercreation](AD-Lab-Screenshots/pass%26usercreation.PNG)

A user account provisioned here can be used to log into a Windows Client device with the selected credentials:

> ![usercreation](AD-Lab-Screenshots/GPOUserLoginClient.png)

## Group Policy Implementation:
Group policies were created and linked to organizational units to control user behaviour.
Example: Restricting user access to the control panel:

1. Create the group policy:

       New-GPO 
       -Name "IT-Admins-Policy" 
       -Domain "lab.local"
   
> ![policycreation](AD-Lab-Screenshots/GroupPolicyCreation.png)
       
2. Define the group policy:
   
       Set-GPRegistryValue 
       -Name "IT-Admins-Policy" 
       -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" 
       -ValueName "NoControlPanel" 
       -Type DWord -Value 1

> ![policydefinition](AD-Lab-Screenshots/GroupPolicyDefinition.png)

    3. Link the policy to the desired OU:
       New-GPLink 
       -Name "IT-Admins-Policy" 
       -Target "OU=IT,DC=lab,DC=local"
       
> ![policylinktoOU](AD-Lab-Screenshots/GroupPolicyLinkToOU.png)

    4. Adding a generic user to this OU enforces the policy on their account:
         Move-ADObject 
         -Identity "CN=LabUser4,CN=Users,DC=lab,DC=local" 
         -TargetPath "OU=IT,DC=lab,DC=local"

> ![policylinktoOU](AD-Lab-Screenshots/MoveUserOU.PNG)

When a user is placed inside an OU, any group policies associated to it are automatically applied to the user account. 

<table style="width:100%">
  <tr>
    <td><img src="AD-Lab-Screenshots/PolicyActive1.png" alt="User Creation" width="100%"></td>
    <td><img src="AD-Lab-Screenshots/PolicyActive2.png" alt="Policy Definition" width="100%"></td>
  </tr>
</table>

## Password Reset Example
A common help desk task in an Active Directory environment is resetting a user's password. The following is an example of this process:

    $securePass = Read-Host "Enter a secure password:" -AsSecureString
    Set-ADAccountPassword 
    -Identity "LabUser1" 
    -Reset 
    -NewPassword $securePass
    

