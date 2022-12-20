# Deploy ASAv on Azure

The beginner's guide to deploying ASAv on Azure from start to finish.

## References

**Cisco ASAv Getting Started Guide:**

<https://www.cisco.com/c/en/us/td/docs/security/asa/asa98/asav/quick-start-book/asav-98-qsg/asav-azure.html>

**Azure Virtual Network Traffic Routing**

<https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview>


## Notes

- ASAv on Azure is "BYOL": bring your own license. Unlicensed ASAv instances still have a full feature set but interface speeds are greatly reduced
- Refer to the [Cisco ASAv datasheet](https://www.cisco.com/c/en/us/products/collateral/security/adaptive-security-virtual-appliance-asav/adapt-security-virtual-appliance-ds.html) for recommended VM types
- Never assign a static IP within the OS of an Azure virtual machine. VM's should always be set to DHCP and within the Azure IP configuration on the virtual NIC you can set a static IP. Think of this like DHCP reservations: the client is still set to DHCP but it will always get the "static" IP you set within the Azure portal. 

## Deployment

### 1. Create Resource ASAv - BYOL (bring your own license)
- From the Azure Marketplace (search "Market Place" from the Azure Portal), search for "ASAv", click "Create", and select "Cisco Adaptive Security Appliance (ASAv) - BYOL"

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/0.%20Create-ASAv-Marketplace.png?raw=true "asav-on-azure-marketplace")

- ASAv Basics

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/1.%20asav-deployment-basics.png?raw=true "asav-deployment-basics")


- ASAv Settings

	- Config requires four subnets, one for each ASAv required interface (Management, Gigabit Ethernet 0 - 2)

	- Name each subnet appropriately for easy interface identification like below

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/2.%20asav-deployment-settinsg.png?raw=true "asav-deployment-settings")



- Review + Create

### 2. Deploy Ubuntu Bastion Host
- Add subnet to the Hub Vnet for the bastion host

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/3.%20bastion-subnet-creation.png?raw=true "bastion-subnet-creation")


- Deploy Ubuntu VM into the bastion subnet with SSH allowed from the internet
- Update ASAv interfaces & routing for Bastion access
	- SSH into the ASAv public IP (tied to the management interface)
	- Configure the inside IP 

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/4.%20update-asav-inside-interface.png?raw=true "update-asav-inside-interface")

```
int g0/1
nameif inside
ip address dhcp
no shut
```

- Add an inside static route to the bastion subnet and update SSH, HTTP, and ICMP

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/5.%20add%20asav-inside-route.png?raw=true "add-asav-inside-route")


```
route inside 10.20.20.0 255.255.255.0 10.20.11.1
ssh 10.20.20.0 255.255.255.0 inside
icmp permit 10.20.20.0 255.255.255.0 inside
http server enable
http 10.20.20.0 255.255.255.0 inside
wr
```

- SSH into the bastion host from the internet and confirm access to the ASAv instance via the bastion host. **Be sure to save config before proceeding.**

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/6.%20ssh-from-bastion.png?raw=true "ssh-from-bastion")



### 3. Configure Public IP & Routing

- Be sure to save the ASAv config.
- Shutdown the ASAv instance from the Azure portal (select option to preserve public IP).
- In the ASAv Azure portal network settings, update the static IP assignment to .254 for each ASAv interface
	- Settings > Networking > Interface > Settings > IP Configuration > ipconfig1 > update static assignment

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/7.%20update-asav-ipconfig1-settings.png?raw=true "update-asav-ipconfig1-settings")


- Repeat for each interface

- From the Azure portal, search public IP and create a new basic public IP to be used on the outside interface.

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/8.%20create-asav-outside-public-ip.png?raw=true "create-asav-outside-pubilc-ip")


- Under the IP Configurations for demo-asav-1-NIC1, associate the new outside public IP created in the last step.

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/9.%20associate-public-ip-with-outside.png?raw=true "associate-public-ip-with-outside")


- Attach the demo-asav-1-SecurityGroup SG to the demo-asav-1-Nic1 interface.
- Turn on the ASAv instance and SSH into it via the bastion host.
- Update the outside interface using setroute on the dhcp command (remove set route from management).

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/10.%20update-asav-outside-interface.png?raw=true "update-asav-outside-interface")

``` 
int g0/0
nameif ouside
ip address dhcp
no shut
int management 0/0
no ip address dhcp setroute
shut
no shut
```
- Add a default quad zero route to the default gateway of the outside interface subnet (Azure gateways will always be the first available IP in the subnet) and update the SSH/ICMP/HTTP config. ***Please note that allowing access from any IP on the internet is only for the purposes of this lab. In productions deployments ssh/http/icmp should be locked down to trusted IPs.***

	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/11.%20update-asav-routing-and-access.png?raw=true "update-asav-routing-and-access")


```
route outside 0.0.0.0 0.0.0.0 10.20.10.1
ssh 0.0.0.0 0.0.0.0 outside
http 0.0.0.0 0.0.0.0 outside
icmp permit 0.0.0.0 0.0.0.0 outside
```
- Test logging in via the outside interface public IP. Save config.
    
	![alt text](https://github.com/wiknetauto/deploy-asav-on-Azure/blob/main/Images/12.%20test-asav-outside-access.png?raw=true "test-asav-outside-access")