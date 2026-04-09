Verification Commands

Since this was my first time configuring a switch learning this new language was very difficult, however there was one character that actually helped me a lot without having to read through pages of commands. That being the ? character, because if you type a command such as show and place the ? after it, it will show you all valid options that can follow. This helped me because if a command returned an error I was able to troubleshoot it easily.

There were a few important commands that I used to verify that all of my configurations were correct which I will now go over.

	console# show vlan
	
	Example:
	
	|Vlan |    Name     |  Tagged Ports     | UnTagged Ports     |   Type     |  Authorization
	|-----| ----------- | ----------------- | ------------------ |  --------- |  ----------------
	|1    | 1           |                   |   gi1/0/17-48,     |  Default   |    Required
	|10   | homelab     |    gi1/0/24       |   gi1/0/9-12       |  permanent |    Required
	|20   | IoT         |    gi1/0/24       |   gi1/0/13-16      |  permanent |    Required



This is an essential command for verification because it lists all of the VLANs, and the information about their configuration. Let’s take VLAN 10 for example, we can see the vlan-id = 10, the name is homelab, it’s tagged (trunk) port is port 24. The untagged ports which are the ports assigned to the VLAN are 9-12 and its a permanent VLAN that requires authorization. This is all the configurations I made to VLAN 10, and are correct, but if they weren’t I’d be able to identify it with this command and then fix it.


	console# show ip interface

	Example:
	
	    IP Address         I/F            Type     Directed   Precedence   Status
	                                                             Broadcast
	------------------- --------- ----------- ---------- ---------- -----------------------------------
	 10.0.10.254/24      vlan 10    Static     disable            No               Valid
	 10.0.20.254/24      vlan 20    Static     disable            No               Valid
	 10.0.30.254/24      vlan 30    Static     disable            No               Valid
	 10.0.99.254/24      vlan 99    Static     disable            No               Valid

This is another essential command for verifying that your configuration is correct. This command displays your IP address, which interface (I/F) that IP address is assigned to, and its type, which can be either static or dynamic. Directed broadcast is disabled by default specifically on layer 3  devices, for security reasons as it permits one device to send a broadcast to another VLAN, meaning it would reach every device on that other VLAN. Precedence refers to QoS and because we are not using QoS we do not have any precedence configured. However, if we did, the IP with the highest precedence would be placed at the highest priority for traffic. Finally, status tells us if the IP is either valid or invalid, and in this instance we see all of our IPs in this example are valid.[^1]

	console# show running config

	Example:
	
	vlan database
	vlan 10,20,30,99
	exit
	voice vlan oui-table add 000181 Nortel__________________
	voice vlan oui-table add 0001e3 Siemens_AG_phone________

This command is important to check your current running configuration which is important to check the current configuration. This is helpful to ensure that all changes you’ve made during this session are correct. In this example you can see the VLAN database which contains the VLANs 10,20,30,99. The oui tables show the mac address prefix for Nortel which is 000181 and Siemens which is 0001e3. These are the prefixes to the mac address of a device, and in this example I do not have VoIP enabled but it is still added by default.
I also didn’t include its brother command show startup-config but this shows the configuration that the switches boots with.


	console# show interface switchport gi1/0/24

	Example:
	
	Name: gi1/0/24
	Switchport: enable
	Administrative Mode: trunk

This looks like an unconventional command however with Dell’s CLI instead of show interface trunk, which is used in cisco, this is the command that is used to show the trunk. However, it is important to note that using the last port on a switch is considered good practice in some edge cases it may differ. In those instances you would replace the 24 with whichever port is configured to the trunk, it would look like show interface switchport gi1/0/x where x is the chosen port. Another important thing to note is that this exact command can be performed on non-trunk ports to view the details of those ports.

	console# show users

	Example:
	
	 Username       Protocol          Location
	--------------- ------------ -----------------------
	                           Serial              0.0.0.0
	   admin              SSH                192.168.1.20

