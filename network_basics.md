# 1.0 Networks

*LAN (Local Area Networks)* - "physical" networks that provide the connection between devices within a physical location. 

*IP (Internet Protocol)* - layer that provides an abstraction for connecting multiple LANs into the internet. 

*TCP* - deals with transport and connections and sending and receiving user data

## 1.1 Layers

> LAN, IP, and TCP are often called **layers**, they make 3 layers: Link layer (LAN), Internet-work layer (IP) and the transport layer (TCP). Together with the application layer (the software used), it forms the ***4-layer*** model for networks.

- A given layer communicates directly only with the 2 layers above and below. An application hands off a chunk of data to the TCP library, which in turn makes calls to the IP library, which in turn calls the LAN layer for actual delivery. An application ***does not*** interact directly with the IP and LAN layers at all.

> *LAN Layer* is in charge of delivery of packets, using a LAN-layer-supplied addresses. The *Physical Layer* is generally of direct concern only to those designing LAN hardware; the kernel software interface to the LAN corresponds to the Logical LAN layer.

Application -> Transport -> IP -> Logical LAN -> Physical LAN

*Physical and Logical LAN division, gives us the Internet **5-layer model***

**In total there are 7 Layers.**

## 1.2 Data rate, Throughput and Bandwidth 

> Any network connection (eg LAN) has a *data rate*. Which is the rate which bits are transmitted.

- Some LANs like Wi-Fi data rate can vary with time. 
- *Throughput* - refers to the overall effective transmission rate, taking into account concepts like *transmission overhead*, *protocol inefficiencies* and *competing traffic*. It is generally measured at a higher network layer than data rate
- *Bandwidth* - can refer to either data rate or throughput; however it is used more for data rate. Comes from radio transmissions, where ***the width of the frequency band available is propotional, all else being equal, to the data rate that can be achieved***.

> **Goodput** is sometimes used to refer to what might also be called ***application-layer throughput*** - the amount of usable data delivered to the receiving application, specifically retransmitted data is counted only once when calculating goodput but might be counted twice under some interpretations of throughput.

Data rates are generally measured in ***Kilobits per second (kpbs) or megabits per second (Mbps)***. The use of lowercase *b* here denotes **bits**. 
- a kilobit is 10^3 bits 
- a megabit is 10^6 bits
- Upper case B denotes *bytes*. 
- Can also use Kib or Mib for kpbs or Mbps respectively.

## 1.3 Packets 

> ***Packets*** - modest-sized *buffers* of data, transmitted as a unit through some shared set of links. Packets need to be *prefixed* with a **header** containing delivery information. 

- In cases like ***Data forwarding***, the header contains a destination address.
- Headers in networks using ***virtual-circuit*** forwarding contain instead an identifier for the *connection*. Almost all networking today is packet-based. 

```Text
    # single and multiple headers
    header | data
    header1 | header2 | data 
```

> At the LAN Layer, packets can be viewed as the imposition of a buffer (and addressing) structure on top of low-level serial lines. Informally, packets are often referred to as ***frames*** at the LAN layer, and as ***segments*** at the transport layer.

- Max packet size supported by a given LAN ia an intrinsic attribute of that LAN. 
- Ethernet allows a max of 1500 bytes of data
- TCP/IP packets often held only 512 bytes of data
- Token Ring packets could contain up to 4kB of data
- ATM (Asynchronous Transfer Mode) protocol uses 48 bytes of data per packet

Potential Issue - how to forward packets from a large packet LAN to (or through) a small-packet LAN. **There is ways to address this on the IP Layer**.

Each layer adds its own header:
- Ethernet header - 14 bytes
- IP headers - 20 bytes 
- TCP headers 20 bytes

> If a TCP connection sends 512 bytes of data per packet, then the headers amount to 10% of the total overhead. 

- Example: Voice-over-IP option, packets contain 160 bytes of data and 54 bytes of headers, making the header about 25% of the total packet. Compressing the 160 bytes of audio, however, may bring the data portion down to 20 bytes, and the headers become then 73% of the total packet.

> In *data forwarding* networks the appropriate header will contain the address of the destination and sometimes other delivery information. Internal nodes of the network are called ***routers*** or ***Switches*** will then try to ensure that the packet is delivered to the requested information.

**Packets** are buffers built of ***8-bit*** bytes. Meaning 1 byte = 8 bits

**NOTE - Binary integers can be represented as a sequence of bytes in either *big-endian* or *little-endian* byte order. Internet protocols use big-endian byte order, thus being called network byte order.**

## 1.4 Datagram Forwarding

> In *datagram-forwarding model of packet delivery* packet headers contain the destination address. Switches and routers have to look at the address and route it to the correct destination. 

- *Datagram-forwarding* is achieved by providing each switch with a **forwarding table** of `<destination, next_hop>` pairs. 
- Switch looks up the destination address in forwarding table and finds *`next_hop`* information which is the immediate neighbor address or interface to which the packet should be forwarded to get closer to the final destination.
- `next_hop` value in a forwarding table is a single entry. 
- Each switch is responsible for only 1 step in the packet's path.
- If all good, a network of switches can delivery the packet on hop at a time.

> *Destination entries* in forwarding table do not have to correspond exactly with the packet destination addresses. However they do for **Ethernet datagram forwarding**. 

> For IP routing, the table *destination entries* will correspond to **prefixes** of IP addresses leading to saving in space.

