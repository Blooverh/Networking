# 1.0 Networks

*LAN (Local Area Networks)* - "physical" networks that provide the connection between devices within a physical location. 

*IP (Internet Protocol)* - layer that provides an abstraction for connecting multiple LANs into the internet. 

*TCP* - deals with transport and connections and sending and receiving user data

## 1.1 Layers

> LAN, IP, and TCP are often called **layers**, they make 3 layers: Link layer (LAN), Internet-work layer (IP) and the transport layer (TCP). Together with the application layer (the software used), it forms the ***4-layer*** model for networks.

- A given layer communicates directly only with the 2 layers above and below. An application hands off a chunck of data to the TCP library, which in turn makes cakks to the IP library, which in turn calls the LAN layer for actual delivery. An application ***does not*** interact directly with the IP and LAN layers at all.

> *LAN Layer* is in charge of delivery of packets, using a LAN-layer-supplied addresses. The *Physical Layer* is generally of direct concern only to those designing LAN hardware; the kernel software interface to the LAN corresponds to the Logical LAN layer.

Application -> Transport -> IP -> Logical LAN -> Physical LAN

*Physical and Logical LAN division, giver us the Internet **5-layer model***

**In total there are 7 Layers.**

## 1.2 Data rate, Throughput and Bandwidth 

> Any network connection (eg LAN) has a *data rate*. Which is the rate which bits are transmitted.

- Some LANs like Wi-Fi data rate can vary with time. 
- *Throughput* - refers to the overall effective transmission rate, taking into account concepts like *transmission overhead*, *protocol inefficiencies* and *competing traffic*. It is generally measured at a higher network layer than data rate
- *Bandwidth* - can refer to either data rate or throughput; however it is used more for data rate. Comes from radio transmissions, where ***the width of the frequency band available is propotional, all else being equal, to the data rate that can be achieved***.

> **Goodput** is sometimes used to refer to what might also be called ***application-layer throughput*** - the amount of usable data delivered to the receiving application, specifically retransmitted data is counted only once when calculatin goodput but might be counted twice under some interpretations of throughput.

Data rates are generally measured in ***Kilobits per second (kpbs) or megabits per second (Mbps)***. The use of lowercase *b* here denotes **bits**. 
- a kilobit is 10^3 bits 
- a megabit is 10^6 bits
- Upper case B denotes *bytes*. 
- Can also use Kib or Mib for kpbs or Mbps respectively.

## 1.3 Packets 

> ***Packets*** - modest-sized *buffers* of data, transmitted as a unite through some shared set of links. Packets need to be *prefixed* with a **header** containing delivery information. 

- In cases like ***Data forwarding***, the header contains a destination address.
- Headers in networks using ***virtual-circuit*** forwading contain instead an identifier for the *connection*. Almost all networking today is packet-based. 

```Text
    # single and multiple headers
    header | data
    header1 | header2 | data 
```

> At the LAN Layer, packets can be viewed as the imposition of a buffer (and addressing) structure on top of low-level serial lines. Informally, packets are often referred to as ***frames*** at the LAN layer, and as ***segments*** at the transport layer.

- Max packet size supported by a given LAN ia an intrinsic attribute of that LAN. 
- Ethernet allows a max of 1500 bytes of data
- TCP/IP packets often held only 512b bytes of data
- Token Ring packets could contain up to 4kB of data
- ATM (Asynchronous Transfer Mode) protocol uses 48 bytes of data per packet

Potential Issue - how to forward packets from a large packet LAN to (or through) a small-packet LAN. **There is ways to address this on the IP Layer**.

Each layer adds its own header:
- Ethernet header - 14 bytes
- IP headers - 20 bytes 
- TCP headers 20 bytes

> If a TCP connection sends 512 bytes of data per packet, then the headers amount to 10% of the total overhead. 

- Example: Voice-over-IP option, packets contain 160 bytes of data anf 54 bytes of headers, making the header about 25% of the total packet. Compressing the 160 bytes of audio, however, may bring the data portion down to 20 bytes, and the headers become then 73% of the total packet.

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

It is fundamental that the switch can perform a lookup operation, using its forwarding table and the destination address in the arriving packet, to determine the next hop. 