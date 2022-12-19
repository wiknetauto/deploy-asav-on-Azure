# Deploy ASAv on Azure

The beginner's guide to deploying ASAv on Azure from start to finish.

## References

**Cisco ASAv Getting Started Guide:**

<https://www.cisco.com/c/en/us/td/docs/security/asa/asa98/asav/quick-start-book/asav-98-qsg/asav-azure.html>

**Azure Virtual Network Traffic Routing**

<https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview>

## Deployment

### 1. Create Resource ASAv - BYOL (bring your own license)
	a. ASAv Basics
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/1.%20asav-deployment-basics.png?raw=true "asav-deployment-basics")

	b. ASAv Settings
		 i. Config requires four subnets, one for each ASAv required interface (Management, Gigabit Ethernet 0 - 2)
		ii. Name each subnet appropriately for easy interface identification like below
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/2.%20asav-deployment-settinsg.png?raw=true "asav-deployment-settings")


	c. Review + Create

### 2. Deploy Ubuntu Bastion Host
	a. Add subnet to the Hub Vnet for the bastion host
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/3.%20bastion-subnet-creation.png?raw=true "bastion-subnet-creation")

	b. Deploy Ubuntu VM into the bastion subnet with SSH allowed from the internet
	c. Update ASAv interfaces & routing for Bastion access
		  i. SSH into the ASAv public IP (tied to the management interface)
		 ii. Configure the inside IP
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/4.%20update-asav-inside-interface.png?raw=true "update-asav-inside-interface")

		iii. Add an inside static route to the bastion subnet and update SSH, HTTP, and ICMP
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/5.%20add%20asav-inside-route.png?raw=true "add-asav-inside-route")

		 iv. SSH into the bastion host from the internet and confirm access to the ASAv instance via the bastion host. **Be sure to save config before proceeding.**
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/6.%20ssh-from-bastion.png?raw=true "ssh-from-bastion")


### 3. Configure Public IP & Routing

	a. Be sure to save the ASAv config.
	b. Shutdown the ASAv instance from the Azure portal (select option to preserve public IP).
	c. In the ASAv Azure portal network settings, update the static IP assignment to .254 for each ASAv interface
	  	 i. Settings > Networking > Interface > Settings > IP Configuration > ipconfig1 > update static assignment
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/7.%20update-asav-ipconfig1-settings.png?raw=true "update-asav-ipconfig1-settings")

		ii. Repeat for each interface

	d. From the Azure portal, search public IP and create a new basic public IP to be used on the outside interface.
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/8.%20create-asav-outside-public-ip.png?raw=true "create-asav-outside-pubilc-ip")

	e. Under the IP Configurations for demo-asav-1-NIC1, associate the new outside public IP created in the last step.
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/9.%20associate-public-ip-with-outside.png?raw=true "associate-public-ip-with-outside")

	f. Attach the demo-asav-1-SecurityGroup SG to the demo-asav-1-Nic1 interface.
	g. Turn on the ASAv instance and SSH into it via the bastion host.
	h. Update the outside interface using setroute on the dhcp command (remove set route from management).
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/10.%20update-asav-outside-interface.png?raw=true "update-asav-outside-interface")

	i. Add a default quad zero route to the default gateway of the outside interface subnet (Azure gateways will always be the first available IP in the subnet) and update the SSH/ICMP/HTTP config.
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/11.%20update-asav-routing-and-access.png?raw=true "update-asav-routing-and-access")

	j. Test logging in via the outside interface public IP. Save config.
![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/12.%20test-asav-outside-access.png?raw=true "test-asav-outside-access")