It is fundamental that the switch can perform a lookup operation!, using its forwarding table and the destination address in the arriving packet, to determine the next hop. 

![[Screenshot 2025-04-22 at 6.46.21 PM.png]]

The following are the forwarding tables for each switch:

```text 
S1
destination | next_hop
A -> 0
C -> 1
B -> 2
D -> 2
E -> 2

S2
destination | next_hop
A -> 0
C -> 0
D -> 1
E -> 2
B -> 3
```

> All links are point-to-point, and so each interface corresponds to the unique immediate neighbor reached by that interface. Thus we can replace the interface entries in the `next_hop` column with the name of the corresponding *neighbor*.

```text
S1 
destination | next_hop
A -> A 
C -> C
B,C,E -> S2 #this means that to reach B,C,E we can to move to switch 2
```

**Stateless Forwarding** - A central feature of datagram forwarding where each packet is forwarded "in isolation". The switches involved do not have any awareness of higher-layer logical connections established between endpoints. And forwarding tables have no per-connection state (RFC 1122).
- To improve robustness of the communication system, gateways are designed to be stateless, forwarding each IP datagram independently of other datagrams. Thus, redundant paths can be exploited to provide robust service in spite of failures of intervening gateways and networks.

**Virtual Circuits** - Alternative to data forwarding. Each router maintains state about each connection passing through the router itself. Different connections can be routed differently. 
- If packet forwarding depends on per-connection information (example: both TCP port numbers) it is *not* considered datagram forwarding, unless its web traffic from TCP to TCP port 80, which is forwarded differently from all other traffic, because that rule does not depend on a specific connection.

> Datagram forwarding is sometimes allowed *to use other information beyond the destination address*. 
> IP routing can be done based on the destination address and some *quality-of-service* information, allowing, for example, different routing to the same destination for *high-bandwidth* bulk traffic and for *low-latency* real-time traffic.

- Most ISPs (Internet Service Providers) ignore user-provided quality-of-service information in the IP header, except by *prearranged agreement, and route only based on the destination*.

*Switching devices* acting at the LAN layer, and forwarding packets based on the LAN address are called **switches or bridges**, while such devices acting at the IP layer and forwarding on the IP address are called **routers**.
- Datagram forwarding is used both by *Ethernet switches and by IP Routers*.
	- Destinations in Ethernet forwarding tables are individual nodes
	- Destinations in IP routers are entire *Networks (set of nodes)*

> In IP routers within end-user sites it is common for a forwarding table to include a *catchall default entry*, matching any IP address that is nonlocal and so needs to be routed out into the internet at large.
> A *Default Entry* is a single record representing where to forward the packet if no other destination match is found. Meaning if destination is on another switch, then we have a default entry where the packet will be sent since switch 1 does not have destination address. 

```text 
S1 forwarding table with default entry
B,D,E destinations are still on S2 hence S1 forwarding table will have 
port 2 as next_hop since its the port with a connection to another switch 

S1
destination | next_hop
A -> 0
C -> 1 
default -> 2 
```

**Default entries make sense only when we can tell by looking at an address that it does not represent a nearby node.**
- This common in IP networks because an IP address encodes the destination network and routers generally know all the local networks.
- It is rare in Ethernets, because there is generally no correlation between ethernet addresses and locality. *Ethernet switches do not know what kind of device connects to a given interface*.

## 1.5 Topology 

Some network diagram's contain no loops, and are considered an *acyclic network graph, or a tree*. In a *loop-free* network, there is a unique path between any pair of nodes. The forwarding table algorithm has only to make sure that every destination appears in the forwarding tables: the issue of choosing between alternative paths does not arise.

**Acyclic graph** - No loops between any pair of nodes. Unique path to every node. To achieve maximum speed algorithm has to know the quickest path between 2 pairs of nodes with no looping, meaning that algorithm has to backtrack in path every time it discovers an endpoint that is not the desired node.

```text 
To find the quickest path between two nodes in an acyclic graph, an algorithm typically follows these steps:

1. **Traversal**: Use Depth-First Search (DFS) or Breadth-First Search (BFS) to explore the graph.

2. **Backtracking**: If a dead end is reached (i.e., a node with no further connections leading to the desired endpoint), the algorithm backtracks to explore alternative paths.

3. **Path Tracking**: Maintain a record of the paths explored to ensure that the algorithm can backtrack efficiently.
```

- If there are no loops then there is no *redundancy*, meaning any broken link will result in partitioning the network into pieces that cannot communicate. 
- Including redundancy will implicate the decision making among the multiple paths to a destination.

![[Screenshot 2025-04-24 at 11.17.24 PM.png]]

Both `A-S1-S2-S4-B` and `A-S1-S3-S4-B` get there, there is no right answer, even if one path is "faster" than the other, taking the slower path is not exactly wrong (especially if the slower path is less expensive). To decide in this options, many LAN's (in particular ethernet) prefer *tree* networks with no redundancy, while IP has complex protocols in support of redundancy.

### Traffic Engineering 
> Refers to any intentional selection of one router over another, or any elevation of the priority of one class of traffic. 
- The route selection can either be directly intentional, through configuration, or can be implicit in the selection or tuning of algorithms that then make these route-selection choices automatically.
- **`Distance-Vector Routing-Update Algorithm`** build forwarding tables on their own, but those tables are greatly influenced by the administrative assignment of link costs.

