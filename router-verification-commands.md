While most of the work done was setting up the router, it is always important to be able to verify that what you’ve deployed is functioning as intended. Some of the most important commands that I used when troubleshooting or verifying that my deployment worked are:

```
/interface print

Example:

#    NAME    TYPE      ACTUAL-MTU  L2MTU  MAX-L2MTU  MAC-ADDRESS      
 0 R  ether1  ether           1500               1596         2026        00:00:00:00:00
```

This command prints all of the interfaces, both physical and virtual, in the terminal. This is an important command because it shows only valid interfaces as well as the options that have been configured to those interfaces. 

```
/interface bridge vlan print

Example:

# BRIDGE  VLAN-IDS
;;; Homelab_VLAN
0 bridge        10
```

This command prints all of the VLANs contained within the bridge directory. This is important because when configuring VLANs, you need to ensure that they are in the bridge so that the bridge can route traffic to them. I had to use this command when creating my VLANs as I had originally forgotten to put them into the bridge.

```
/interface bridge port print

Example:

#    INTERFACE  BRIDGE  HW   PVID  PRIORITY  HORIZON
;;; defconf
0 IH ether2               bridge  yes     1           0x80          none   
```

This is used to see each physical port of the router. In this example, we see an IH following the number. This represents inactive and HW offload, meaning that this port is inactive and supports HW offload. HW offload allows for traffic traveling within the same VLAN to be processed by the switch chip (layer 2) instead of the CPU (layer 3). This allows for faster travel of traffic within a network. However, it is important to note that the cpu will be used to process traffic that triggers a firewall rule. This includes traffic leaving ether1, and inter-VLAN traffic.

```
/ip address print

Example:

#   ADDRESS           NETWORK      INTERFACE
;;; defconf
0   192.168.1.1/24    192.168.1.0           bridge
```

This command is important for verification because it shows all the IP addresses contained in the router. I used this command to ensure that the IP addresses I assigned my VLANs were correct, as well as ensuring that my default gateway IP address matched the currently existing IP before the router was switched out.

```
/ip dhcp-server print

Example:

# NAME                INTERFACE  ADDRESS-POOL      LEASE-TIME
0 dhcp1                      bridge           dhcp_pool1                    1d        
```

This command prints the DHCP servers that you’ve configured

```
/ip pool print

Example:

#  NAME              RANGES                           TOTAL  USED  AVAILABLE
0  dhcp_pool1 192.168.1.50-192.168.1.200    151         7        144
```

This command prints the DHCP pools that are used to assign IP addresses. This prints every DHCP pool that you’ve configured, which is important because generally you’ll have a set range of IP addresses you want to assign to devices. This command allows you to verify that you’ve set the proper ranges for your DHCP pools.

```
/ip firewall filter print stats

Example:

 #   CHAIN       ACTION                          BYTES          PACKETS
;;; special dummy rule to show fasttrack counters
 0 D forward  passthrough           531 897 567 269  481 606 487
```

This command prints the stats of the firewall rules that reside under the filter sub-directory. What it does is show the packets and bytes that have passed through the firewall that matched a specific rule. This is important to check after deployment to ensure that all rules are working as intended.

```
/ip firewall nat print 

Example:

# CHAIN   ACTION            BYTES    PACKETS
;;; defconf: masquerade
0 srcnat  masquerade  326 294 260  1 512 702
```

This command prints the stats of the firewall rules that reside under the NAT sub-directory, which is where this differs from the last command. This command was important during my configuration as I used it to ensure that my masquerade command was working as intended.

```
/ip dhcp-server lease print

Example:

#   ADDRESS        MAC-ADDRESS        HOST-NAME        SERVER              STATUS  LAST-SEEN
 0 D 10.0.30.101    68:4E:05:EF:50:4D         Home         vlan30_dhcp_server       bound   3h8m44s  
```

This command prints all of the devices that are currently leasing an IP address from the router and provides information on each one. This helped me a lot personally when I was trying to configure my pi-hole because after moving it to this router I needed to know the new IP address to ssh into the pi-hole and set a static IP address. It is also useful for ensuring that all devices are on the correct VLANs

```
/ip route print

DST-ADDRESS        GATEWAY      DISTANCE
DAd 0.0.0.0/0            x.x.x.x                 1
DAc x.x.x.x/21          ether1                 0
DAc 10.0.10.0/24      VLAN10               0	
```

Let me break the information in this command down quickly. The DST-ADDRESS is the address in which the gateway sends its traffic. I learned while working on this configuration that distance in this output represents administrative distance, rather than hop counts. Meaning that the router uses a scale between 0-255 to determine how trustworthy a route path is, where 0 is the most trustworthy as it is an interface on the router itself. 255 is deemed to be the most untrustworthy. Another important number, as seen in this output is 1 which means that it is very trustworthy as it is a static route that you have put in the router.
