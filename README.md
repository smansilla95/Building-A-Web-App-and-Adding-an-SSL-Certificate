<h1>Building A Web App and Adding an SSL Certificate</h1>


<h2>Description</h2>
Project consisted of setting up a domain through GoDaddy and then creating a web application using Azure Labs. Using the tools available within Azure, a resource group, virtual network, and network security group were set up, complete with accessibility rules. Additionally, three virtual machines were established, one being a jump box while the other two were used for web access. The jump box virtual machine was used to set up a docker container, provisioner, and Ansible playbook. A load balancer was also set up, configuring a static IP address and a backend pool connecting both web virtual machines to it. Using OpenSSL, self-signed certificates were created and then added to a Key Vault so that they could be bound to the web application.
<br />


<h2>Utilities and Environments Used</h2>

- Microsoft Azure Cloud Services
- Terminal
- GoDaddy

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
 
 - Source: All, so that it can block all traffic sources.
 - Source Port Ranges: Wildcard(*), so that it can match all source ports.
 - Destination: Any, so that it will block all traffic associated with the security group.
 - Service: Custom.
 - Destination Port Ranges: Wildcard(*), so that it will block all destination ports.
 - Protocol: Any, so that all protocols used will be blocked.
 - Action: Deny, so that it will block any traffic that matches this rule.
 - Priority: The highest priority number (in this case 4,096).
 - Name: Default-Deny
 - Description: "Deny All Inbound Traffic"
 
<p align="center"><br/>
<img src="https://i.imgur.com/ZoRz22c.png" height="60%" width="60%" alt="Set up Inbound Rules for the Network Security Group"/>
 
<p align="center"><br/>
<img src="https://i.imgur.com/qKz4foW.png" height="60%" width="60%" alt="Set up Inbound Rules for the Network Security Group"/>

<p align="center">
Setting Up a Jumpbox Virtual Machine: <br/>

Begin by creating an SSH key pair using Terminal. Run the command ssh-keygen and after verifying the output looks correct, run the command cat ~/.ssh/id_rsa.pub. Copy the output of that command, which is the SSH key string, and keep in your clipboard for later. After the key pair is generated, return to Azure and search for "virtual machines" in the search bar. Click on +Add to begin creating the VM. This VM requires these settings: 
 
<p align="left">
Basics: <br/>
 
 - Resource Group: The same as the one used for Red Team, in this case Red-Team.
 - Virtual Machine Name: This VM will be called Jump Box Provisioner.
 - Region: Try to match the VM region to the one used for the resource and security groups (in this case Central US), but if it is unavailable, then create a new resource and security group that will match the one used to create the VM.
 - Availability Options: This VM will continue to use the default settings for availability.
 - Image: The Ubuntu Server 18.04 option will be used for this VM.
 - Size: This VM will use Standard-B1s, 1 CPU, and 1 RAM.
 - SSH: The authentication type will be an SSH Public Key, the username will be RedAdmin, and in the "SSH Public Key" field, paste the SSH key that was created before and copied to the clipboard.
 - Public Inbound Ports: Leave the default setting provided, since it will be overwritten by choosing the security group created earlier.
 - Select Inbound Ports: Like the previous selection, leave the default setting, as it will be overwritten by the security group rules.

<p align="left">
Networking: <br/>
 
 - Virtual Network: The VNet created for Red Team, in this case RedNet.
 - Subnet: The subnet that was created earlier, RedNetBase, will be used here.
 - Public IP: The default setting for this selection will be used.
 - NIC Network Security Group: The Advanced option will be used here, since it will allow for a specification of a custom security group.
 - Configure Network Security Group: RedTeamSG, which was created earlier, will be used here.
 - Accelerated Networking: Select "off" for this setting.
 - Load Balancing: Click "No" for this setting, which should be the default.
 
<p align="center"><br/>
<img src="https://i.imgur.com/GmKGnAB.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>
 
<p align="center"><br/>
<img src="https://i.imgur.com/O8osUDz.png" height="60%" width="60%" alt="Setting Up a Jumpbox Virtual Machine"/>

<p align="center">
Setting Up Two Web-Access Virtual Machines: <br/>

Two VMs will need to be created for web access in the network. After going to the virtual machines tab in Azure, select +Add in order to create the two web VMS. They need to have these settings:

<p align="left">
Basics: <br/>
 
 - Resource Group: The same as the one used for Red Team, in this case Red-Team.
 - Virtual Machine Name: These VMS will be called Web-1 and Web-2.
 - Region: Try to match the VM region to the one used for the resource and security groups (in this case Central US), but if it is unavailable, then create a new resource and security group that will match the one used to create the two VMs.
 - Availability Options: Select "Availabilty Set" and then choose "Create New". It will be called RedTeamAS and have 2 Fault Domains and 5 Update Domains. After creating it for the first web VM, make sure to choose the same availability set for the second web VM.
 - Image: The Ubuntu Server 18.04 option will be used for these VMs.
 - Size: These VMs will use Standard-B1s, 1 CPU, and 2 RAM.
 - SSH: The authentication type will be an SSH Public Key, the username will be RedAdmin, and in the "SSH Public Key" field, paste the SSH key that was created before and copied to the clipboard when creating the Jumpbox VM.
 - Public Inbound Ports: Leave the default setting provided, since it will be overwritten by choosing the security group created earlier.
 - Select Inbound Ports: Like the previous selection, leave the default setting, as it will be overwritten by the security group rules.
 
<p align="left">
Networking: <br/>
 
 - Virtual Network: The VNet created for Red Team, in this case RedNet.
 - Subnet: The subnet that was created earlier, RedNetBase, will be used here.
 - Public IP: Choose None, as these web VMs should not have a public IP address.
 - NIC Network Security Group: The Advanced option will be used here, since it will allow for a specification of a custom security group.
 - Configure Network Security Group: RedTeamSG, which was created earlier, will be used here.
 - Accelerated Networking: Select "off" for this setting.
 - Load Balancing: Click "No" for this setting, which should be the default.
 
<p align="center"><br/>
<img src="https://i.imgur.com/pg3xMZ1.png" height="60%" width="60%" alt="Setting Up Two Web-Access Virtual Machines"/>

<p align="center"><br/>
<img src="https://i.imgur.com/8yYLVMI.png" height="60%" width="60%" alt="Setting Up Two Web-Access Virtual Machines"/>
 
<p align="center"><br/>
<img src="https://i.imgur.com/HhOP0p8.png" height="60%" width="60%" alt="Setting Up Two Web-Access Virtual Machines"/>

<p align="center">
Configuring Jumpbox Administration: <br/>

In order to SSH into the jumpbox virtual machine, a rule needs to created that allows connections only from your current IP address and no other IP addresses. This is to ensure the security of the web application while also being able to control it via SSH connection. This will be achieved by first going to whatismyip.org or any other IP address identifier and taking note of the IPv4 address of the network that is currently being used. Return to Azure to create a new security rule. This will be done by finding the security group that is listed under the resource group that was used throughout the activity. From there, choose Inbound Security rules and click on +Add to begin creating the rule, which will have these selections:
 
 - Source: IP Addresses, which will open a new selection beneath it to identify the desired IP address.
 - Source IP Addresses/CIDR Ranges: Use the IP address that was identified earlier, in this case 209.58.129.97.
 - Source Port Ranges: Wildcard(*), so that it can match all source ports.
 - Destination: IP Addresses, so that the specific IP address of the Jumpbox VM can be used.
 - Destination IP Addresses/CIDR Ranges: The internal IP of the Jumpbox VM will be used here, which in this case is 10.0.0.4.
 - Service: SSH
 - Destination Port Ranges: 22
 - Protocol: TCP
 - Action: Allow
 - Priority: The highest priority number, but lower than the rule to deny all traffic.
 - Name: SSH
 - Description: "Allow SSH from specific IP address."
 
<p align="center"><br/>
<img src="https://i.imgur.com/5jPJbir.png" height="60%" width="60%" alt="Configuring Jumpbox Administration"/>
 
After the rule is set up, use the command line to SSH into the Jumpbox VM. This is done by typing <i>ssh sysadmin@10.0.0.4</i>. Once the SSH connection is achieved, check sudo permissions by running the command <i>sudo -l</i>. This is used to see that the admin has full permissions without needing a password.
 
<p align="center">
Setting Up the Docker Container: <br/>

To install docker.io on the Jumpbox VM, begin by running <i>sudo apt update</i>, followed by <i>sudo apt install docker.io</i>.
 
<p align="center"><br/>
<img src="https://i.imgur.com/Cx53Bnh.png" height="60%" width="60%" alt="Setting Up the Docker Container"/>
 
To double-check that the docker service is running, type the command <i>sudo systemctl status docker</i> into the command line. If the Docker service is not running automatically, then run the command <i>sudo systemctl start docker</i>.
 
<p align="center"><br/>
<img src="https://i.imgur.com/sfCpWs8.png" height="60%" width="60%" alt="Setting Up the Docker Container"/>
 
After verifying that the Docker is working properly, run the command <i>sudo docker pull cyberxsecurity/ansible</i>. Optionally, run the command <i>sudo su</i> to switch to the root user in order to avoid having to type <i>sudo</i> before every command.

<p align="center"><br/>
<img src="https://i.imgur.com/tb1Bxp3.png" height="60%" width="60%" alt="Setting Up the Docker Container"/>
 