With *pure datagram forwarding*, used at either the LAN or the IP layer, the path taken by a packet is determined solely by its destination, and traffic engineering is limited to the choices made between alternative paths
- As quality of service information is taken into account in datagram forwarding, voice traffic for example with its relatively low bandwidth but intolerance for delay, take an entirely different path compared to bulk file transfers. Hence a network manager might take voice traffic with higher priority, so it does not have to wait in queues behind file transfer. **Some algorithms might use priority queues to add priority to the packet type**
- The quality of service information may be set by the end-user, which case an ISP may wish to recognize it only for designated users, meaning **ISPs will implicitly use the traffic source when making decisions**. 
- ISPs routing decision can be based on packet size, port number, and other contents.

At the LAN layer, *traffic engineering* mechanisms are historically limited. 
At the IP layer more strategies are available when taking in account *quality of service*.

## 1.6 Routing Loops

> **Possibility of routing loops is a drawback to datagram forwarding**.
> A set of entries in the forwarding tables that cause some packets to circulate endlessly.

Routing loops typically arise because the creation of the forwarding tables is often *distributed*, and there is no global authority to detect *inconsistencies*. Even when there is such an authority, temporary routing loops can be created due to notification delays.

> *Routing Loops* can also occur in networks where the underlying link topology is loop-free. *Linear routing loops* - 2 nodes looping back from one to another.

**All datagram-forwarding protocols need some way of detecting and avoiding routing loops** 
- Ethernet - avoids nonlinear routing loops by disallowing loops in the underlying network topology, and avoids linear routing loops by not having switches forward a packet back out the interface by which it arrived.
- IP - provides for a one-byte "Time to live" (TTL) field in the IP header; it is set by the sender and decremented by 1 at each router; and a packet is discarded when TTL reaches 0. This limits the number of times a wayward packet can be forwarded to the initial TTL value, which is typically *64*.

> In datagram forwarding, a *switch* is responsible only for the `next_hop` to the ultimate destination; if a switch has a complete path in mind, there is no guarantee that the `next_hop` switch or any other downstream switch will continue to forward along that path. 
- Misunderstandings can potentially lead to routing loops. Hence datagram forwarding requires cooperation and a consistent view of the network. 

![[Screenshot 2025-04-28 at 10.31.18 PM.png]]

**If D thinks best path to B is D-E-C-B, and E thinks the best route is E-D-A-B, then *Linear routing* happens because both paths are not followed and D and E will sent packet to each other**

## 1.7 Congestion

> Switches introduce the possibility of *congestion* - Packets arriving faster than they can be sent out. This can happen by:

- This can happen with just 2 interfaces:
	- If *inbound interface* has higher bandwidth than *outbound interface*.
- Traffic arriving on multiple inputs and destined for the same output

If packets are arriving for a given *outbound interface* faster than they can be sent, a *queue* will form for that interface. If *queue* gets full, packets will be dropped. 
Most common strategy is:
- drop any packets that arrive when the queue is full (However some packets that are arriving when queue is full can be more important than the ones already in the queue).

*Congestion* may refer either to: 
- the point where the queue is just beginning to build up - slope of the *load vs throughput graph flattens* (AKA *CONTENTION*)
- point where the queue is full and packets are lost - where packet losses may lead to a precipitous decline in throughput. 

**Most packet losses on the internet are due to congestion, because other packet losses like packet corruption are insignificant compared to congestion.**

*Congestion* - does not mean shortage of bandwidth. It can be seen as simply the *network's feedback* that the maximum transmission rate has been reached.

-> In networks losses due to congestion must be generally kept to an absolute minimum, and one way to achieve this is to limit the acceptance of new connections unless sufficient resources are available.

## 1.8 Packets *Again*

*Packets* are the key to supporting *shared transmission lines* - they support the *multiplexing* of multiple communications channels over a single cable.
The alternative of a separate physical line between every pair of machines grows prohibitively complex very quickly. Although *virtual circuits* between every pair of machines in a datacenter are not uncommon.

*Maximum packet size* - feature that represents the maximum time a sender can send before other senders get a chance.
*Unbounded packet sizes* would lead to prolonged network unavailability for everyone else if someone downloaded a large file in a single 1 Gigabit packet.
Another *drawback to large packets* is that, if packet corrupted, the entire packet must be retransmitted. 

> **Store-and-Forward** - when a router or switch receives a packet, and reads the entire packet before looking at the header to decide which node to forward it to. 
> **This introduces a *forwarding delay* equal to the time needed to read the entire packet.**
> For individual packets the forwarding delay is hard to avoid. However, some. switches implement *cut-through* switching.
> **Cut-through switching** - forwarding a packet before it has fully arrived. but if one is sending a long train of packets then by keeping multiple packets *en route* at the same time can essentially eliminate the significance of the forwarding delay.

**TOTAL PACKET DELAY FROM SENDER TO RECEIVER IS THE SUM OF THE FOLLOWING:**
1. **Bandwidth delay** - sending 1000 Bytes/millisecond will take 50 ms. This is a per-link delay
2. **Propagation delay** - due to speed of light. For Example, if we start sending a packet right now on a 5000KM cable across the US with a propagation speed of 200KM/ms, which is about 2/3 the speed of light in vacuum, the first bit will not arrive at the destination until 25 ms later. The bandwidth delay then determines how much after that the entire packer will take to arrive.
3. **Store-and-forward delay** - equal to the sum of the bandwidth delays out of each router along the path.
4. **Queueing delay** - waiting in line at busy routers. At bad moments this can exceed 1 second, though it is rare. Generally it takes less than 10ms and often is less than 1ms. Queuing delay is the only delay component amenable to reduction through engineering.

