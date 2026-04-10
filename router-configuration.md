This was another steep learning curve for me as it was my first time working with a fully customizable router interface, but I did try to force myself to learn and understand the CLI that RouterOS has. So I first started by changing the router’s ip address subnet to match the one that my network was already working on. To do this I performed the following command:


	/ip address add address=192.168.88.1 interface=ether1

The reason the code is structured like this is that RouterOS is based off of linux so its CLI functions in a similar fashion. We start with / because that is the equivalent of a directory, and then specify the directory that we are navigating to, in this case, IP. After that we select our sub-directory, which is address and then specify the action we want to perform, in this case adding an address. The reason that we select ether1 as the interface for this command is it is our WAN IP and will be masqueraded with a later firewall rule. After configuring my IP address I decided that the next logical step was to set up my DHCP. I started with creating the DHCP server which I used the command,

	/ip dhcp-server add name=defconf interface=ether5 address-pool=defconf_pool

This created the DHCP server defconf which operates on interface five, and uses the defconf_pool to assign IP addresses from that pool. After creating the DHCP server, we have to configure the pool, which looks like this,

	/ip pool add name=defconf_pool ranges=192.168.88.100-200

This command navigates to  defconf_pool and identifies the ranges between 192.168.88.100-200 as the pool for the DHCP server. Finally after the DHCP server and DHCP pool have been created, we need to create the DHCP network that it will function on. 

	/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=8.8.8.8

This was the last command, which identified which network the defconf DHCP server was going to function on, the gateway of the network and the dns-server that the network uses. My setup varied slightly as I use unbound for my DNS server however, I didn’t want to include my actual IP for the Pi-hole in this write-up. After creating the DHCP server, the next logical step was to create the VLANs, beginning with the interface. I will only create one VLAN with CLI commands as the workflow for all the other VLANs was the same.
	/interface vlan add name=”Vlan10” vlan-id=10 interface=ether5 mtu=1500

Essentially, this command declares the name of the VLAN (VLAN10), the id of the VLAN (10) and the maximum transmission unit (mtu) of 1500 bytes. While this sets the frame for the VLAN, there are quite a few more steps we need to take in order for the VLAN to become functional, the first is creating an IP address and then assign it to the VLAN we want it to function in

	/ip address add address 10.0.10.1/24 interface=vlan10

With this command we create the IP subnet 10.0.10.1/24 and then assign it to the VLAN we want to use this range, in this case VLAN 10. This corresponds to the subnet VLAN 10 that I created on the switch, and uses 10.0.10.1 as the subnet gateway. After creating this we need to create the VLAN in the bridge as well. The reason we need to create this is that it creates a VLAN table for the bridge which allows it to recognize the VLANs by their id and what interface(s) carry them. In order to do this we will follow this workflow:

	/interface bridge vlan add bridge=bridge vlan-ids=10 tagged=ether5

After this we need to configure an important firewall rule that pertains to NAT. This rule is the masquerade rule. Essentially what this does is provides all local IP addresses with a single public-facing IP address. It does this by using a NAT table, where when a request is received by a local IP address it is stored in the NAT table. The NAT table stores the IP address and port the request was sent from and then a corresponding entry is created for the public IP and the source port it used. After the response from the destination is received, the router checks the source port and then matches it to the IP address and port that made the request. The command that 
I used to implement this rule was:

	/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade

Another important firewall rule that we need to configure is prevention of inter-VLAN communication. The reason that this is important specifically with my setup is that on my homelab I plan to do malware analysis and I want to make sure that if anything goes wrong it can only harm my homelab network specifically. To implement this rule we would use the command:

	/ip firewall filter add chain=forward in-interface=bridge out-interface=bridge action=drop
Let’s break this command down quickly, essentially chain=forward references traffic that is passing through the router. In RouterOS there are three options for the firewall filter, input which is traffic going to the router. Output which is traffic coming from the router, and forward which we’ve already discussed. The reason that we use forward in this instance is that the traffic is intended to pass through the router for inter-VLAN communication. We use the in-interface and out-interface as the bridge because the traffic will pass from the switch to the router on the bridge, and when it tries to exit back to the switch from the bridge it will trigger the firewall rule dropping it.  
As mentioned in my previous repository, I had implemented a pi-hole/unbound configuration in my network. Now, I have a couple options, either I can place it at the root of my network directly connected to the router. The other option is to create a VLAN for the pi-hole and then allow all devices to communicate via port 53 with it. I ended up opting for the second option because it is more security focused and forces me to learn more about RouterOS. In order to do this we need to create a filter rule that explicitly allows only the port 53 tcp and udp on the VLAN to be accessed by other VLANs so that it can also be used as our DNS server. In order to do this we will use this following command:

	/ip firewall filter add chain=forward protocol=tcp dst-port=53 in-interface=vlan10 out-interface=vlan99 action=accept
	/ip firewall filter add chain=forward protocol=udp dst-port=53 in-interface=vlan10 out-interface=vlan99 action=accept

In the last command we discussed why we use forward, and it applies here as well. The traffic is intended to go through the router to VLAN 99 where the pi-hole/unbound setup resides. Now, if you look closely you’ll notice that the only difference between the two commands is the protocol. The reasoning for this is that DNS functions on both tcp and udp so it’s important to allow both so all traffic reaches the unbound server. It also allows for the pi-hole to perform network wide protection for all traffic. We use dst-port=53 because that is the port on which DNS functions, and specify the in-interface in this example as VLAN 10 as it will be the interface requesting information. The out-interface is VLAN 99 as it will be the interface sending out the information, and finally we use accept because we want this traffic to go through the firewall.
After providing our VLANs with a DNS server, we need to allow them to connect to the internet. When the router is configured, the firewall only creates rules that allow the physical ports (ether2-5) to access the internet. This means that we need to create firewall rules to allow for the new virtual interfaces to access the internet. The command that we use to create these rules is:

	/ip firewall filter add chain=forward in-interface=vlan10 out-interface=ether1 action=accept

This command is quite simple. Once again we are using the forward chain since our traffic is intended for the internet. The in-interface is VLAN 10 because that is the interface that will be sending the traffic, and the out-interface is ether1 because that is the WAN port. Finally, we set the action to accept because we want to allow this traffic to pass through the filter
Related to this, we need to allow the VLANs to establish new connections. Since we have already enabled them to access the internet, we need to allow them to establish connections with websites and other media. In order to do this we would use the command:

	/ip firewall filter add chain=forward in-interface=vlan10 out-interface=ether1 connection-state=new action=accept

Let’s break this command down. Starting with the chain, once again the traffic is intended to pass through the router so we will use forward. The in-interface we are using in VLAN 10 in this example because that is the interface that is going to be establishing a new connection. The out-interface is ether1 because that is the internet and we want to establish a new connection with any resource we request. After that we specify connection-state=new because that tells the 
firewall this rule pertains to only new connections and the action is accept, telling the router to allow these new connections.