In order to launch the Ansible container and connect to it, run these commands: <i>docker run -ti cyberxsecurity/ansible:latest bash</i>. This will start the container, and after it does run the command <i>exit</i> to quit.

<p align="center"><br/>
<img src="https://i.imgur.com/jimWOa5.png" height="60%" width="60%" alt="Setting Up the Docker Container"/>
 
The next step will be to create a new security group rule that will allow the Jumpbox VM to have full access to the VNet. The firt step will be to find the private IP address of the Jumpbox VM, which can be found in the overview tab of the VM.
 
<p align="center"><br/>
<img src="https://i.imgur.com/SH7hVT6.png" height="60%" width="60%" alt="Setting Up the Docker Container"/>
 
After taking note of the private IP address for the Jumpbox VM, create a new inbound rule that allows SSH connections in the security group settings. This rule will have these settings:

 - Source: IP Addresses, which will open a new selection beneath it to identify the desired IP address.
 - Source IP Addresses/CIDR Ranges: Use the private IP address of the Jumpbox VM, in this case 10.0.0.4.
 - Source Port Ranges: Wildcard(*), so that it can match all source ports.
 - Destination: VirtualNetwork
 - Service: SSH
 - Destination Port Ranges: 22
 - Protocol: TCP
 - Action: Allow
 - Priority: The highest priority number, but lower than previous established rules.
 - Name: Jump-Box-Access
 - Description: "SSH Access from Jump Box"
 
<p align="center"><br/>
<img src="https://i.imgur.com/kXm7KzJ.png" height="60%" width="60%" alt="Setting Up the Docker Container"/>

After the security rule is created, look at all the rules for the Red Team Security Group to verify that all rules have been created. These include: JumpBox-Access, SSH, Default-Deny, AllowVNetInBound, AllowAzureLoadBalancerInBound, DenyAllInBound. 

<p align="center"><br/>
<img src="https://i.imgur.com/iPMjBMZ.png" height="60%" width="60%" alt="Setting Up the Docker Container"/>
 
<p align="center">
Setting Up the Provisioner: <br/>

A new VM from the Azure portal will be launched, which can only be accessed by using a new SSH key from the container running in the Jumpbox VM. Continuing from the last step, use the command line to connect to the Ansible container and then create a new SSH key and copy the public key. This will be done by running the command <i>docker images</i> to view the image. Run the command <i>docker run -it cyberxsecurity/ansible /bin/bash</i> to start and connect to the container. To create a new SSH key, run the command <i>ssh-keygen</i>. To view all generated SSH keys, run the command <i>ls .ssh/</i>. To display the public key string, run the command <i>cat .ssh/id_rsa.pub</i>, and then copy the output. At this point, return to the Azure portal and navigate to one of the Web VM details page. There, reset the VM's password with the newly generated public key from the Ansible container, which should still be copied into the clipboard. Still in the details page of the Web VM, take note of the VM's internal IP address. 

Once the VM launches, test the connection by using SSH from the Jumpbox VM's Ansible container. This will be done by running the command <i>ping 10.0.0.6</i>, which is Web 1 VM's internal IP address. If the ping is successful, SSH into the container by running <i>ssh sysadmin@10.0.0.6</i>. After the SSH connection is successful, run the command <i>exit</i> to shut down the session. Do the same for Web 2 VM.
 
To find the Ansible config and hosts files, run the command <i>ls /etc/ansible</i>, which should reveal three files: ansible.cfg, hosts, and roles. To add Web 1 and 2 VM's interal IP addresses to the Ansible hosts file, begin by editing the file using the command <i>nano /etc/ansible/hosts</i>. From there, uncomment the [webservers] header line by deleting the # and then add Web 1 and 2 VM's internal IP addresses under the same header. Beside both IP addresses that have been edited into the file, add the line <i>ansible_python_interpreter=/usr/bin/python3</i>.
 
<p align="center"><br/>
<img src="https://i.imgur.com/iNN8UZt.png" height="60%" width="60%" alt="Setting Up the Provisioner"/>
 
Edit the Ansible config file so that it will use the admin account or SSH connections by using the command <i>nano /etc/ansible/ansible.cfg</i>. Scroll down to the <i>remote_user</i> option and uncomment it by deleting the # next to it. Once uncommented, replace <i>root</i> with <i>sysadmin</i>, which is the admin username for both Web 1 and 2 VMs. 
 
<p align="center"><br/>
<img src="https://i.imgur.com/HgvKGms.png" height="60%" width="60%" alt="Setting Up the Provisioner"/>
 
Test the Ansible connection by using the command <i>ansible_python_interpreter=/usr/bin/python3</i>.
 
<p align="center"><br/>
<img src="https://i.imgur.com/DajYaSi.png" height="60%" width="60%" alt="Setting Up the Provisioner"/>
 
<p align="center">
Setting Up Ansible Playbooks: <br/>



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