This command shows all active users in the switch, including their username, the protocol they used to access the switch and location they are signed in at. In this example you can see that I am accessing the admin account via ssh at 192.168.1.20. The first entry is what a serial entry would look like, however I had to investigate it as the output should only include active sessions and I will include my findings below.


	console# show interface status

	Example:
	
	                                                                        Flow  Link            Back    Mdix
	Port     Type         Duplex  Speed     Neg    ctrl    State       Pressure Mode
	-------- ------------ ------  ----- -------- ---- ----------- -------- -----------------------------
	gi1/0/1  1G-Copper    Full    1000  Enabled  On   Up       Disabled   On
	
	Note: Truncated output as it is redundant.

This command is used to view the each interface and its configurations. In this example we can see that the output provides us with port 1, which is using a 1 gigabit copper wire connection (1000Base-T ethernet). We can see that the negotiated duplex is full meaning that it can both send traffic and receive traffic. We can also see that our speed is at 1000 which indicates 1000mbps or 1gbps. Neg is short for negotiation and by modern standards is turned on because it allows for the duplex and speed negotiation to automatically occur. Flow control is on meaning that if the switch gets overloaded with traffic it is able to pause the traffic, and keep in mind that this was designed for full duplex traffic only, we will discuss half-duplex in a sec. Link state is used to ensure that the connection is up. Back pressure is a deprecated method of flow control for half-duplex, it is off because we are using full duplex. Mdix was one that I had to learn, and it essentially means that the switch is able to detect if the cable connected to it is a straight through or crossover.

	console# show ip route

	Example:
	
	Maximum Parallel Paths: 1 (1 after reset)
	IP Forwarding:          enabled
	
	Codes: C - connected, S - static, D - DHCP
	
	S  0.0.0.0/0             [1/1] via  192.168.1.1  406:18:40        vlan 1
	C  10.0.10.0/24       is directly connected                        vlan 10
	C  10.0.20.0/24       is directly connected                        vlan 20
	C  10.0.30.0/24       is directly connected                        vlan 30
	C  10.0.99.0/24       is directly connected                        vlan 99
	C  192.168.1.0/24   is directly connected                        vlan 1


This command is another really important command when verifying configurations within the powerconnect. We can see that 0.0.0.0/0 is set as static and connects to 192.168.1.1 which is our router's ip address. This is the route that our switch will send traffic to when it doesn’t know where to send traffic, this is known as the default route. Another example we can take a look at is the second output, C 10.0.10.0/24. The switch tells us that this IP address is connected to VLAN 10 meaning that whenever the switch receives a packet destined for this IP subnet it will know to send it to VLAN 10. The same goes for packets sent from VLAN 10, when the switch see that it comes from an IP address in this subnet it recognizes and tags it as VLAN 10.[^2]

	console# show mac address-table

	Example:
	
	Aging time is 300 sec
	
	  Vlan        Mac Address         Port       Type
	-------- --------------------- ---------- ----------
	   1       05:g4:2d:90:ab:5b    gi1/0/24   dynamic
	   1       05:g4:2d:90:ab:5e    gi1/0/24  dynamic
	   1       2v:f4:5d:9c:9d:87    gi1/0/24   dynamic
	   1       04:18:65:f8:20:84    gi1/0/24   dynamic
	   1       04:18:65:f8:d3:bc    gi1/0/24   dynamic
	   1       g0:67:e5:bd:90:50       0          self

This command is important because it shows the MAC addresses that are connected to the switch, which VLAN they reside on, the port they’re connected to and as well as how the switch obtained the mac address. We can use the first output as an example, seeing that the device is connected to VLAN one via port 24 (our trunk port) and is a dynamic type meaning the switch learns the mac address after switching traffic from the device. Let’s take a look at output 6, where we can see that type is self, and this means that it is the switches own mac address, which explains why it resides on port 0.

[^1]: Output has been truncated.
[^2]: MAC Addresses have been obfuscated for security reasons.
