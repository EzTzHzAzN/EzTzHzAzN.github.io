# EzTzHzAzN.github.io
An overview of my current homelab network, primarily focusing on physical network changes, wifi configuration, a toplogy overview and vlan configuration.

### Physical Hardware
Main Switch - Dell PowerConnect 5524P
IoT Switch - NETGEAR GS108E
Modem - Xfinity XB8
Router - NETGEAR Nighthawk RAX70
Raspberry Pi 5

### Deployed Software
OPNSENSE VM
Kali Linux VM
Pihole
Unbound
Wireshark

### 1. Diagnosing the Network
#### Note: I solved this issue before I thought to document it, this write up is based on memory and notes that I had written during the diagnosis.
This was the first task that I decided to tackle, to get hands on experience with troubleshooting and configuring a network. My computer had always run slow and I assumed it was because of the MoCA connector that I was using being a bottleneck. However, while studying for NET+ I learned that MoCA 2.5 should be capable of running 2.5 gbps, so I had to investigate. 

I decided to start my investigation with an internet speed test to ensure that my internet was *actually* running slow. The results came in: 300 mbps down, 15 mbps up. I found this odd because the MoCA 2.5 allows up to a 2.5 gbps speed, and my internet plan ensures at least 1.2 gbps. So the results where much slower than expected. I decided to tackle configuring the router first, I noticed that "smart connect" was enabled so I disabled that and pushed a 5 ghz network, as well as a seperate 5 ghz network with QoS. Not much else was out of place here, but I did take note that this device was in Router Mode. I then decided to look into the XB8, and the first thing that I see is that Router Mode is enabled as well. I realized that both devices were running NAT, causing double NAT, where my traffic was being translated twice before reaching the internet. While I am aware that this issue does cause negligible performance decrease, I wanted to solve it and see if I could uncover anything else.

### 2. Addressing The Issue
With the issue pinpointed 
