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

## 1.9 LANs and Ethernet
