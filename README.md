# Multicast Lab – GNS3 Topology
This project is a basic GNS3 topology to demonstrate multicast communication using PIM Sparse Mode.

## What is Multicast?
Multicast is a way of sending a single stream of data to multiple devices at once without sending multiple separate copies. It’s often used in streaming, video conferencing, and real-time data feeds. 
Unlike broadcast, multicast traffic only goes to devices that are part of the multicast group.


The followint topology will be used:
![Topology](/home/munia/Multicast-Routing/Topology/Multicast.PNG)


Multicast uses the following protocols:
-  IGMP (layer 2 multicast)
-  PIM (layer 3 multicast)


## IGMP 
IGMP is the protocol used by devices (hosts or routers simulating hosts) to tell the local router that they want to join a multicast group.

For example, if a host wants to receive data sent to 239.1.1.10, it sends an IGMP join request.

Routers use this info to know which interfaces have active receivers.

IGMP can explictly call a source from which it will receive a multicast stream (Source specific Multicast)

To verify IGMP:

``` bash

R5#sh ip igmp interface e0/0
Ethernet0/0 is up, line protocol is up
  ! Output omitted for brevity
  Multicast designated router (DR) is 172.19.1.1 (this system)
  IGMP querying router is 172.19.1.1 (this system)
  Multicast groups joined by this system (number of users):
      239.1.1.10(1)  224.0.1.40(1)

```

``` bash
R5#sh ip igmp groups
IGMP Connected Group Membership
Group Address    Interface                Uptime    Expires   Last Reporter   Group Accounted
239.1.1.10       Ethernet0/0              00:08:51  00:02:11  172.19.1.1

```



## PIM Sparse Mode (Protocol Independent Multicast)

PIM is the routing protocol used to build multicast distribution trees between sources and receivers.


In sparse mode, multicast traffic is not forwarded by default. It’s only sent when receivers explicitly request to join a group through IGMP ("pull method").

A central point called the Rendezvous Point (RP) is manually configured in this lab. It acts as a meeting place for sources and receivers.

Once routers learn about active receivers, they build a multicast tree from the source to the receivers via the RP.

PIM-SM is used by routers to build multicast distribution trees.

When a source starts sending multicast traffic, it is forwarded to the Rendezvous Point (RP) first — this forms the shared tree (RPT).

Receivers also join via the RP, and traffic flows from source → RP → receiver.


### Switching to a Better Path (Shortest-Path Tree)

After initial traffic is received via the RP, routers closer to the receivers can switch to a better path directly to the source (using the configured dynamic routing).

This is called moving from the Shared Tree (via RP) to the Shortest-Path Tree (SPT).

When this happens, the router sends a prune message upstream toward the RP, telling it to stop forwarding traffic for that group down the shared tree.

This reduces unnecessary hops and improves efficiency



To verify the PIM RP configured:

``` bash
R5#sh ip pim rp
Group: 239.1.1.10, RP: 192.168.0.1, uptime 00:15:01, expires never
Group: 224.0.1.40, RP: 192.168.0.1, uptime 00:15:01, expires never

```

To verify Multicast routing table (S,G) the shortest path tree:

``` bash
R5#sh ip mroute
!output ommitted for brevity!

(*, 239.1.1.10), 00:17:01/stopped, RP 192.168.0.1, flags: SJCLF
  Incoming interface: Ethernet0/3, RPF nbr 10.0.0.17
  Outgoing interface list:
    Ethernet0/0, Forward/Sparse, 00:16:59/00:02:03

(172.19.1.1, 239.1.1.10), 00:02:03/00:03:26, flags: LFT
  Incoming interface: Ethernet0/0, RPF nbr 0.0.0.0, Registering
  Outgoing interface list:
    Ethernet0/3, Forward/Sparse, 00:02:03/00:03:23

(10.0.0.18, 239.1.1.10), 00:03:07/00:02:56, flags: LFT
  Incoming interface: Ethernet0/3, RPF nbr 0.0.0.0
  Outgoing interface list:
    Ethernet0/0, Forward/Sparse, 00:03:07/00:02:03

(*, 224.0.1.40), 00:17:00/00:02:06, RP 192.168.0.1, flags: SJCL
  Incoming interface: Ethernet0/3, RPF nbr 10.0.0.17
  Outgoing interface list:
    Ethernet0/0, Forward/Sparse, 00:16:59/00:02:06
```


