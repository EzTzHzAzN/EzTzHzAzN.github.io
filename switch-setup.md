# Setting up the switch

I started this project by first researching the way airflow works in the PowerConnect 5524 which I learned goes front to back, so when I set it up I ensured that it was in a clean environment, and that its exhaust had enough room. After that, I knew I was going to configure VLANs so I mapped out the VLANs I was going to create, as well as the ports that corresponded to them. This was what I decided on:

	VLAN 10 - Homelab VLAN (gi1/0/9-12)
	V LAN 20 - Entertainment (gi1/0/13-16)
	VLAN 30 - Wireless (gi1/0/1-4)
	VLAN 99 - Networking Devices (gi1/0/5-8)
	Trunk - gi1/0/24

I only wanted to assign four ports per VLAN because I wanted to leave ports 17-23 for future expansion as I plan on adding on a lot more to the network. 
I learned while trying to configure my switch that it requires a serial cable and that to communicate with the interface I was going to need to download PuTTY. So I ordered the cable, and downloaded PuTTY, and was finally able to access the switch’s interface. 