## 1.9 LAN and Ethernet

*LAN* - Local-area network, consists of:
- physical links that are ultimately, serial lines
- common interfacing hardware connecting the hosts to the links
- protocols to make everything work better

-> With cooperation of intermediate nodes acting as switches, every LAN node is able to communicate with every other LAN node.

*Collision* - when 2 stations transmit data over the ethernet at the same time, rendering the data unintelligible.

Ethernet feature intended to minimize the bandwidth wasted on collisions:
- (**The Slot Time and Collisions**) - stations before transmitting, check to be sure the line is idle, they monitor the line while transmitting to detect collisions during the transmission, and if a collision is detected, they execute a *random backoff strategy* to avoid immediate *recollision*.
- Ethernet collisions reduce throughput, hence being seen as a remarkably inexpensive *shared access mediation protocol*


> Unswitched Ethernets, every *packet* is received by every hosts.
> Network card in each host determines if arriving packet is address to that host
> Threat posed - **Password Sniffing** due to the option of NIC forwarding all arriving packets to the host.
  
All Ethernets are switched nowadays, ensures that packet is delivered only to the host to which it is addressed.

Advantage of *Switching*;
- Eliminates most ethernet collisions, while in principle it replaces them with a *queueing* issue leading to *congestion*
- Also prevents host-based eavesdropping, however encryption is a better solution to this problem.

**Ethernet addresses are 6 bytes long**
-> Each NIC is assigned a unique address at the time of manufacturing. It is burned in to NICs ROM (read-only memory) and is considered the MAC (Media Access Control) address of the NIC.
- First 3 Bytes - Unique MAC address of NIC
- Second 3 Bytes - serial number assigned by the manufacturer
  
**IP Addresses are assigned administratively by the local site**.

Addresses on hardware Advantage:
- hosts automatically know their own addresses on startup, not needing manual configuration or server query.
  
> Network interface continually monitors all arriving packets, if packet has destination address the physical address of NIC, grabs packet and forwards it to CPU (via CPU interrupt).

*Broadcast Address* - host sending to the broadcast address has its packet received by every other host on the network. If a switch receives a broadcast packet on one port, it forwards the packet out every other port. This allows host A to contact host B when A does not yet know host B's physical address. Broadcast queries have forms of:
- ARP Protocol (`arp -a`)
  
**Unicast traffic** - traffic addressed to a particular host that is not broadcast.

> Switched Ethernet, the *switches* must have a *forwarding table* record for each individual Ethernet address on the network, however for super large networks this is difficult to carry. Works well for networks between 10,000 to 100,000 nodes. This allows *forwarding tables* with size in the range to be straightforward to manage.

-> Switches must know where all active destination address in the LAN are located to forward packets correctly.
- Traditional Ethernet Switches use **passive learning algorithm**
- IP Routers use *"active" protocols*
- Newer Ethernet switches use **Software defined networking**

  
> -> Host's physical address is entered into a switch's forwarding table when a packet from that host is first received.
> -> Switch notes the packet's arrival interface and *source* address and assumes that the same interface is to be used to deliver packets back to that sender.
> -> If a given destination address has not been seen yet, hence not being in forwarding table
> -> Ethernet switches still have the backup delivery option of *flooding*
> -> **Flooding** - forwarding the packet to everyone by treating the destination address like the broadcast address, and allowing the host NIC to sort it out.
> -> Since flooding process is not generally used for more than one packet (switch after one packet will store correct forwarding table entry), risk of excessive traffic and eavesdropping is minimal

`<host, interface>` forwarding table is the same as `<host, next_hop>`, where `next_hop` node is whatever switch or host is at the immediate other end of the link connecting to the given interface.
- In fully switched network where each link connects only 2 interfaces, `<host, interface> and <host, nex_hop>` are equivalent.
## 1.10 IP - Internet Protocol

-> Used to solve the Ethernet scaling problem, and to allow support for other types of LANs and point-to-point links.
-> Central issue in designing IP was to support universal connectivity, in such a way as to allow scaling to enormous size (everyone can connect to everyone).

To support universal connectivity IP provides:
- global mechanism for **addressing and routing**, so that packets can actually be delivered from any host to any other host.

IP Addresses:
**IPv4** - 4 bytes (32 bits) and are a part of the *IP Header* that generally follows the Ethernet header.
**IPv6** - 16 bytes (128 bits) also part of the IP header that generally follows the ethernet header

**Ethernet header** only stays with a packet for *one hop*
**IP header** - stays with the packet for entire journey across the internet.
  
**IPv4 and IPv6** addresses has a feature of being able to be divided into a *network* part (prefix) and a host part (remainder).

==*IP addresses* are administratively assigned.==

IPv4 Legacy mechanism for designating IP addresses

```

first few bits | first byte | network bits | host bits | name | application
0 | 0-127 | 8 bits | 24 bits | class A | few very large networks
10 | 128-191 | 16 bits | 16 bits | class B | institution-sized networks
110 | 193-223 | 24 bits | 8 bits | class C | sized for smaller entities
1110 | 224-239 | MULTICAST ADDRESS

```

If a university IP address is 147.126.0.0 - class B, first few bits are based on 147 (10010011) where `10` are 2 first bits
IP addresses usually serves not just as an *endpoint identifier* but also a *locator*, containing embedded location information based on IP hierarchy class and not geolocation.
Ethernet addresses are *endpoint identifiers* but *not locators*.

