# AzureLocalDemo

- [Introduction](#intro)
	- [Know your Local IP information](#know-your-local-ip-information)
	- [Deploy AD/DNS](deploy-the-ad-dns-server)
	- [Set Uniform Time](before-you-start,-make-sure-your-host/ad-dns-server/vm-are-all-using-time.windows.com-for-clock-and-not-the-cmos)
	- Prepare Active directory
	- Make sure your host node meets these requirements
	- Now to run some code to create the VMs for us and register our node
	- On The Host node
	- Set these permissions first
	- Find your Azure subscription
	- Find your Microsoft Entra tenant
	- On Local Node
	- Now you are registered and ready to deploy
- Now you have an Azure Local Cluster...now what?
	- Add a logical network to your cluster
	- Setting up a Windows Server VM on Azure Local
	- Setting up RDP
	- Congratulations you should now be in your cluster and in the server...Now to set up WAC
	- Finally to add the cluster
- Is that all folks?

## Intro

Create and register Azure Local Node (VM version)

Hello and welcome to my "this has been done before but I struggled quite a bit so I want to try to help you with something that includes all the things I missed and steps that make sense to me" guide to deploying a virtual Azure Local edge node/cluster to try it out and get a feel for the setup/management etc.

I set out on this project because I knew it was possible and because, while I have been selling Azure Local (under several names now), I have not had any real experience setting up-deploying-managing a node/cluster hands on. This was a chance for me to do it on my own and stumble through and learn. Some people learn how I do...in which case I would say maybe use this as a guide path, but go learn on your own you'll love it. Some people just want to get up and running and good documentation with a clear path through the pages and pages Microsoft has available can be the thing that makes the experience a nice one rather than a daunting task.

Either way, this guide is my attempt to make sense for me, and even though I know there are several others out there, maybe help make sense of the process for you too. It is laid out as if you are new to all of this, because I am and I started from scratch here. I did have some networking knowledge coming into this I am trying to not make the assumption you do, and hopefully wont miss any notation you need. With that in mind, if you have something in place already feel free to skip on down to a later part if the process. Also, I make little to no claim to the code and large portions of the text in this guide. This is a compilation of steps from other guides (I have linked them throughout) and then me trying to lay it out in a full start to finish path with my notes around it.

I am starting this as if you have no AD/DNS DC set up. I am running this on two pieces of hardware, a small NAS  with an intel N100 - 16GB of memory - and 2.5GB NIC that I set up to run a VM with Windows server to be my AD/DNS server and a small micro PC with an AMD Ryzen 7 5825U - 64GB of memory - 2TB of NVME SSD and 2.5GB NIC. I am leaving names of manufactures out of this guide to be neutral but providing relevant information to help you make hardware decisions if you need to. All in my solution here cost less than $1000.00 The is a small switch, the NAS, the mini PC, and memory and hard drive upgrades for the NAS and Mini PC. My router is just from my internet provider. So nothing is super fancy here, and all of it can be used for more. I am running a small home lab, hosting a game server, running some containers for home automation. That is all to say you can use this hardware for multiple things if you need to buy new and need a reason other than "I want to try Azure Local".

I created this guide using markdown and put the code into blocks with notes. that means you can copy a codeblock into any text editor and edit it to your needs then just copy paste that into an elevated powershell instance. I have also tried to call out where you need to run what as that initially caused me some confusion. 

Anyway enough about me lets get into the project.


## Know your Local IP information

This is important for a few parts of the setup and deployment. I am outlining it here and have put detailed notes in the other places it comes up later in the guide.

This assumes you are on a home network. If you are in a corporate environment you can get this information from your network administrator. If you are on a home network you'll have an internal ip address range and your router typically does ip assignment, but you'll take that over here. This also means you need to keep track of your IP assignments. I would suggest keeping an excel table and filling it out and so you know what you have and haven't assigned. Your router will assign around what you have set, so just dodge the lower ip addresses and you should be fine. You can also use some auto discovery tools if you want, those can feel daunting if you aren't used to them, so if you are new, just a spreadsheet is fine for small networks. typically for a home network your ip address will be 192.168.xxx.xxx/24 your subnet mask 255.255.255.0 your default gateway 192.168.xxx.1 and your DNS server we set up earlier but as always checking is best.

- **Windows:**
    
    - Open the Start Menu and type "cmd" to open Command Prompt. 
    - Type `ipconfig /all` and press Enter. 
    - Your IP address will be listed under "IPv4 Address". 
    - Alternatively, go to Settings > Network & Internet > Wi-Fi (or Ethernet) > Advanced Settings > Network connection. 

When you run ipconfig you will see a list of outputs. There may be several adapters listed, but one will look something like this

	Ethernet adapter Ethernet 3:

	   Connection-specific DNS Suffix  . : lan
	   Link-local IPv6 Address . . . . . : fe80::f5e:82e4:ff2:b18c%28
	   IPv4 Address. . . . . . . . . . . : 192.168.95.23
	   Subnet Mask . . . . . . . . . . . : 255.255.255.0
	   Default Gateway . . . . . . . . . : 192.168.87.1

In this case 
your IP address range for you network will be
192.168.95.0 - 192.168.95.255
your Subnet Mask will be
255.255.255.0
your Default Gateway will be
192.168.95.1
and your router is the DNS server (or it is resolving to one somewhere)

This is important info for later

## Deploy the AD DNS server

On the NAS or some other node external to what will be your cluster start a VM running Windows server.

Lots of people have done this part, I am not sure I could do it better. I am going to link a video here **[How To Make A Windows Domain Controller In Hyper-V](https://www.youtube.com/watch?v=7-XL5r3pPZQ&t=488s)** Its as easy in a NAS as well as several have a VM host that makes the VM creation easy you just need a windows server trial .iso You can download one here **[Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)** Just make sure you install it with the GUI experience.

To create a Windows domain controller within Hyper-V, you'll first create a new virtual machine, install a Windows Server operating system, then install the [Active Directory Domain Services (AD DS)](https://www.google.com/search?sca_esv=b17bece7f59cc74c&cs=1&q=Active+Directory+Domain+Services+%28AD+DS%29&sa=X&ved=2ahUKEwiY76TB5JCMAxXu78kDHSE9Az8QxccNegQIAxAB&mstk=AUtExfBtiIUDVvg2DbFQjzGJ7SnhbXosFWYC9pIAStcki_wmMQ89_yMpavKU2zh5ve4hOM1hOIk5pAIbo-uw2ECIZnbrUuGu7S_Jb8u0HICHcztwv0PcFWZI2nZENM3gWSySVyIJ6aUVYnA8fT3ZoTYSmL9-GYZJoyXknvb4I8PnEWHdLys&csui=3) role and promote the server to a domain controller. 

Here's a more detailed breakdown:

1. Prepare the Hyper-V Environment:

- **Ensure Hyper-V is installed and configured:** Make sure the Hyper-V role is enabled on your host server. 
- **Verify Hardware Requirements:** Your host server needs an x64 processor, hardware-assisted virtualization (Intel VT or AMD-V), and Hardware Data Execution Protection (DEP). 
- **Create a Virtual Network Switch (if needed):** If you need external network connectivity, create a virtual network switch in Hyper-V and connect the virtual machine's network adapter to it. 

2. Create the Virtual Machine:

- **Open Hyper-V Manager:** Navigate to the Hyper-V Manager and select your server. 
- **Create a New Virtual Machine:** Click "New" and then "Virtual Machine". 
- **Follow the New Virtual Machine Wizard:**
    - **Name the VM:** Choose a descriptive name for your domain controller. 
    - **Select Generation:** Choose the appropriate generation (Generation 1 or Generation 2). 
    - **Assign Memory:** Allocate sufficient RAM to the VM. 
    - **Configure Virtual Hard Disk:** Create a virtual hard disk for the VM's operating system. 
    - **Network Adapter:** Connect the VM to a virtual network switch (if applicable). 
    - **Install Operating System:** Choose the Windows Server ISO image and install the operating system. 

3. Install and Configure [Windows Server](https://www.google.com/search?sca_esv=b17bece7f59cc74c&cs=1&q=Windows+Server&sa=X&ved=2ahUKEwiY76TB5JCMAxXu78kDHSE9Az8QxccNegQISxAC&mstk=AUtExfBtiIUDVvg2DbFQjzGJ7SnhbXosFWYC9pIAStcki_wmMQ89_yMpavKU2zh5ve4hOM1hOIk5pAIbo-uw2ECIZnbrUuGu7S_Jb8u0HICHcztwv0PcFWZI2nZENM3gWSySVyIJ6aUVYnA8fT3ZoTYSmL9-GYZJoyXknvb4I8PnEWHdLys&csui=3):

- **Install Windows Server:** Follow the on-screen instructions to install Windows Server. 
- **Join the Domain (if applicable):** If you are adding a new domain controller to an existing domain, join the VM to the domain. 
- **Configure Static IP Address (recommended):** Assign a static IP address to the VM's network adapter (this should be one within your ip address range from earlier). 
- **Open Server Manager:** Launch Server Manager on the new VM. 
- **Install Active Directory Domain Services (AD DS) Role:**
    - Click "Add roles and features". 
    - Select "Role-based or feature-based installation". 
    - Select the server. 
    - Choose "Active Directory Domain Services". 
    - Add necessary features (e.g., DNS). 
    - Install the role. 
- **Promote the Server to a Domain Controller:**
    - Click "Promote this server to a domain controller". 
    - Choose the appropriate option (e.g., "Add a new forest" for the first domain controller). 
    - Enter the domain name. 
    - Set the [Directory Services Restore Mode password](https://www.google.com/search?sca_esv=b17bece7f59cc74c&cs=1&q=Directory+Services+Restore+Mode+password&sa=X&ved=2ahUKEwiY76TB5JCMAxXu78kDHSE9Az8QxccNegQIeBAB&mstk=AUtExfBtiIUDVvg2DbFQjzGJ7SnhbXosFWYC9pIAStcki_wmMQ89_yMpavKU2zh5ve4hOM1hOIk5pAIbo-uw2ECIZnbrUuGu7S_Jb8u0HICHcztwv0PcFWZI2nZENM3gWSySVyIJ6aUVYnA8fT3ZoTYSmL9-GYZJoyXknvb4I8PnEWHdLys&csui=3). 
    - Complete the promotion wizard. 
- **Configure DNS (if needed):** If you are creating the first domain controller in a new forest, the DNS server role will be installed automatically. 

4. Post-Promotion Configuration:
    
- **Configure Time Synchronization:**
    
    Ensure that your domain controller is configured to synchronize time with a reliable time source.

## Before you start, make sure your host/AD-DNS server/VM are all using time.windows.com for clock and not the CMOS

you can use powershell to do this easily

``` Powershell

#Set time not from CMOS
w32tm /query /status
w32tm /config /manualpeerlist:"time.windows.com" /syncfromflags:manual /update
w32tm /query /status

```

Follow the guide, do the perquisites...feel free to read it a few times

[Deploy Azure Local](https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-introduction?view=azloc-24112)

[Deploy a virtual Azure Local System](https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-virtual?view=azloc-24113)

[Alternative guide](https://www.linkedin.com/pulse/building-my-first-azure-local-aka-stack-hci-cluster-home-pomato-6gdqf/)

## Prepare Active directory

Make sure your AD is ready. I missed this and thought because I had a forest and domain to join I was ready but I wasn't, and if you are new to this like me, you might not be too. 

Run this script  in powershell on your AD/DNS server

``` Powershell

Install-Module AsHciADArtifactsPreCreationTool -Repository PSGallery -Force

```

Now you need to create an OU and add a user (this can be new or a user if you already have one set just copy them into the new OU) make sure the password for the user meets Azure Local requirements. **Use a password that is at least 12 characters long and contains: a lowercase character, an uppercase character, a numeral, and a special character.**


``` Powershell

New-HciAdObjectsPreCreation -AzureStackLCMUserCredential (Get-Credential) -AsHciOUName "<OU name or distinguished name including the domain components>"

```

In case again like me you are new to this an example of a formal OU name would be "OU=ASHCI,DC=AzurelocalDC,DC=aztest" Assuming you have a domain that is AzurelocalDC.aztest 


## Make sure your host node meets these requirements


### Physical host requirements

The following are the minimum requirements to successfully deploy Azure Local.

Before you begin, make sure that:

- You have access to a physical host system that is running Hyper-V on Windows Server 2022, Windows 11, or later. This host is used to provision a virtual Azure Local deployment.
    
- You have enough capacity. More capacity is required for running actual workloads like virtual machines or containers.
    
- The physical hardware used for the virtual deployment meets the following requirements:
    
| Component             | Minimum                                                                                                                                                                                                                                                                                                                                        |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Processor             | Intel VT-x or AMD-V, with support for nested virtualization. For more information, see [Does My Processor Support Intel® virtualization technology?](https://www.intel.com/content/www/us/en/support/articles/000005486/processors.html).                                                                                                      |
| Memory                | The physical host must have a minimum of 32 GB RAM for single virtual node deployments. The virtual host VM should have at least 24 GB RAM.  <br>  <br>The physical host must have a minimum of 64 GB RAM for two virtual node deployments. Each virtual host VM should have at least 24 GB RAM for deployment and 32 GB for applying updates. |
| Host network adapters | A single network adapter.                                                                                                                                                                                                                                                                                                                      |
| Storage               | 1 TB Solid state drive (SSD).                                                                                                                                                                                                                                                                                                                  |

### Virtual host requirements

Before you begin, make sure that each virtual host system can dedicate the following resources to provision your virtualized Azure Local instance:

| Component                            | Requirement                                                                                |
| ------------------------------------ | ------------------------------------------------------------------------------------------ |
| Virtual machine (VM) type            | Secure Boot and Trusted Platform Module (TPM) enabled.                                     |
| vCPUs                                | Four cores.                                                                                |
| Memory                               | A minimum of 24 GB.                                                                        |
| Networking                           | At least two network adapters connected to internal network. MAC spoofing must be enabled. |
| Boot disk                            | One disk to install the Azure Stack HCI operating system from ISO. At least 200 GB         |
| Hard disks for Storage Spaces Direct | Four dynamic expanding disks. Maximum disk size is 1024 GB.                                |
| Data disks                           | At least 127 GB each. The size must be the same for each disk                              |
| Time synchronization in integration  | Disabled.                                                                                  |


## Now to run some code to create the VMs for us and register our node

Below are a few code blocks that can be copy pated into an elevated powershell session and used to create the VM, install the required software and get things set and registered in Azure. I have tried to add notes to help you understand what is happening where. Anywhere you see a variable it can be changed into something that works for you. I have several defaults set so that if you are working on a fresh machine things will install in some nice default locations. These come from Microsoft's example but work well. 

Locations for things
1. This is where the primary hard drive for your VM lives
	- C:\ProgramData\Microsoft\Windows\Virtual Hard Disks\
2. This is where your VM configuration data is store
	- C:\ProgramData\Microsoft\Windows\Hyper-v\
3. This is where we are keeping the data hard drives for the node
	- C:\vms\Node1\

Next we have the variables you can change if you would like to call them something other than what is outlined.

- Node(n) (if you are creating more than one node you have to adjust this or at the least the number for each node you create)
- NIC(n)
- ip address (xxx.xxx.xxx.xxx)

## On The Host node

Open Hyper-V manager

To enable Hyper-V and install the Hyper-V Manager, go to Control Panel > Programs > Programs and Features, click "Turn Windows features on or off," select "Hyper-V," and then click "OK". 

Here's a more detailed breakdown:

Steps to Enable Hyper-V and Install Hyper-V Manager:

1. **Open Control Panel:**
    - Click the Start button and type "Control Panel" in the search bar. 
    - Select "Control Panel" from the search results. 
2. **Navigate to Programs and Features:**
    - Click on "Programs". 
    - Click on "Programs and Features". 
3. **Turn Windows Features On or Off:**
    - Click on "Turn Windows features on or off" in the left-hand pane. 
4. **Select Hyper-V:**
    - Expand the "Hyper-V" folder. 
    - Check the box next to "Hyper-V". 
    - If you want to also install the Hyper-V Management Tools, select that option as well. 
5. **Confirm Changes:**
    - Click "OK". 
6. **Restart Computer:**
    - The system will prompt you to restart your computer to complete the installation. 
    - Restart your computer. 
7. **Access Hyper-V Manager:**
    - After restarting, you can access Hyper-V Manager by searching for it in the Start menu or through Administrative Tools.

Open an elevated powershell session on the host machine and run the following code block (after adjusting any of the variables we called out above)


``` Powershell

#In Powershell on host

#Create New VM

New-VHD -Path "C:\ProgramData\Microsoft\Windows\Virtual Hard Disks\Node1.vhdx" -SizeBytes 127GB
New-VM -Name Node1 -MemoryStartupBytes 24GB -VHDPath "C:\ProgramData\Microsoft\Windows\Virtual Hard Disks\Node1.vhdx" -Generation 2 -Path "C:\ProgramData\Microsoft\Windows\Hyper-V"
Set-VMMemory -VMName "Node1" -DynamicMemoryEnabled $false
Set-VM -VMName "Node1" -CheckpointType Disabled

#create NICs and attach to Virtual Switch in Hyper-V
#you only need 1 NIC for VM but if you want to create more to practice teaming or multiple 
#intents make more, just remember to adjust steps related to NICs throughout the rest of the deployment
Get-VMNetworkAdapter -VMName "Node1" | Remove-VMNetworkAdapter
Add-VmNetworkAdapter -VmName "Node1" -Name "NIC1"
Add-VmNetworkAdapter -VmName "Node1" -Name "NIC2"
Get-VmNetworkAdapter -VmName "Node1" |Connect-VmNetworkAdapter -SwitchName "azure_local_external_switch"
Get-VmNetworkAdapter -VmName "Node1" |Set-VmNetworkAdapter -MacAddressSpoofing On

#enable security on VM
$owner = Get-HgsGuardian UntrustedGuardian
$kp = New-HgsKeyProtector -Owner $owner -AllowUntrustedRoot
Set-VMKeyProtector -VMName "Node1" -KeyProtector $kp.RawData
Enable-VmTpm -VMName "Node1"

#set core count and create hard drives and mount them to the node
#importnat, remember because this is a VM and dynamic (thin) provisioned you can add
#way more space than you actually have. Be careful with this.
Set-VmProcessor -VMName "Node1" -Count 8

new-VHD -Path "C:\vms\Node1\s2d1.vhdx" -SizeBytes 1024GB
new-VHD -Path "C:\vms\Node1\s2d2.vhdx" -SizeBytes 1024GB
new-VHD -Path "C:\vms\Node1\s2d3.vhdx" -SizeBytes 1024GB
new-VHD -Path "C:\vms\Node1\s2d4.vhdx" -SizeBytes 1024GB
new-VHD -Path "C:\vms\Node1\s2d5.vhdx" -SizeBytes 1024GB
new-VHD -Path "C:\vms\Node1\s2d6.vhdx" -SizeBytes 1024GB
Add-VMHardDiskDrive -VMName "Node1" -Path "C:\vms\Node1\s2d1.vhdx"
Add-VMHardDiskDrive -VMName "Node1" -Path "C:\vms\Node1\s2d2.vhdx"
Add-VMHardDiskDrive -VMName "Node1" -Path "C:\vms\Node1\s2d3.vhdx"
Add-VMHardDiskDrive -VMName "Node1" -Path "C:\vms\Node1\s2d4.vhdx"
Get-VMIntegrationService -VMName "Node1" |Where-Object {$_.name -like "T*"}|Disable-VMIntegrationService
Set-VMProcessor -VMName "Node1" -ExposeVirtualizationExtensions $true

```

Now that is done in Hyper-V manager Before this adjust memory and add DVD with ISO in boot order in Hyper-V manager

``` Powershell

Start-VM "Node1"

```


Once started you may need to wait until after fail screen comes up and hit tab to
activate keyboard. Restart and install OS > rename node in Sconfig > do not join AD
you'll be asked for an admin password. **Make sure that the password meets complexity and length requirements. Use a password that is at least 12 characters long and contains: a lowercase character, an uppercase character, a numeral, and a special character.**


``` Powershell

#Set NICs and configre network settings on the NICS
Get-VMNetworkAdapter -VMName "Node1" -Name "NIC1"
Get-VMNetworkAdapter -VMName "Node1" -Name "NIC2"

#set MAC addresses properly
$Node1macNIC1 = Get-VMNetworkAdapter -VMName "Node1" -Name "NIC1"
$Node1macNIC1.MacAddress
$Node1finalmacNIC1=$Node1macNIC1.MacAddress|ForEach-Object{($_.Insert(2,"-").Insert(5,"-").Insert(8,"-").Insert(11,"-").Insert(14,"-"))-join " "}
$Node1finalmacNIC1
$Node1macNIC2 = Get-VMNetworkAdapter -VMName "Node1" -Name "NIC2"
$Node1macNIC2.MacAddress
$Node1finalmacNIC2=$Node1macNIC2.MacAddress|ForEach-Object{($_.Insert(2,"-").Insert(5,"-").Insert(8,"-").Insert(11,"-").Insert(14,"-"))-join " "}
$Node1finalmacNIC2

#reaname with the new MAC address and name all correct
$cred = get-credential
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {param($Node1finalmacNIC1) Get-NetAdapter -Physical | Where-Object {$_.MacAddress -eq $Node1finalmacNIC1} | Rename-NetAdapter -NewName "NIC1"} -ArgumentList $Node1finalmacNIC1
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {param($Node1finalmacNIC2) Get-NetAdapter -Physical | Where-Object {$_.MacAddress -eq $Node1finalmacNIC2} | Rename-NetAdapter -NewName "NIC2"} -ArgumentList $Node1finalmacNIC2

#Disable DHCP
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {Set-NetIPInterface -InterfaceAlias "NIC1" -Dhcp Disabled}
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {Set-NetIPInterface -InterfaceAlias "NIC2" -Dhcp Disabled}

#add the IP addresses you want, as an example if you are on a home network you'll have an internal ip
#address range your router typically does assignment, but you'll take that over here. This also means you
#need to keep track of your IP assignments. I would suggest keeping an excel table and filling it out
#and so you know what you have and haven't assigned. Your router will assign around what you have
#set, so just dodge the lower ip addresses and you should be fine. You can also use some auto discovery
#tools if you want, those can feel daunting if you aren't used to them, so if you are new, just a spreadsheet
#is fine for small networks.
#typically for a home network your ip address will be 192.168.xxx.xxx/24 your subnet mask 255.255.255.0
#your default gateway 192.168.xxx.1 and your DNS server we set up earlier
#but as always checking is best in any powershell terminal you can run ipconfig and that will spit out 
#the info for you
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {New-NetIPAddress -InterfaceAlias "NIC1" -IPAddress "XXX.XXX.XXX.XXX" -PrefixLength 24 -AddressFamily IPv4 -DefaultGateway "XXX.XXX.XXX.XXX"}
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {New-NetIPAddress -InterfaceAlias "NIC2" -IPAddress "XXX.XXX.XXX.XXX" -PrefixLength 24 -AddressFamily IPv4 -DefaultGateway "XXX.XXX.XXX.XXX"}

#set DNS server in the NICs
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {Set-DnsClientServerAddress -InterfaceAlias "NIC1" -ServerAddresses "XXX.XXX.XXX.XXX"}
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {Set-DnsClientServerAddress -InterfaceAlias "NIC2" -ServerAddresses "XXX.XXX.XXX.XXX"}

#enable Hyper-v for the VM
Invoke-Command -VMName "Node1" -Credential $cred -ScriptBlock {Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All }

#Now your node is ready to be regesterd as an Azure Local node is Azure

``` 



## Set these permissions first

these are available in this guide [# Register your machines and assign permissions for Azure Local deployment](https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-arc-register-server-permissions?view=azloc-24112&tabs=powershell) I used the portal for this, it's easier.

1. Register your subscription with the required resource providers (RPs). You can use either the [Azure portal](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider-1) or the [Azure PowerShell](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types#azure-powershell) to register. You need to be an owner or contributor on your subscription to register the following resource RPs:
    
    - _Microsoft.HybridCompute_
    - _Microsoft.GuestConfiguration_
    - _Microsoft.HybridConnectivity_
    - _Microsoft.AzureStackHCI_
    
     Note
    
    The assumption is that the person registering the Azure subscription with the resource providers is a different person than the one who is registering the Azure Local machines with Arc.
    
2. If you're registering the machines as Arc resources, make sure that you have the following permissions on the resource group where the machines were provisioned:
    
    - Azure Connected Machine Onboarding
    - Azure Connected Machine Resource Administrator
    
    To verify that you have these roles, follow these steps in the Azure portal:
    
    1. Go to the subscription that you use for the Azure Local deployment.
    2. Go to the resource group where you're planning to register the machines.
    3. In the left-pane, go to **Access Control (IAM)**.
    4. In the right-pane, go the **Role assignments**. Verify that you have the **Azure Connected Machine Onboarding** and **Azure Connected Machine Resource Administrator** roles assigned.
3. Check your Azure policies. Make sure that:
    
    - The Azure policies aren't blocking the installation of extensions.
    - The Azure policies aren't blocking the creation of certain resource types in a resource group.
    - The Azure policies aren't blocking the resource deployment in certain locations.

Now you can run the script below to register the machine. You need to find a few things in your azure portal.

## Find your Azure subscription

Follow these steps to retrieve the ID for a subscription in the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com/).
    
2. Under the Azure services heading, select **Subscriptions**. If you don't see **Subscriptions** here, use the search box to find it.
    
3. Find the subscription in the list, and note the **Subscription ID** shown in the second column. If no subscriptions appear, or you don't see the right one, you may need to [switch directories](https://learn.microsoft.com/en-us/azure/azure-portal/set-preferences#switch-and-manage-directories) to show the subscriptions from a different Microsoft Entra tenant.
    
4. To easily copy the **Subscription ID**, select the subscription name to display more details. Select the **Copy to clipboard** icon shown next to the **Subscription ID** in the **Essentials** section. You can paste this value into a text document or other location.

## Find your Microsoft Entra tenant

Follow these steps to retrieve the ID for a Microsoft Entra tenant in the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com/).
    
2. Confirm that you are signed into the tenant for which you want to retrieve the ID. If not, [switch directories](https://learn.microsoft.com/en-us/azure/azure-portal/set-preferences#switch-and-manage-directories) so that you're working in the right tenant.
    
3. Under the Azure services heading, select **Microsoft Entra ID**. If you don't see **Microsoft Entra ID** here, use the search box to find it.
    
4. Find the **Tenant ID** in the **Basic information** section of the **Overview** screen.
    
5. Copy the **Tenant ID** by selecting the **Copy to clipboard** icon shown next to it. You can paste this value into a text document or other location.

## Create a resource group in a region that supports Azure Local

To create a resource group in the [Azure portal](https://www.google.com/search?sca_esv=ea7911cd1c52328e&cs=1&q=Azure+portal&sa=X&ved=2ahUKEwj25v-F85CMAxUP4ckDHaRvMbgQxccNegQIAxAB&mstk=AUtExfBJ9AnUmF8cNByWLe3ZvafIRgO49_aYC8A3Dgz4_X4kTBXnoxxulgtmJYffpocE0QGKWPmrQzSl8Jm8fhILTCRp9LAMTS2jrdIKoUgXJc351EM3lsIurtmNNsvhjSj5KkK85u8Ji2MpXzH7SajDXi8PEVgYSM4iSyARu-bJjnxzAO0i0PaRCN2JO4hHgGIhUw1mEpNEzF0_vFZYf2irxgIuf4Vo4qA-AKMxMlciekLBpQ&csui=3), sign in, navigate to "Resource groups," click "+ Create," select your subscription, enter a name and region, then click "Review + create" and "Create". 

Here's a more detailed step-by-step guide:

1. Sign in to the Azure Portal:
	- Go to the Azure portal: https://portal.azure.com/ 

2. Navigate to Resource Groups:
	- Click on "Resource groups" in the left-hand menu. 

3. Create a New Resource Group:
	- Click the "+" button to create a new resource group. 

4. Fill in the Details:
	- **Subscription:** Select the Azure subscription where you want to create the resource group.
	- **Resource Group:** Enter a unique name for the resource group.
	- **Region:** Select the Azure region where you want to deploy resources within this resource group. 

5. Review and Create:
	- Click "Review + create" to review the details.
	- Click "Create" to create the resource group.

## **Azure regions: Azure Local is supported for the following regions:** (I have formatted them to work below)

- eastus
- westeurope
- australiaeast
- southeastasia
- indiacentral
- canadacentral
- japaneast
- southcentralus

## On Local Node

``` Powershell

#Set time not from CMOS
w32tm /query /status
w32tm /config /manualpeerlist:"time.windows.com" /syncfromflags:manual /update
w32tm /query /status

#Define the subscription where you want to register your machine as Arc device 
$Subscription = "your subscription id here"

#Define the resource group where you want to register your machine as Arc device
$RG = "your resource group name here"

#Define the region to use to register your server as Arc device
#Do not use spaces or capital letters when defining region
$Region = "the region you choose"

#Define the tenant you will use to register your machine as Arc device
$Tenant = "your tennant id here"

#Connect to your Azure account and Subscription
Connect-AzAccount -SubscriptionId $Subscription -TenantId $Tenant -DeviceCode

#Get the Access Token for the registration
$ARMtoken = (Get-AzAccessToken -WarningAction SilentlyContinue).Token

#Get the Account ID for the registration
$id = (Get-AzContext).Account.Id

#Invoke the registration script. Use a supported region.
Invoke-AzStackHciArcInitialization -SubscriptionID $Subscription -ResourceGroup $RG -TenantID $Tenant -Region $Region -Cloud "AzureCloud" -ArmAccessToken $ARMtoken -AccountID $id #(if you have proxy server remove this comment)-Proxy $ProxyServer

```


Now assign permissions for deployment

1. In [the Azure portal](https://portal.azure.com/), go to the subscription used to register the machines. In the left pane, select **Access control (IAM)**. In the right pane, select **+ Add** and from the dropdown list, select **Add role assignment**.
    
2. Go through the tabs and assign the following role permissions to the user who deploys the instance:
    
    - **Azure Stack HCI Administrator**
    - **Reader**
3. In the Azure portal, go to the resource group used to register the machines on your subscription. In the left pane, select **Access control (IAM)**. In the right pane, select **+ Add** and from the dropdown list, select **Add role assignment**.
    
4. Go through the tabs and assign the following permissions to the user who deploys the instance:
    
    - **Key Vault Data Access Administrator**: This permission is required to manage data plane permissions to the key vault used for deployment.
    - **Key Vault Secrets Officer**: This permission is required to read and write secrets in the key vault used for deployment.
    - **Key Vault Contributor**: This permission is required to create the key vault used for deployment.
    - **Storage Account Contributor**: This permission is required to create the storage account used for deployment.
5. In the right pane, go to **Role assignments**. Verify that the deployment user has all the configured roles.
    
6. In the Azure portal go to **Microsoft Entra Roles and Administrators** and assign the **Cloud Application Administrator** role permission at the Microsoft Entra tenant level.


## Now you are registered and ready to deploy

If you followed through this point you should be registered with a node that is available in the Azure > Azure ARC > Azure Local > Deploy Azure Local workflow. 

You can deploy using this guide and follow it all the way do the bottom to the enable RDP portion because you'll need that for the next couple of parts.

## [Deploy Azure Local using the Azure portal](https://learn.microsoft.com/en-us/azure/azure-local/deploy/deploy-via-portal?view=azloc-24112)

## Validated network topologies (its going to have a visual for you)

When you deploy Azure Local from Azure portal, the network configuration options vary depending on the number of machines and the type of storage connectivity. Azure portal guides you through the supported options for each configuration.

#### Supported network topologies

|Network topology|Azure portal|Resource Manager template|
|---|---|---|
|One machine - no switch for storage|By default|Supported|
|One machine - with network switch for storage|Not applicable|Supported|
|Two machines - no switch for storage|Supported|Supported|
|Two machines - with network switch for storage|Supported|Supported|
|Three machines - with network switch for storage|Supported|Supported|
|Three machines - with no network switch for storage|Not supported|Supported|
|Four to 16 machines - with no network switch for storage|Not supported|Not supported|
|Four to 16 machines - with network switch for storage|Supported|Supported|

The two storage network options are:

- **No switch for storage**. When you select this option, your Azure Local system uses crossover network cables directly connected to your network interfaces for storage communication. The current supported switchless deployments from the portal are one or two machines.
    
- **Network switch for storage**. When you select this option, your Azure Local system uses network switches connected to your network interfaces for storage communication. You can deploy up to 16 machines using this configuration.


## Now you have an Azure Local Cluster...now what?

Now you should have a cluster with a management name for the cluster. In my case I chose JSVS2D so I'll use this as the frame of reference for example but yours is whatever you named it in the deployment process. The registration process created a resource group for you, and the deployment process added your cluster to that resource group with all the required things it needs. You have to add one more thing to do other things in your cluster.  You need to create a logical network. This will allow the VMs and containers you run to interact withing the cluster and with the outside world. 


## Add a logical network to your cluster

To do this in your azure portal go to your resource group > then to your cluster > then to resources in the left blade > then to logical networks. From there you are going to create a logical network. You might need to enter the virtual switch name the first time (I did). If you didn't change it in the deployment it should be "ConvergedSwitch(compute_management)" This was what I chose. Yours might be different if you chose a different topology. You can check by running the following in powershell on the cluster in hyper-v manager and logging into the the machine exiting sconfig to the CLI and running the command below. Or connecting to the node in Azure and using password verification to remote into the CLI.

``` powershell

get-vmswitch

```

Once you have that you need to set the IP address assignment. 

If you have been following along, you need to set it for your whole IP address range. We figured this out at the very beginning of the guide.

So if your IP address is 192.168.95.201 you would add 192.168.95.1/24 as your space then indicate the pool you want the cluster to have so like 192.168.95.210 to 192.168.95.230 for example. in this case your default gateway would be 192.168.95.1 and your DNS server would be what you set earlier 192.168.95.xxx you can also set a VLAN ID here if you want, but it isn't necessarily required.

Now that we have that done you can begin to install VMs on your cluster. But first we want to walk through a VM to set up WAC on to manage it and give you a starting idea for how to do more.

## Setting up a Windows Server VM on Azure Local

First you want to go to your resource group and click into your cluster > resources > VM images > add VM image from that menu you can add an iso if you have one or download one from the marketplace. If you get one from the marketplace just make sure it isn't core, you want a GUI.

Once that is downloaded you want to go back to your cluster > resources > virtual machines > create a vm from there you can set it up like any VM Give it a username and password, chose to atomically join your domain and it will create it on your cluster.

## Setting up RDP

We are finally ready to set up RDP. For easy mode you want this to be on a PC in your domain. Download the RDP app from Microsoft app store, and sign in with your Azure subscription account. (if you are like me and this isn't a corporate account you'll need to look up the long form account user name. You can find this by going to your subscription > access control (IAM) > role assignments > your account (it will be in several places where you have given it permissions) > there you can see the long username...that is what you need to sign into RDP). 

Once signed in you can add pc's I used the IP address of the windows server we just created. Then add it and connect using your credentials.

## Congratulations you should now be in your cluster and in the server...Now to set up WAC

We are almost there From this point we just need to set up WAC. For this in the browser navigate to the **[evaluation center](https://www.microsoft.com/en-us/evalcenter/download-windows-admin-center)** We've been here before. Download Windows admin center and install it. Since it's running on a server it is going to assume the role of a gateway server for WAC which offers more flexibility and the ability to add your cluster. You need to be able to log in though. If you have never done it before it took me a minute to figure it out. The easy way is to open powershell and use this command

``` powershell

whoami /user

```

The output will look something like this

	USER INFORMATION

	User Name                     SID
	================== =============================================
	pcname\loggedinuser     S-1-5-21-3331493666-949227554-3045976163-1001

This will spit out a username and SID for you and that username is the username WAC is looking for. The password is just the password for the login to the server.

## Finally to add the cluster

Since WAC is a gateway and in your AD Domain adding server running on it is easy enough, you can click the add server button and then search active directory and then use a wildcard * to see all available servers but what about adding a cluster. When you created the cluster you created a resource that managed the cluster as a whole in Azure. This same name was applied to your AD OU and so now you have cluster in your AD by that name. In my case JSVS2D. So you click add cluster, search for yours, then use your AD credentials (the full ones blank@blank.blank) and password to log into that and you are off to the races, managing your cluster in WAC.

# Is that all folks?

That is where I'll end this guide. From this point on you know how to add VM images, start VMs, Manage in WAC, and you are ready to deploy containers and VMs and workloads to your cluster. From here the world is your oyster.

Now that you see how easy it is once everything is set up you can do it on the full blown version as well by acquiring validated hardware from a Microsoft approved OEM vendor [Azure Local Catalogue and sizer](https://azurelocalsolutions.azure.microsoft.com/).

I hope this guide helps, and wish you a good journey in your Azure Local adventures.
