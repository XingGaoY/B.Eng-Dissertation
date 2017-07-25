# Introduction

## Layering

- Four layers of TCP/IP protocol suite
  - link layer: or data-link layer or network layer, include device driver in the os and the corresponding NIC, handle all the hardware details of physically interfacing with the cable.
  - network layer: or internet layer, handles the movement of packets around the network, routing of packets takes place here.
  - transport layer: provides a flow of data between two hosts. 
    - TCP provides reliable flow of data between two hosts. It is concerned with things such as dividing the data passed to it from the application into appropriately sized chunks for the network layer below, acknowledging received packets, setting timeouts to make certain the other end acknowledegs packets that are sent, etc.
    - UDP provides a simpler service, sends packets of data called datagrams from one host to the other, but there is no guarantee that data grams reach the other end.
  - application layer: handles the details of the particular application.
  - Purpose: network layer--handles the details of the communication media; application layer--handles one specific user application.

- The easiest way to build an internet is to connect two or more networks with **router**, which provide connections to many different types of physical networks: Ethernet, token ring, point-to-point links, FDDI, etc.

- Protocol difference: the app layer and trans layer use end-to-end protocols, net layer provides hop-by-hop protocol and used on the rwo end systems ane every intermediate system.

- In the TCP/IP:
  - IP provides an **unreliable** service, does its best job of moving a packet from source to destination, but no guarantees. Datagram is a unit of information that travels from the sender to the receiver.
    - ICMP is an adjunct to IP, used by IP to exchange error messages and other vital information with the IP layer in another host or router.
    - IGMP is used with multicasting, sending a UDP datagram to multiple hosts.
  - TCP provides a reliable transport layer using the unreliable service of IP, performs timeout and retransmission, sends and receives end-to-end acknowledgments.

- A router by definition, has two or more network interface layers. Any system with multiple interfaces is called multihomed.

- One goals of an internet is to hide all the details of the physical layout of the internet form the applications.

## Internet Addresses

- Every interface on an internet must have a unique Internet address(IP address). 32-bit numbers written in four decimal numbers.
- Divided by the first number of a dotted-decimal address, IP address is categorized into 5 classes.
- Three types of IP addresses: **unicast**(destined for a single host), **broadcast**(destined for all hosts on a given network) and **multicast**(destined for a set of hosts that belong to multicast group).

## Encapsulation

- When data is sent down the protocol stack, through each layer, each layer adds information to the data by prepanding headers(sometimes trailer).
  - TCP header is called a TCP segment, and IP header is IP diagram.
- IP has multiple sources, IP must add some type of identifier to IP header that it generates, to indicate the layer to which the data belongs. IP handles this by storing an 8-bit value in its header called the protocol field.

## Demultiplexing

- When an Ethernet frame is sent up the protocol stack, all the headers are removed by the appropriate protocol box. Each protocal box looks at certain identifiers in its header to determine wich box in the next upper layer receives the data.

## Client-Server Model

- Most networking applications are written assuming one side is the client and the other the server, for the server to provide some defined service for clients:
  - iterative server: process the client request and then send response back to the client.
  - concurrent server: start a new server to handle this client's request, involving creating new process, task or thread.
- The advantage of concurrent server is that the server just spawns other servers to handle the client requests.

# Link Layer

## Introduction

Purpose of the link layer in the TCP/IP protocol is to send and receive
- IP datagrams for the IP module
- ARP requests and replies for the ARP module
- RARP requests and replies for the RARP module

## Ethernet and IEEE 802 Encapsulation

- Ethernet is the predominant form of local area network tech used with TCP/IP today, operates at 10 Mbits/sec and uses 48-bit addresses.
> I omit much detail of header and bytes limit, check when necessery

## SLIP & Compressed SLIP
- Both of them use a END character to terminate the IP datagram. And to prevent any line noise before this datagram from being interpreted as part of this datagram, END character is placed at the beginning.
- If byte in the IP datagram equals END character, another two-byte sequence is used to replace that byte.
- SLIP: stands for Serial Line IP, is a simple framing method and is widely used despite the following drawbacks
  - Each end must know the other's IP address, no method for one end to inform its IP address
  - No type field. If a serial line is used for SLIP, it can't be used for some other protocol at the same time.
  - No checksum added, depends on the upper layers to detect and throw away corrupted frames.

Since SLIP is slow and frequently used for interactive traffic, there tend to be many small TCP packets exchanged across a SLIP line, and leading to overhead of large header.  
- Compressed SLIP is used to solve this overhead:
  - CSLIP normally reduces the 40-byte header to 3 or 5 bytes.
  - It maintains the state up to 16 TCP connections on each end of the CSLIP link and knows that some of the fields in the two header form a given connection normally don't change.
  - These smaller headers greatly improve the interactive response time.

## Point-to-Point Protocol
- PPP consists of three components
  - A way to encapsulate IP datagrams on a serial link, supports either an asynchronous link with 8 bits of data and no parity or bit-oriented synchronous links.
  - A link control protocol to establish, configure, and test the data-link connection. Allows each end to negotiate various options.
  - A family of network control protocols specific to different network layer protocols.
- Each frame begins and ends with a floag byte shose value is 0x7e, followed with an address byte whose value is always 0xff, and then a control byte, with a value of 0x03.
- Next comes the protocol field, a value to categorize the field after it.
- CRC field is a cyclic redundancy check to detect errors in the frame.
- Also PPP needs to avoid flag character in the data field, using some rule to avoid it.
- PPP has the following advantages:
  - support for multiple protocols on a single serial line
  - a cyclic redundancy check on every frame
  - dynamic negotiation of the IP address for each end
  - TCP and IP header compression similar to CSLIP
  - a link control protocol for negotiating many data-link options.
> The author think it will replace SLIP someday.

## Loopback Interface

Loopback interface allows a client and server on the same host to communicate with each other using TCP/IP. 127 is reserved for it, and most systems assign 127.0.0.1 to it and name it localhost.

- most implementations perform complete processing of the data in the transport layer and network layer, and only loop the IP datagram back to itself when the data gram leaves the bottom of the network layer.
  - Everything sent to the loopback address appears as IP input
  - Datagrams sent to a broadcast address or a multicast address are copied to the loopback interface and sent out on the Ethernet, because the definition of broadcasting and multicasting includes the sending host.

It simplifies design even though seems inefficient.

## MTU & path MTU

Ethernet and 802.3 limit the number of bytes of data to 1500 and 1492. Thus, MTU(maximum transmission unit) is introduced to link layer.  
If IP's datagram is longer than MTU, IP performs fragmentation.

When two hosts on the same network are communicating with each other, each link can have a different MTU, and the smallest called path MTU makes sense.

# Serial Line Throughput Calculations

- Time to send a packet is the length of packet devided by the speed of the line. And on average, half of the time need to wait if we are using SLIP link for an interactive application, along with an application such as FTP.
- Reduce the length of packet help to reduce latency but also reduces the maximum throughput as well.
- The average wait calculation only applies when a SLIP link is used for both interactive traffic and bulk data transfer.