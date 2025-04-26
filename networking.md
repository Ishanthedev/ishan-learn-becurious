### Networking to VPC fundamentals and concepts:
*Object is to -- make a single point of reference to instantly brush up the concepts*
Internet--> inter network of networks

**Main Topics for Networking**
* LOCAL Networking -Ethernet
* Routing 
* Segmentation, Ports and Sesssion
* Application

OSI 7 Layer of Model -- Conceptual How each components works and interact with others


| OSI Model Layers | Description                                                                              |
| ---------------- | ---------------------------------------------------------------------------------------- |
| **Host Layer**   | How data is segmented and reassembled for transport, and formatted for mutual understanding. |
| Layer 7 - Application | (Web Browser/Web Server)                                                                |
| Layer 6 - Presentation  |                                                                                         |
| Layer 5 - Session     |                                                                                         |
| Layer 4 - Transport   |                                                                                         |
| **Media Layer**  | How data is transmitted from point A to point B.                                        |
| Layer 3 - Network     |                                                                                         |
| Layer 2 - Data Link   |                                                                                         |
| Layer 1 - Physical    | (Network card or interface)                                                             |



## Layer 1 Physical | DUMB layer just standards |
* Median to carry unstructured inforamtion
* Medium can be Copper(electrical) Fiber(light) (Wifi RF)
* Specification denies the transmission and reception of RAW BIT STREAM between a device and shared physical medium. It defines things like
Voltage level, timing, rates and distance, modulation and connectors
* Network devices undersands Binary 1 and Binary 0 based on the agreed upon standards
    - Binary 0 and 1 and specific voltage type that both the connecting devices agree on

* **HUB**-- Anything received on any port is transmitted on every other port *including errors and collisions*, hub have a single collision domain.
* layer 1 is broadcast medium
* No individual devices address 
* All the data is processed by all devices 
* If multiple devices transmits at once -- a collision occurs therefore only 1 devices can transmit the data
* L1 has no media access control and no collision detection
* no access control
* No device to device communication
* More devices hence more collisions 

## LAYER 2 Data LINK layer | THE MOST CRITICAL LAYER|