**Class D** addresses are for *multicast* - sending an IP packet to every member of a set of recipients without actually transmitting more than once on any one link.
  
Nowadays the division into the *network and host bits* is *dynamic* and can be made at different positions in the address at different levels of the network.
/27 address block (1/8 the size of a class C /24) from its ISP (200.1.130.96/27)
Cloud Ninjas address is (50.246.50.177) - ISP host IP address and cloud ninjas gets a prefix like (/27 - example)
At some higher level, routing might be based on the prefix (/18), which represent and *address block* assigned to the ISP.

The network host division point is *not* carried within the IP header; routers negotiate this division point when they negotiate the `next_hop` forwarding information.

**This is based on Classless IP delivery algorithm**
> Network portion of an IP address is called `network number, network address, or network prefix`
> **Forwarding decision are made using only the network prefix**.
> **Network prefix** - denoted by setting the host bits to 0 and ending the resultant address with a slash followed by the number of network bits in the address.

12.0.0.0/8 - class A, network bits are 8 bits host bits are 24
147.126.0.0/16 - class B, network bits are 16 bits and host bits are 16 bits

> All hosts with the same network address (same network bits) are said to be on the same *IP Network* and must be located together on the same LAN.
> If 2 hosts share the same network address then they will assume they can reach each other directly via underlying LAN, and if they cannot then connectivity fails. A consequence of this rule is that outside of the site *only network bits need to be looked at to route a a packet to the site*

-> All hosts/ All interfaces on the same physical LAN share the same network prefix and thus are part of the same IP network; occasionally, one LAN is divided into multiple IP networks.

Each individual LAN technology has a **maximum packet size** it supports:
- Ethernet 1500 bytes
- Token Ring 4 kB (kilobytes)

In addition to routing and addressing, IP must also support *fragmentation*.
- *Fragmentation* - The division of large packets into multiple smaller ones.

*IP is a best effort system* - There are no IP-layer acknowledgments or retransmissions. Packet is shipped off and hoped to land at destination.

> Best effort models represents what is known as *connectionless networking*.
> **Connectionless Networking** - IP-layer does not maintain information about endpoint-to-endpoint connections, and simply forwards the packets like a giant LAN.

> Alternative to connectionless networking -> *connection-oriented internetworking*, in which routers do maintain state information about individual connections. **Virtual Circuits** can be used to implement a connection-oriented approach, virtual-circuit switching is the primary alternative to datagram switching.

-> The responsibility for creating and maintaining connections is left for the *TCP Layer* (layer up).
- Connectionless networking Advantages:
- more reliable
- routers do not hold connection state, then they cannot lose connection state
- path taken by packets in some higher-level connection can easily be dynamically rerouted
- Makes it hard for providers to bill by connection
- Connection oriented networking Advantages:
- routers are then much better positioned to accept *reservations* and to make *quality-of-service guarantees*
- Connection oriented networking Disadvantage:
- If using Voice-over-IP or VoIP, telephones, video conferencing, the packets in this model will be treated by the internet core just the same as if they were low priority file transfers. There is no "priority service" option.

Most common form of IP packet loss are:
- router queue overflows, representing network congestion
- packet corruption (rare)

**In connectionless networking a large number of hosts can simultaneously attempt to send traffic through one router, in which case queue overflows are hard to avoid**
### 1.10.1 IP Forwarding

> *IP routers use datagram forwarding* - "destination" values listed in forwarding tables are *network prefixes* - representing entire LANs instead of individual hosts.
> Goal of *IP Forwarding*, then becomes delivery of packets to the correct LAN, then a separate process is used to deliver to the final host once the final LAN has been reached.

-> Entire point of having a *network/host* division within IP addresses is so that *routers need to list only the network prefixes* of the destination addresses in their IP forwarding tables, which is the key to *IP scalability*, and saves large amounts of forwarding-table space, and faster lookup.

-> With IP's use of network prefixes as forwarding table destinations, matching an actual packet address to a forwarding table entry is no long a mater of simple equality comparison; routers must compare appropriate prefixes.
#### How IP Forwarding (Routing) Works

> Assuming all nodes are either hosts or routers (which do packet-forwarding only).
> - Routers are not directly visible to users, and always have at least 2 different network interfaces representing different networks that the router is connecting.

If `A` is a sending host, sending a packet to host `D`, the IP header of the packet will contain both `A and D` IP address as source and destination address.
1. `A` has to find out if `D` is on the same LAN as itself (whether `D` is local). This is done by checking the network part of `D's` address. If the same as `A`, then source host (`A`) can use direct LAN delivery to `D`. It uses *ARP (Address Resolution Protocol)* to lookup physical address of `D`, and attaches LAN header to the packet in front of the IP header, and sends the packet straight to `D` via the LAN.
2. If `A` network `!==` to `D` network, `A` looks up a router to use. Most hosts use only one router for all non-local packet deliveries. `A` then forwards the packet to the router, using direct delivery over LAN. The IP destination address in the packet remains `D`, although the LAN destination address will be that of the router.
When router receives packet, strips off the LAN header but leaves IP header with the IP destination address. Extracts the destination `D`, and looks up at `D's` network. Router first checks to see if any of its *network interfaces* are on the same LAN as D; router connects to at least one additional network besides the one for `A`. If it is in the same LAN as the router than router delivers directly via LAN to destination `D`. If `D` is not on the same network as the router, then the router consults its internal forwarding table.
Which consists of networks associated with the router on the `next_hop` address. These `<net, next_hop>` tables compare with *switched-Ethernet's* `<host, next_hop>` tables; the former type will be smaller because there are many fewer nets than hosts. The *`next_hop`* addresses in the table are chosen so that the router can always reach them via direct LAN delivery via one of its interfaces, which generally are other routers.
Router looks up `D net` in the table, finds the `next_hop` address, and uses direct LAN delivery to get the packet to that `next_hop` machine.
The packet's IP header remains essentially the same, although router most likely attaches an entirely new LAN header.
The packet continues being forward this way, from router to router until it arrives at a router that is connected to `D net`, it is then delivered by that final router directly to host D using LAN.

