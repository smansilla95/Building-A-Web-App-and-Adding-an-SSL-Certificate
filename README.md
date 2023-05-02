<h1>Building A Web App and Adding an SSL Certificate</h1>


<h2>Description</h2>
Project consisted of setting up a domain through GoDaddy and then creating a web application using Azure Labs. Using the tools available within Azure, a resource group, virtual network, and network security group were set up, complete with accessibility rules. Additionally, three virtual machines were established, one being a jump box while the other two were used for web access. The jump box virtual machine was used to set up a docker container, provisioner, and Ansible playbook. A load balancer was also set up, configuring a static IP address and a backend pool connecting both web virtual machines to it. Using OpenSSL, self-signed certificates were created and then added to a Key Vault so that they could be bound to the web application.
<br />


<h2>Utilities and Environments Used</h2>

- <b>Microsoft Azure Cloud Services</b>
- <b>Terminal</b>
- <b>GoDaddy</b>

<h2>Program walk-through:</h2>

<p align="center">
Creating a Resource Group: <br/>

Enter the Azure portal and go to the home screen, then select the <b>+ Create</b> button on the <b>Resource group</b> section. Name the resource group and choose its region, which is supposed to have low latency and high availability. After reviewing and creating, finalize settings to create the group. Navigate to the resource group to check that all settings are correct. In this case, the resource group was named Red-Team and the selected region was Central US.

<p align="center"><br/>
<img src="https://i.imgur.com/kpxnIit.png" height="60%" width="60%" alt="Creating a Resource Group"/>
<p align="center"><br/>
<img src="https://i.imgur.com/58URVmI.png" height="60%" width="60%" alt="Creating a Resource Group"/><br/>

<p align="center">
Setting up the Virtual Network: <br/>

Search for "net" in the home screen and choose the search result for <b>Virtual Networks</b>. After clicking <b>Create virtual network</b> at the bottom of the page, fill in the network settings. These consist of this subscription, resource group created before, the name of the VNet (RedNet), and region, which is supposed to match the one used for the resource group. Additionally, a default network and subnet must be used to set up IP Addresses, default security settings, and tags. Under the review section, double check that everything lines up and finish creating the VNet.
 
<p align="center"><br/>
<img src="https://i.imgur.com/PD4SqbA.png" height="60%" width="60%" alt="Setting up the VNet"/>
<p align="center"><br/>
<img src="https://i.imgur.com/q1RatIw.png" height="60%" width="60%" alt="Setting up the VNet"/>
<p align="center"><br/>
<img src="https://i.imgur.com/DSLjiXP.png" height="60%" width="60%" alt="Setting up the VNet"/>
<p align="center"><br/>
<img src="https://i.imgur.com/selVXC8.png" height="60%" width="60%" alt="Setting up the VNet"/>

<p align="center">
Setting up the Network Security Group: <br/>

Look for "web" once more in the home page, and then select the option to create a network security group. Assign a name to the network security group, in this case RedTeam-SG, and add it to the resource group created before. Double-check that the security group is set up in the same region as the resource group and the virtual network that were set up in previous steps.
 
<p align="center"><br/>
<img src="https://i.imgur.com/byvq9oB.png" height="60%" width="60%" alt="Setting up the Network Security Group"/>
 
<p align="center">
Set up Inbound Rules for the Network Security Group: <br/>

After creating the security group, click on it to begin its configuration. Choose the Inbound security rules button and then click on the +Add button in order to add a new rule. The new rule should have these selections:
 
 - <b>Source: All, so that it can block all traffic sources.</b>
 - <b>Source Port Ranges: Wildcard(*), so that it can match all source ports.</b>
 - <b>Destination: Any, so that it will block all traffic associated with the security group.</b>
 - <b>Service: Custom.</b>
 - <b>Destination Port Ranges: Wildcard(*), so that it will block all destination ports.</b>
 - <b>Protocol: Any, so that all protocols used will be blocked.</b>
 - <b>Action: Deny, so that it will block any traffic that matches this rule.</b>
 - <b>Priority: The highest priority number (in this case 4,096).</b>
 - <b>Name: Default-Deny</b>
 - <b>Description: "Deny All Inbound Traffic"</b>
 
<p align="center"><br/>
<img src="https://i.imgur.com/ZoRz22c.png" height="60%" width="60%" alt="Set up Inbound Rules for the Network Security Group"/>
 
<p align="center"><br/>
<img src="https://i.imgur.com/qKz4foW.png" height="60%" width="60%" alt="Set up Inbound Rules for the Network Security Group"/>

<p align="center">
Setting Up a Jumpbox Virtual Machine: <br/>

Begin by creating an SSH key pair using Terminal. Run the command ssh-keygen and after verifying the output looks correct, run the command cat ~/.ssh/id_rsa.pub. Copy the output of that command, which is the SSH key string, and keep in your clipboard for later. After the key pair is generated, return to Azure and search for "virtual machines" in the search bar. Click on +Add to begin creating the VM. This VM requires these settings: 
 
<p align="left">
Basics: <br/>
 
 - <b>Resource Group: The same as the one used for Red Team, in this case Red-Team.</b>
 - <b>Virtual Machine Name: This VM will be called Jump Box Provisioner.</b>
 - <b>Region: Try to match the VM region to the one used for the resource and security groups (in this case Central US), but if it is unavailable, then create a new resource and security group that will match the one used to create the VM.</b>
 - <b>Availability Options: This VM will continue to use the default settings for availability.</b>
 - <b>Image: The Ubuntu Server 18.04 option will be used for this VM.</b>
 - <b>Size: This VM will use Standard-B1s, 1 CPU, and 1 RAM.</b>
 - <b>SSH: The authentication type will be an SSH Public Key, the username will be RedAdmin, and in the "SSH Public Key" field, paste the SSH key that was created before and copied to the clipboard.</b>
 - <b>Public Inbound Ports: Leave the default setting provided, since it will be overwritten by choosing the security group created earlier.</b>
 - <b>Select Inbound Ports: Like the previous selection, leave the default setting, as it will be overwritten by the security group rules.</b>

