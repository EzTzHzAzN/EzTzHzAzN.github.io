## Updates to Configurations

As I was performing this write up, I wanted to check my configuration against common practices and will document any changes that I make here. I believe that it’s important to always go back and audit my work as there is always something new to learn.

1. Updated VLAN internet access rule
When I initially created this configuration, I thought that in order for a VLAN to access the internet you had to create two separate commands. One to allow the traffic from the switch, through the bridge and into the internet. However, I learned that the two commands I used were redundant in nature since the first one already allows all connections to be established. Since I wanted a little more control over the connections that are established, I decided to create a rule for new, established and related. The reason for this is that it allows more visibility into the connections, I learned that when we use one command the router can’t distinguish new connections from related, established ones. So by separating these when analyzing traffic it will show the connections and their respective firewall rules. The new commands that I used for the firewall are:
```
/ip firewall filter add chain=forward in-interface=vlan10 out-interface=ether1 connection-state=new action=accept
/ip firewall filter add chain=forward in-interface=vlan10 out-interface=ether1 connection-state=related,established action=accept
```
2. Added VLAN Filtering
	While looking through this write-up to ensure that everything was correct, I noticed that I had missed a crucial step for my vlans. I had forgot the enable VLAN filtering as you can see from the output  /interface bridge print  vlan-filtering=no. I noticed that this was off and did some research on it learning that this feature tags and untags VLAN traffic which is incredibly important to ensuring that frames reach their correct destination. In order to turn on VLAN filtering you would use the command
```
	/interface bridge set bridge vlan-filtering=yes
```