Diagram from sending packet from A to D through multiple routers:
![[Screenshot 2025-05-02 105857.png]]

IP addresses for the ends of R1-R2 and R2-R3 links are not shown, they could be assigned global IP addresses or they could be "private" IP addresses. Assuming these links are point-to-point, they might not actually need IP addresses at all.

> Newer protocols that support different net/host division points at different places in the network - sometimes called **hierarchical routing** - allow support for addressing schemes that correspond to addresses done like `zip/street/user`.  

*Internet Backbone* - can be referred as those IP routers that specialize in large-scale routing on the commercial internet, and which generally have forwarding-table entries covering all public IP addresses. Routers that do not have a default entry.

IP routers at non-backbone sites generally know all locally assigned network prefixes. If a destination does not match any locally assigned network prefix, the packet needs to be routed out into the internet at large.
- This often means the packet is sent to the ISP that provides Internet connectivity.

Local routers will contain a catchall *default* entry covering all nonlocal networks, meaning that the router needs an explicit entry only for locally assigned networks. This reduces the forwarding-table size.

IP Routers do *not* have a "flooding" delivery mechanism as a fallback, so the tables must be constructed in advance. (There is a limited form of IP broadcast, but it is basically intended for reaching the local LAN only, and does not help at all with delivery in the event that the destination network is unknown).  

> Most forwarding-table construction algorithms on a set of routers under common management fall into either the *distance-vector* or the *link-state* category.
> Routers not under common management - neighboring routers belonging to different organizations - exchange information through the BGP (Border Gateway Protocol).
> *BGP* - allows routing decisions to be based on a fusion of "technical information" (which are sites reachable at all, and through where) together with "policy" information representing legal or commercial agreements; in which outside routers are "preferred", whose traffic an ISP will carry even it is is not to one of the ISP's customers, etc...

NOTE: most common residential "routers" involve *network address translation* in addition to packet forwarding

### 1.10.2 Future of IPv4

> Allocation of blocks of IP addresses is the responsibility of the IANA (Internet Assigned Numbers Authority). Now allocating network prefixes is the job of individual sites, and IANA limited themselves to handing out only `/8` blocks (Class A blocks) to the 5 *regional registries*:
1. ARIN - North America
2. RIPE - Europe, Middle East and parts of Asia
3. APNIC - East Asia and the Pacific
4. AfriNIC - most of Africa
5. LACNIC - Central and South America

`/8` blocks have ran out !

New solution is the adoption of IPv6 addresses (128 bit addresses)

## 1.11 DNS (Domain Name System)

> DNS helps creating a way to convert hierarchical text names to IP addresses.
> Virtually all Internet Software uses the same basic library calls to convert DNS to actual addresses.

*DNS makes it possible to change a website's IP address while leaving the name alone.*
- Different DNS names can resolve to the same IP address, and through some tinkering we can have the HTTP (web) server at that IP address handle the different DNS names as completely different websites.
- DNS is hierarchical and distributed

*DNS root zone* - looking up a domain like uh.edu, 4 DNS servers can be queried, for `edu, uh.edu, cs.uh.edu`. Searching a hierarchy can be cumbersome, so DNS search results are normally cached locally. If a name is not found in cache, the look up may take a couple seconds. The DNS hierarchy need have nothing to do with the IP-address hierarchy.
## 1.12 Transport

IP layer gets packets from one node to another, but it is not well suited for transport since:
- IP routing is a "best effort" mechanism, meaning packets can and do get lost sometimes
- Data that does arrive can arrive out of order.
- IP only supports sending to a specific host, normally, one wants to send a given application running on that host.
- Email and Web traffic, or 2 different web sessions, should not be mixed

> The **transport layer** is the layer above the IP layer that handles these sort of issues, often by creating some sort of *connection abstraction*.
> Most popular transport mechanism is **TCP (Transmission Control Protocol)**.
> *TCP* extends IP with the following features:
1. **Reliability** - TCP numbers each packet and keeps track of which are lost and retransmit them after a timeout. It holds early-arriving out of order packers for delivery at the correct time. Every arriving data packet is acknowledged by the receiver, timeout and retransmission occurs when an acknowledgement packet is not received by the sender within a given time.
2. **Connection Orientation** - Once a TCP connection is made, an application sends data simply by writing to that connection. No further application-level addressing is needed. TCP connections are managed by the *Operating System kernel*, not by the application.
3. **Stream Orientation** - An application using TCP can write 1 byte at a time, or 100kB at a time; TCP will buffer and/or divide up the data into appropriate sized packets.
4. **Port Numbers** - These provide a way to specify the receiving application for the data, and also to identify the sending application.
5. **throughput Management** - TCP attempts to maximize throughput, while at the same time not contributing unnecessarily to network *congestion*

