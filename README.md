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

Search for "net" in the home screen and choose the search result for <b>Virtual Networks</b>. After clicking <b>Create virtual network</b> at the bottom of the page, fill in the network settings. These consist of this subscription, resource group created before, the name of the VNet, and region, which is supposed to match the one used for the resource group. Additionally, a default network and subnet must be used to set up IP Addresses, default security settings, and tags. Under the review section, double check that everything lines up and finish creating the VNet.
 
<p align="center"><br/>
<img src="https://i.imgur.com/PD4SqbA.png" height="60%" width="60%" alt="Creating a Resource Group"/>
<p align="center"><br/>
<img src="https://i.imgur.com/q1RatIw.png" height="60%" width="60%" alt="Creating a Resource Group"/>
<p align="center"><br/>
<img src="https://i.imgur.com/DSLjiXP.png" height="60%" width="60%" alt="Creating a Resource Group"/>
<p align="center"><br/>
<img src="https://i.imgur.com/selVXC8.png" height="60%" width="60%" alt="Creating a Resource Group"/>

<p align="center">
Setting up the Network Security Group: <br/>

Look for "web" once more in the home page, and then select the option to create a network security group. Assign a name to the network security group, in this case RedTeam-SG, and add it to the resource group created before. Double-check that the security group is set up in the same region as the resource group and the virtual network that were set up in previous steps.
 
<p align="center"><br/>
<img src="https://i.imgur.com/byvq9oB.png" height="60%" width="60%" alt="Creating a Resource Group"/>
 
<p align="center">
Set up Inbound Rules for the Network Security Group: <br/>






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
