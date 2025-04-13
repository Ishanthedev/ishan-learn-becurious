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


   - Router compares packet destination IP and royte table matching destination. The more specific prefix are preferred(0 lowest, 32 highest). Packet is forwarded on to the next Hop/Target
   - Routing is the process where the packet hop by hop across the internet from source to destination(Mac address of router)
   - ARP(Address resolution Protocol L3 work) protocol will give you the mac address of the given IP address
   - Limitation -- 
     - No method for channels of communcation  SRC IP <=> DIST IP Only
     - IP can be delivered out of order