> TCP endpoints are of the form `<host, port>`; these pairs are known as **Socket addresses**, or sometimes as just *sockets*. 
> Sockets - operating systems objects that receive the data sent to the socket addresses.
> *Servers (Server applications)* - *listen* for connections to sockets they have opened; the *client* is then any endpoint that initiates a connection to a server.

- When we enter host name is web browser, a TCP connection to the server's 80 port, (server socket with socket-address 80) `<server, 80>` .
- If we have several tabs open, each might connect to the same server socket, but the connections are distinguishable by virtue of using separate ports (having separate *socket addresses*) on the *client end (our end)*
- A busy server may have thousands of connections to its port 80 and hundreds of connections to port 25 *(the mail port)*. 

-> A TCP connection is determined by the `<host, port>` socket address at each end; traffic on different connections does not intermingle.
- TCP uses the *sliding-windows algorithm* to keep multiple packets en route at any one time. The *windows size* represents the number of packets simultaneously in transit (TCP actually keeps track of the window size in bytes, but packets are easier to visualize).
- If window size is 10 packets, than at any one time 10 packets are in transit (5 data packets and 5 returning acknowledgements).
- Assuming no packets are lost, then as each acknowledgement arrives, the window "slides forward" by one packet. The data packet 10 packets ahead is sent, to maintain a total of 10 packets in transit. 
- When the 10 packets 20-29 are circulating, once the *ack packet* for 20 is received, the total number of packets becomes 9, if window is 10, then packet 30 is sent to keep window at 10 packets, once 21 *Ack packet* is received packet 31 is sent.

*Sliding Windows* - minimizes the effect of store-and-forward delays, and propagation delays, as these only count once for the entire windowful and not once per packet.
- Also provides an automatic, if partial, brake on *congestion*.
- *Constant-rate transmission* - if available bandwidth falls below transmission rate, always leads to overflowing queues and significant percentage of dropped packets.
- If window size too large, a sliding windows sender may also experience dropped packets.
- *ideal window size*:
	- from throughput perspective - takes one round-trip to send an entire window, so that the next *ACK* will always be arriving just as the sender has finished transmitting the window.
	- ideal size varies with network load
	- TCP approximates the ideal size, via strategies like **TCP RENO**.

*TCP Reno* - The window size is slowly raised until packet loss occurs. Which TCP takes as a sign that it has reached the limit of available network resources. At that point the window size is reduced to half its previous value, and the slow climb resumes.
- Attempts to maximize the available bandwidth
- greatly limits the number of packet loss events

TCP is charged by the Internet Protocol (IP) with: 
- Manages congestion on the internet 
- Ensuring fairness of bandwidth allocations to competing connections.

>**head-of-line blocking**  - Real time performance of TCP is not always consistent; if a packet is lost, the receiving TCP host will not turn over anything further to the receiving application until the lost packet has been retransmitted successfully.
- Serious problem for sound and video applications, which can discreetly handle modest losses but which have much more difficulty with sudden large delays.
- A few lost packets ideally should mean just a few brief voice dropouts or flicker/snow on the video screen - better than sudden pause.