Layer 2 can run on different types of **Layer 1 (Ethernet|fibar|wifi|**

* **FRAMES** -- Are the format for sending the infromation to the over layer 2
* **MAC address** -- hardware address - 48 bits long - MAC is hardware layer- made by manufacture is **globally unique**
	- Written as six pairs of hexadecimal digits (e.g., AA:BB:CC:DD:EE:FF)
	- Made out of OUI (Organizationally Unique Identifier) – 24 bits and device identifer bits
* **Frame layout**
    - **Preamble**(start of the frame) 56 bits SFS **8 bit**, allows devices to know that this is the start of the frame.
    - **Destination MAC Address** Layer 2 to address of destination device.
    - **Source MAC Address** Layer 2 address of the source device, device transmitting the frame
    - **Ether Type** **16 bits**, contains the information of IP or the internet protocol, which layer 3 protocol originally put data into that frame.
    Preamble, Destination, MAC Address, Source MAC Address and Ether Type are called as **MAC HEADER**
    - **Payload** Anywhere from 46 to 1500 bites, contains the data that the frame is sending.Data is generally provided by the layer 3 protocol and EtherType attributes defines which L3 protocol is used. 
    - **FCS** **32 bit** To check if the corruption has occurred or not.
* Layer 2 provides controlled access to the physical medium.
* Imagine a 2 PC connected by an ethernet cable left PC and write PC
* [**Carrier Sense Multiple Access (CSMA)**](https://www.ccbp.in/blog/articles/csma-in-computer-networks#:~:text=Carrier%20Sense%20Multiple%20Access%20(CSMA)%20is%20a%20network%20protocol%20that,Ethernet%20network%20or%20wireless%20environment.) is a network protocol that operates in the data link layer (Layer 2) of the OSI model. It is designed to prevent collisions when multiple devices attempt to transmit data simultaneously over a shared medium, such as an Ethernet network or wireless environment
  
   - Left player intends to send data to mac address 3e:22:fb:b9:5b:76 layer 2 creates a frame F1
   - Then left PC will check for the carrier
   - If no carrier, layer 1 takes the frame, converts to physical standard and transmits
   - L1(Right) Receives and passes frame to layer 2.
   - When the right game needs to send to left game, its layer 2 builds a frame.
   - If carrier is detected, then wait another device is transmitting.
   - Frame sent to layer 1 for transmission.
**Collsion Detention** this is essential for layer 2 
**Hub** Does not avoid collision as it is a layer 1 device.
**Switch** Is a layer 2 device just like hub however understands layer 2, hence provides significant advantages.
    - Switch understands **frame** and **MAC Address**. They maintain a mac address table which starts off empty.
    - As the swtich receives frames on its ports, it learns which devices are connected and populates the Mac address table.
    - Switch will limit collision. 
    - Switch deals with frames only, if it is valid, it then forwards it.
    - We can do unicast meaning 1:1 communication.
    - We can do Broadcast 1: All communication.

## Layer 3 Network layer

* Layer 3 is a common protocol which can spam multiple different layer 3 networks.
* Internet Protocol (IP) is a layer 3 protocol which adds cross network IP addressing and routing to move data between Local Area Netwrok
* IP packets are moved step by step from source to destination via intermediate networks. Encapsulated in different frames along the way.
* **Routers** Is a layer 3 devices which moves packets of Data across different networks.
* Router devices remvoed frame encapsulation and add new frame encapsulation at every hop.
  
- Layer 3 IP V4 packet structure only important ones:
    - Protocol
    - Source IP address
    - Destination IP address
    - DATA(By Layer 4)
    - Time to Live(TTL) How many hops a packets can take before getting discarded.
- Layer 3 IP V6 packet structure only important ones:
  
    - Source IP address (Bigger Large address)
    - Destination IP address (Bigger Large address)
    - Data From layer 4
    - Hop Limit
* **Ip addressing** *Structre of IP Version 4*
  
    - 133.33.3.7 (Dotted decimal notation) 4 X 0-255
    - 133.33 is the network part
    - 3.7 is the host part
    - if the network part of two IP address matches, it means they are on the same IP network, if not the are on the different networks.
    - Each decimal part of the IP address is an **8 bit binary** octets number like following:
  
      -10000101 00100001 00000011 00000111   
    - The above network has a 16 prefix. 16 bits of the IP are the network and the remaining bits are for hosts
    - so 133.33.33.37 are in same network
    - If the network component of the device matches then the device is local other wise it is remote.
    - Ip address are assigned by either humans (Static IP) or assigned by a machine called DHCP(Dynamic Host configuration)
* **Subnet Mask**
  - It is the subnet mask which allows a host of determine if an IP address it needs to communicate with is local or remote- which influences if it needs to use gateway or can communicate locally. 
  - Home router is default gateway
  - Subnet mask tells us which part of the IP is in the network
  - A subnet mask si configured on a host device in addition to an IP address, eg **255.225.0.0** and this is the same as **/16** prefix.
  - 11111111 11111111 00000000 00000000 --**SUBNET MASK**
  - 10000101 00100001 00000011 00000111 --**IP ADDRESS**

    - IP address and the subnet mask start with 1 hence they are in the same network
  
 **Route Tables** *How packets gets transferred from home to destination*
    - Packet SRC 1.3.3.7(Home) Destination 52.217.13.37 Default route 0.0.0.0/0 sends all the packets to ISP(Internet service provider)
    - ISP has multiple interface route table is used for selection
    - Based on the Route table of ISP router
  
| Destination     | Next Hop/Target |
|---------------------|---------------------|
| 52.217.13.0/24       | 52.217.13.1         |
| 0.0.0.0/0           | 52.43.217.1         |
| 52.43.215.0/24      | 52.43.215.1         |


   - Router compares packet destination IP and route table matching destination. The more specific prefix are preferred(0 lowest, 32 highest). Packet is forwarded on to the next Hop/Target
   - Routing is the process where the packet hop by hop across the internet from source to destination(Mac address of router)
   - ARP(Address resolution Protocol L3 work) protocol will give you the mac address of the given IP address
   - Limitation -- 
     - No method for channels of communication  SRC IP <=> DIST IP Only
     - IP can be delivered out of order

## Layer 4 Transport layer and Layer 5 session layer

Issues with layer 3:
 1. Un-ordered packets leaving layer 3.
 2. Missing packets in layer 3.
 3. Delay in delivery
 4. There are no communication channel packets have a source and destination IP but no method of splitting by APP or Channel.
 5. No flow control, if the source transmits faster than the destination can receive it can saturate the destination causing the packet loss.
   
Note: Every packet is different.

**What is layer 4 and how does it fuction?** 

**yer 4 adds protocol:**
1. TCP - Reliability, error correction and ordering of data.
   1. It is used for most important application layer protocol like HTTP,HTTPS,SSH and so on
   2. TCP is connection oriented communication channel, once setup it creates a bidirectional channel for communication.
   3. Slower  
2. UDP - Less Reliable and faster than TCP.

TCP segments are incapsulated within the IP packets.
Segment don't have src and destination IP's the packets provide device addressing

**TCP SEGMENT** aka TCP header
   1. Source port
   2. Destination port
   3. sequence number
   4. Acknowledgement
   5. Flag 'N' Things 
   6. Window
   7. Checksum 
   8. Urgent pointers
   9. Options
   10. Padding
   11. Data

A TCP segment is the fundamental unit of data transmission in the Transmission Control Protocol (TCP), used for reliable communication between a client and a server over a network.


**How segment is used in TCP** /**Architecture of TCP**

TCP is a connection based protocol. A connection is established between two devices using a random port on a client and known port on the server.

Once established the connection is **BI-Directional** The connection is reliable connection, provided via the segment encapsulation in IP packets.

In L3 packets - No Error checking, no ordering no association.

Segment linked to a connection ordering and retransmission.IF packet is lost or unordered the TCP can manage the both the issues.
Channels are created using segment and they are Bi-directional, one for client to server and another for server to client.

There are 2 kinds of port when used in TCP communication:

1. **Well KNOWN PORT** A server port like TCP 443 for HTTPS.
2. **Ephemeral Port** A temporary port that a client uses to communicate to the server.Temporary ports that a client picks as the source port when initiating a connection to a server. These ports are dynamically assigned by the operating system from a predefined range and are used for the duration of the sess

It is important to understand that the for layer 4 perspective that the 2 way communication will be from Ephemeral port to well known port and vise versa. 
One set of rules for laptop to server and another set of rule from server to laptop.

When we mention ephemeral port, it means a port range that a client picks as a source port. These should be allowed back to the client as firewall rules on the server.

That's why you need 2 sets of rules for NETWORK ACL(Access control list within AWS)(State full)

**TCP Connection 3 way handshake** *has 4 steps*

**Step 1** Client sends a **SYN** Send a segment with SYN sequence set 'CS'(ISN) Initial sequence number -- *sequence = cs*.
**Step 2** Server sends  **SYN-ACK**, picks random ISN sequence 'ss' send segment -**SYN** and **ACK** Acknowledge sets to **CS+1**, I have received up tp CS send CS+1.
**Step 3** Client send segment with **ACK** send segment with ACK Acknowledge set to **SS+1**, I have received SS, send SS+1 sequence set to CS+1
**Step 4** The connection is established and the client can send data.

**Session and the state**

 **State less firewall** Network ACL (AWS) has 2 rules one for outbound and another one inbound. As it does not understand the state of TCP connection.

 A **Stateful** firewall views TCP connection as one thing outbound, LAPTOP-IP and TCP 23060 => server IP and TCP port/443 allowing the outbound implicitly allows the inbound response.

 # NAT Network Address Translation

 NAT is fundamental technology used in computer networking to allow multiple devices to share a single PUBLIC IP address. 

 1. Nat is used to designed to overcome IPv4 Shortages.
 2. Private IP address like 10.0.0.0 are used in multiple places and can't be used over internet.
 3. To give internet access to private devices we need to use NAT
 4. NAT also provides some security benefits too
 5. NAT only works for IPV4 and not for IPV6
 6. There are many kinds of NAT but they all translate Private IPV4 address to Public:
    1. **Static Nat** - 1 private to 1 fixed public address (IGW) in both ways.
       1. From Private zone -- a device Ip(Private) are mapped to 1:1 Public IP in by a static NAT in AWS knows as Internet Gateway
       2. Packets are generate as normal with a private source (SRC) IP, and an external Destination DST(IP)
       3. IF a public IP has been allocated to the private IP, the source address of the packet is translates as it passes through the NAT device
       4. The same process is works in both direction from public to private
    2. **Dynamic NAT** -1 private to 1st available public
       1. Public IP allocation are temporary and multiple devices can use the same allocation over time as long as there is no overlap
       2. If the public IP Pool is exhausted then external access can fail
       3. Only one Private IP will be mapped to one Public IP address mapping at any time. It is still 1:1 for the duration of the allocation
    3. **Port address translation (PAT)** many private to 1 public NATGW
       1. In AWS this is how the NATGatway(NATGW) function - a (Many:1)(Private:PublicIP)Architecture 
       2. The NAT Device records the source (Private)IP and source Port.
       3. It replaces the source IPO with single Public IP and Public source port allocated from a pool which allows IP overloading (many to one)
       4. Return Traffic has tcp/443 and 1.3.3.7 as the source and the public IP of the NAT device adn destination
       5. Public port and public IP are translated to Private Port and Private IP
       6. Port address translation PAT maintains a NAT table with following content:
       Private IP Private Port Public IP Public Port

| NAT Entry | Private IP    | Private Port | Public IP     | Public Port |
|-----------|--------------|--------------|---------------|-------------|
| 1         | 10.0.0.100   | 1024         | 155.4.12.1    | 20345       |
| 2         | 10.0.0.101   | 1056         | 155.4.12.1    | 20346       |
| 3         | 10.0.0.102   | 1025         | 155.4.12.1    | 20347       |
| 4         | 10.0.0.103   | 1100         | 155.4.12.1    | 20348       |


*How does my laptop private IP address gets its IP*

| Device | IP Type    | Who Assigns It        | Visibility          |
|--------|------------|----------------------|---------------------|
| Laptop | Private IP | Router (via DHCP)    | Local network only  |
| Router | Public IP  | ISP                  | Visible on internet |

If you want your laptop to be accessible from the internet (for example, to run a server), you would need to configure port forwarding on your router to direct external traffic to your laptop’s private IP address. However, this does not give your laptop a public IP; it simply allows certain types of incoming traffic to reach your laptop through the router

   