<p align="left">
Networking: <br/>
 
 - <b>Virtual Network: The VNet created for Red Team, in this case RedNet.</b>
 - <b>Subnet: The subnet that was created earlier, RedNetBase, will be used here.</b>
 - <b>Public IP: The default setting for this selection will be used.</b>
 - <b>NIC Network Security Group: The Advanced option will be used here, since it will allow for a specification of a custom security group.</b>
 - <b>Configure Network Security Group: RedTeamSG, which was created earlier, will be used here.</b>
 - <b>Accelerated Networking: Select "off" for this setting.</b>
 - <b>Load Balancing: Click "No" for this setting, which should be the default.</b>
 
<p align="center"><br/>
<img src="https://i.imgur.com/GmKGnAB.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>
 
<p align="center"><br/>
<img src="https://i.imgur.com/O8osUDz.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>

<p align="center">
Setting Up Two Web-Access Virtual Machines: <br/>

Two VMs will need to be created for web access in the network. After going to the virtual machines tab in Azure, select +Add in order to create the two web VMS. They need to have these settings:

<p align="left">
Basics: <br/>
 
 - <b>Resource Group: The same as the one used for Red Team, in this case Red-Team.</b>
 - <b>Virtual Machine Name: These VMS will be called Web-1 and Web-2.</b>
 - <b>Region: Try to match the VM region to the one used for the resource and security groups (in this case Central US), but if it is unavailable, then create a new resource and security group that will match the one used to create the two VMs.</b>
 - <b>Availability Options: Select "Availabilty Set" and then choose "Create New". It will be called RedTeamAS and have 2 Fault Domains and 5 Update Domains. After creating it for the first web VM, make sure to choose the same availability set for the second web VM.</b>
 - <b>Image: The Ubuntu Server 18.04 option will be used for these VMs.</b>
 - <b>Size: These VMs will use Standard-B1s, 1 CPU, and 2 RAM.</b>
 - <b>SSH: The authentication type will be an SSH Public Key, the username will be RedAdmin, and in the "SSH Public Key" field, paste the SSH key that was created before and copied to the clipboard when creating the Jumpbox VM.</b>
 - <b>Public Inbound Ports: Leave the default setting provided, since it will be overwritten by choosing the security group created earlier.</b>
 - <b>Select Inbound Ports: Like the previous selection, leave the default setting, as it will be overwritten by the security group rules.</b>
 
<p align="left">
Networking: <br/>
 
 - <b>Virtual Network: The VNet created for Red Team, in this case RedNet.</b>
 - <b>Subnet: The subnet that was created earlier, RedNetBase, will be used here.</b>
 - <b>Public IP: Choose None, as these web VMs should not have a public IP address.</b>
 - <b>NIC Network Security Group: The Advanced option will be used here, since it will allow for a specification of a custom security group.</b>
 - <b>Configure Network Security Group: RedTeamSG, which was created earlier, will be used here.</b>
 - <b>Accelerated Networking: Select "off" for this setting.</b>
 - <b>Load Balancing: Click "No" for this setting, which should be the default.</b>
 
<p align="center"><br/>
<img src="https://i.imgur.com/pg3xMZ1.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>

<p align="center"><br/>
<img src="https://i.imgur.com/8yYLVMI.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>
 
<p align="center"><br/>
<img src="https://i.imgur.com/HhOP0p8.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>

<p align="center">
Configuring Jumpbox Administration: <br/>

In order to SSH into the jumpbox virtual machine, a rule needs to created that allows connections only from your current IP address and no other IP addresses. This is to ensure the security of the web application while also being able to control it via SSH connection. This will be achieved by first going to whatismyip.org or any other IP address identifier and taking note of the IPv4 address of the network that is currently being used. Return to Azure to create a new security rule. This will be done by finding the security group that is listed under the resource group that was used throughout the activity. From there, choose Inbound Security rules and click on +Add to begin creating the rule, which will have these selections:
 
 - <b>Source: IP Addresses, which will open a new selection beneath it to identify the desired IP address.</b>
 - <b>Source IP Addresses/CIDR Ranges: Use the IP address that was identified earlier, in this case 209.58.129.97.</b>
 - <b>Source Port Ranges: Wildcard(*), so that it can match all source ports.</b>
 - <b>Destination: IP Addresses, so that the specific IP address of the Jumpbox VM can be used.</b>
 - <b>Destination IP Addresses/CIDR Ranges: The internal IP of the Jumpbox VM will be used here, which in this case is 10.0.0.4.</b>
 - <b>Service: SSH</b>
 - <b>Destination Port Ranges: 22</b>
 - <b>Protocol: TCP</b>
 - <b>Action: Allow</b>
 - <b>Priority: The highest priority number, but lower than the rule to deny all traffic.</b>
 - <b>Name: SSH</b>
 - <b>Description: "Allow SSH from specific IP address."</b>
 
<p align="center"><br/>
<img src="https://i.imgur.com/5jPJbir.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>
 
After the rule is set up, use the command line to SSH into the Jumpbox VM. This is done by typing #ssh sysadmin@10.0.0.4#. Once the SSH connection is achieved, check sudo permissions by running the command #sudo -l#. This is used to see that the admin has full permissions without needing a password.
 
<p align="center">
Setting Up the Docker Container: <br/>

 
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
