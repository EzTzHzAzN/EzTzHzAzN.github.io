# <ins>Configuring the Switch</ins>
	This was one of the steepest learning curves that I encountered during this project. Once I logged in I had no idea how to actually work the interface so I used Dell’s user guide, as well as deepseek AI for any clarifying questions about the syntax. I didn’t use the setup wizard because I wanted to learn how to work with the interface myself. I started with the basic configuration set up, starting with enabling SSH, creating a user and creating VLANs. I’ll first start with how I configured SSH going over the workflow:

	console> enable (to enter privileged exec mode)
	console# configure (to enter configuration mode)
	console(config)# crypto key generate rsa (to generate the SSH RSA crypto key)
	console(config)# crypto key generate dsa (to generate the SSH DSA crypto key)
	console(config)# ip ssh server (to activate ssh server)
	console(config)# exit
	console# copy running-config startup-config (to save configuration)

	It is important to note that for the model of powerconnect I am using, the 5524p, it is required to generate both an RSA and DSA crypto key to enable ssh. After starting my SSH server, I configured my admin user, and to do that I performed these commands, for this workflow I am already in privileged exec mode.
	
	console#configure
	console(config)# username <myusername> password <mypassword> privilege 15
	console(config)# exit
	console# copy running-config startup-config

	Finally, the last basic configuration I did was configuring my VLANs, I will only run through one of the workflows since they’re all the same.

	console#configure
	console(config)# vlan database
	console(config-vlan)# vlan 10
	console(config-vlan)#name Homelab
	console(config-vlan)# exit
	console(config)# interface vlan 10
	console(config-vlan)# ip address 10.0.10.254 255.255.255.0
	console(config-vlan)#exit
	console(config)# interface range gi1/0/9-12
	console(config-if-range)#switchport mode access
	console(config-if-range)# switchport access vlan 10
	console(config-if-range)# exit
	console(config)# exit
	console#copy running-config startup-config

	It’s important to remember to add the trunk port as well so these ports are tagged as VLAN 10. Here is the workflow for that,
	
	console#configure
	console(config)# interface gi1/0/24
	console(config-if)# switchport mode trunk
	console(config-if)# switchport trunk vlan allowed add 10
	console(config-if)# exit
	console(config)# exit
	console# copy running-config startup-config

	For my other VLANs, I followed this exact workflow. After I ensured that they were all properly configured I enabled IP routing and created a static route to my router so traffic from the switch knows where to go when it doesn’t know where to send traffic. The reason that IP routing is important is that it allows the switch to function at layer 3 so that it can route traffic between the VLANs.This was the work flow I followed for that,
	
	console# configure
	console(config)#  ip routing
	console(config)#  ip route 0.0.0.0 0.0.0.0 192.168.88.1
	console(config)#  exit
	console# copy running-config startup-config

	Now that my switch is fully configured it’s time to work on my router, I actually had to purchase an entirely new router for this setup because after configuring my router and plugging it into my netgear, the router wasn’t able to route to the VLANs. The router that I purchased for this project was a MikroTik hEX RB750Gr3 because it has RouterOS which is common in enterprise situations. 
