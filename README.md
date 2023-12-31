# GRE

to configure basic GRE there is several step:
1. create interface "tunnel".
2. input the GRE ip address(each GRE end has same ip)
3. input the source & destination ip address(it is different from GRE ip address.)

R1 to R4 is 3 hop away

To implement a GRE tunnel, perform the following actions:
1. Create a tunnel interface.
```
Router(config)# interface tunnel tunnel-id
```
2. Configure the GRE tunnel mode. This mode is a default tunnel mode, so it is not necessary to configure it.
```
Router(config-if)# tunnel mode gre ip
```
3. Configure an IP address for the tunnel interface.(this is the virtual ip used for the interface tunnel)
```
Router(config-if)# ip address ip-address mask
```
4. Specify the tunnel source IP address.(this is ip using on the interface)
```
Router(config-if)# tunnel source ip-address
```
5. Specify the tunnel destination IP address.(this is also ip on the interface)
```
Router(config-if)# tunnel destination ip-address
```

The minimum GRE tunnel configuration requires the specification of the tunnel source address and destination address. You must also configure an IP subnet to provide IP connectivity across the tunnel link.

![Creating a VLAN](https://raw.githubusercontent.com/deliawolf/GRE/main/Screenshot%202023-10-26%20at%2009.09.58.png)


NOTE : here we going to create GRE tunnel between for 2 routers(R1 & R4)
1. we create interface tunnel
2. we input GRE ip address in here the IPs is(172.16.99.1 for R1 and 172.16.99.2 for R4).
   as you can see the IPs for each GRE end is having same address
3. input the source and destination ip for each interface that GRE tunnel will created.

R1 Config:
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface tunnel 0
R1(config-if)#
*Oct 26 01:51:50.133: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to down
R1(config-if)#ip address 172.16.99.1 255.255.255.0
R1(config-if)#tunnel source 10.10.1.1
R1(config-if)#tunnel destination 10.10.3.2
*Oct 26 01:52:47.815: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
R1(config-if)#end
```
R4 Config
```
R4(config)#interface tunnel 0
*Oct 26 01:55:06.373: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to down
R4(config-if)#ip address 172.16.99.2 255.255.255.0
R4(config-if)#tunnel source 10.10.3.2
R4(config-if)#tunnel destination 10.10.1.1
*Oct 26 01:55:32.173: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
R4(config-if)#end
R4#
```

verifying the configuration
```
R4# show ip route
R4# show interface Tunnel 0
```

### Enabling OSPF on Tunnel Interface

Being able to forward packets between the two tunnel interfaces is good. But you can also run a dynamic routing protocol through the tunnel. Configure the OSPF process ID 1 on R4. Assign R4 the router ID 0.0.0.4. Include the network 172.16.0.0/16 (which includes the interfaces Ethernet0/1, Loopback0, and Tunnel0) in Area 0.

need to note those interface (eth0/1, lo0, tunnel0) is interface the router 4 has. so tunnel0 is of course the tunnel itself, the lo0 and eth0/1 is interface inside the network.

```
R1# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# router ospf 1
R1(config-router)# router-id 0.0.0.1
R1(config-router)# network 172.16.0.0 0.0.255.255 area 0
R1(config-router)#
*Nov  4 11:41:51.093: %OSPF-5-ADJCHG: Process 1, Nbr 0.0.0.4 on Tunnel0 from LOADING to FULL, Loading Done
R1(config-router)# end
```

```
R4# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R4(config)# router ospf 1
R4(config-router)# router-id 0.0.0.4
R4(config-router)# network 172.16.0.0 0.0.255.255 area 0
R4(config-router)# end
R4#
```