Alternative to TCP is **UDP (User Datagram Protocol**:
What it does:
- Provides port numbers to support delivery to multiple endpoints within the receiving host, in effect to a specific process on the host.
- *UDP* sockets consists of a `<host, port>` pair.
- *UDP* also includes a *checksum* over the data.

What doesn't do: 
- No connection setup 
- No lost-packet detection
- No automatic timeout/retransmission
- Application must manage its own packetization

**RTP (Real-Time Transport Protocol)** - sits above UDP and adds some additional support for voice and video applications.

### 1.12.1 Transport Communications Patterns

> 2 Classic traffic patterns for Internet Connection are these:
> - Interactive or bursty communications via *ssh or telnet*, with long idle times between short bursts
> - Bulk file transfers, such as downloading a webpage.

TCP's congestion management features only applies only when a large amount of data is in transit at once. Web browsing is hybrid, over time, there is usually considerable burstiness, but individual pages now often exceed 1MB.
- We might add *request/reply* operations, example, to query a database or to make DNS requests
- Most DNS traffic still uses UDP. There periodic calls for a new protocol specifically addressing this pattern, though at this point the use of TCP is well established.
- If a *sequence* of *request/reply* operations is envisioned, a single TCP connection is best, as the connection-setup overhead is minimal by comparison. 

> TCP generally works well with video streaming. assuming the receiver can get, for example, a minute ahead, buffering the video that has been received but not yet viewed. That way if there is a dip in throughput due to congestion, the received has time to recover - *Buffering*
> Another issue with video streaming is the bandwidth demand. Most streaming-video services attempt to estimate the available throughput, and then *adapt* to that throughput by changing the video resolution. 
> Typically, video streaming operates on a *start/stop* basis: the sender pauses when the receiver's playback buffer is "full", and resumes when the playback buffer drops below a certain threshold. 
> If video and voice audio are *interactive*, there is much less opportunity for stream buffering. If someone asks a simple question on an Internet telephone call, they generally want an answer more or less immediately; they do no not expect to wait for the answer to make it through the other party's stream buffer.
> *UDP* is often used to avoid *head-of-line blocking*. Lower bandwidth helps; voice-grade communications traditionally need only 8 kB/s, less if compression is used. On the other hand, there may be constraints on the *variation* in delivery time. 
> Interactive video, with its much bigger bandwidth requirements, is more difficult fortunately, users seem to tolerate the common pauses and freezes. 

With the *Transport Layer*, essentially all network connections involve a *client* and a *server*. This pattern is also repeated at the *Application Layer*.
The client contacts the server and initiates a login session, or browses some webpages, or watches a movie.
Sometimes, however, application layer exchanges fit the *peer-to-peer* model better, in which the 2 endpoints are more or less co-equals. For example:
- Internet Telephony - there is no benefit in designating the party who place the call as the "client"
- Message passing in a CPU cluster is often using *RFC (Remote Procedure Call)*
- The routing-communication protocols of *routing update algorithms*. When router `A` reports to router `B` we might call `A` the client, but overtime, as `A and B` report to one another repeatedly, the *peer-to-peer* model makes more sense.
- So-called *Peer-to-Peer* file sharing, where individuals exchange file with other individuals (and as opposed to "cloud-based" file sharing in which the "cloud" is the server)

### 1.12.2 Content-Distribution Networks (CDN)

Sites with an extremely large volume of content to distribute often turn to a *specialized communication pattern* called **Content-distribution Network (CDN)**. 
- To reduce the amount of long distance traffic, or to reduce the round-trip time, a site replicates its content at multiple datacenters (*points of presence (PoPs)*, nodes, access points or edge servers).
- When a user makes a request, the request is routed to the nearest datacenter, and the content is delivered from there.

Large webpages normally have:
- static content (videos, JavaScript)
- dynamic content

The CDN may cache all or most of the static content at each of its edge servers, leaving dynamic content to come from a centralized server. Alternatively, the dynamic content may be replicated at each CDN edge node as well, but it introduces some real-time coordination issues.

Dynamic Content:
- Not replicated, CDN may include private high-speed links between its nodes, allowing for rapid low-congestion delivery to any node.
- CDN nodes may simply communicate using the public internet. 
- CDN may or may not be configured to support fast *interactive* traffic between nodes, example, *teleconferencing traffic*.

Organizations can create their own CDNs, but often turn to specialized CDN providers, who often combine their CDN services with website-hosting services.

To create a CDN it is needed a multiplicity of datacenters, each with its own connection to the internet:
- Private links between datacenters are also common
- Many CDN providers also try to build direct connections with the ISPs that server their customers.
- Google Edge Network does this.
- Can improve performance and reduce traffic costs.

3 Techniques to find the closest edge servers:
1. Edge servers are all given different IP addresses and DNS is configured to have users receive the IP address of the closest edge server. 
2. Each edge server has the *same* IP address, and *anycast* routing is used to route traffic from the user to the closest edge server **(BGP and Anycast).**
3. HTTP applications, a centralized server can look up the approximate location of the user, and them redirect the web page to the closest edge server.

### 1.13 FIREWALLS

One problem with having a program on our machine listening on a open TCP port is that someone may connect and then, using some flaw in the software in our end, do something malicious to the machine. Damage from personal data download, machine takeover, virus and worm distributor, etc...

*Buffer Overflow* - identify a point in a server program where it fills a memory buffer with network supplied data without careful length checking; almost any call to the C library function `gets(buf)` will suffice. 

*Firewall* - mechanism to block connections deemed potentially risky. Generally ordinary workstations do not ever need to accept connections from the internet; client machines instead *initiate* connections to (better-protected) servers. So blocking incoming connections works.
- When necessary some ports can be selectively unblocked mainly for video games.

Original firewalls - built into routers
Nowadays - per-host firewalls or router-based firewalls -  we can configure your workstation not to accept inbound connections to most (or all) ports regardless of whether software on the workstation requests such a connection. Outbound connections can, in many cases, also be prevented.

Typical home router implements something called *network address translation*, which in addition to conserving IPv4 addresses, also provides firewall protection.

### 1.14 Useful utilities

*ping* - useful to determine if another machine is successfully accessible

```Text
ping www.cloudninjas.com 
ping 50.246.50.177 
```

*ifconfig, ipconfig, ip* - To find our own IP address we can use the following commands based on the operating system we use. 
*ifconfig* - Linux and MAC OS
*ipconfig* - Windows
*ip addr list* - Linux

The output generally lists all active interfaces but can be restricted to selected interfaces
*ip* command in particularly can do many other things

Windows command `netsh interface ip show config` - also provides IP addresses

*nslookup, dig and host* - Used for DNS lookups. They differ in convenience and options

*traceroute* - lists the route from you to a remote host (Linux Mac OS)
*tracert* - equivalent to traceroute for windows

> *Traceroute*  sends, by default 3 probes for each router. Sometimes responses do not all come back from the same router. 
> On Linux systems the *mtr* command may be available as an alternative to traceroute; it repeats the traceroute at 1 second intervals and generates cumulative statistics


*route & netstat* 
`route, route pring (windows), ip route show (Linux), and netstat -r (all systems)` - display the host's local IP forwarding table. For workstations not acting as routers, this includes the route to the default router.
The default route is sometimes listed as destination `0.0.0.0` with netmask `0.0.0.0 (equivalent to 0.0.0.0/0)`.

*netstat -a* - shows the existing TCP connections and open UDP sockets

*netcat* - allows the user to create TCP or UDP connections and send lines of text back and forth 

*tcpdump* - command line only packet capture program

*Wireshark* - packet capture and decoding program

Both `tcpdump` and `wireshark` support both live packet capture and reading from `.pcap (packet capture) and .pcapng (next generation) files`.