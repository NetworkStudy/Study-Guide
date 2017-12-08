# Concurrent Server Code:


```python
# create socket
serverSocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# bind socket to host and port
serverSocket.bind((host, port))
# Start listening
serverSocket.listen(numOfConnections)
While True:
	# accept connection
	clientSocket, address = sock.accept()
	# fork child
	pid = os.fork()
	# child
	if pid == 0:
		serverSocket.close()
		handle(clientSocket)
	# parent
	else:
		clientSocket.close()
```
 
 # Steps to create a Daemon:

Runs as a background process that requires no interaction with a user. Typically it will not have an associated terminal. 

**Steps:**
1. Ignore terminal stop signals
1. fork from parent process and exit parent
2. Disassociate from controlling terminal and process group
3. Close standard file descriptors (stderr, stdin, stdout, ...)
4. Change working directory
5. Change file mask mode (umask(0))

# Virtual Circuit vs. Datagram circuit:
**Virtual circuit:** Appears as though there is a dedicated physical layer link between the source and destination end systems.
* Virtual circuit communication resembles circuit switching because it is connection-oriented
* All packets follow same path.
* Resources (buffers, CPU, bandwidth, ect.) reserved for duration of connection. 
* First sent packet reserves resources and configurs network for following packets.
* Packets are recieved in order.
* Reliable but costly.

**Datagram circuit:** The normal method of the internet. There is no dedicated connection between the host processes. 
* Connectionless.
* Packets may follow diffrent paths.
* No resources are reserved.
* All packets must have headers with configuration information.
* Packets may be received out of order.
* Not as reliable but more cost-efficient. 



# Chapter 3: Transportation layer

**UDP:** multiplexing/demultiplexing

## Reliable Data Transfer Protocol

### Go-Back-N (GBN):

**GBN Protocol:** The sender is allowed to transmit multiple packets with out waiting for an ACK but is limited to sending no more then N max unACK'd packets. 

**Sliding-window Protocol:** GBN is called such as the window slides over the sequence number space. 

![Sender Go Back N Image](http://www.linyibin.cn/images/Technology-ComputerNetworking-Internet-GBN-Window.png)

* *base:* Sequence Number of oldest unACK'd packet
* *nextseqnum:* Smallest unused sequence number, i.e. sequence number of next packet to be sent
* *Window Size* or *N:* Range of permissible sequence numbers for unACK'd packets

In practice the range can be determined from the sequence number in the packet's header. If *k* is the number of bits of the sequence number then [0, 2ᵏ - 1] is the range of sequence numbers. That is the the next sequence number after 2ᵏ - 1 will be 0. 


![](http://www.myreadingroom.co.in/images/stories/docs/dcn/gobackn%20automatic%20repeat%20request.JPG)

![](https://astro.temple.edu/~stafford/cis320f05/lecture/chap3/gifs/kurose_320719_c03f22.gif)

### Selective Repeat:

![](https://astro.temple.edu/~stafford/cis320f05/lecture/chap3/deluxe-content_files/03-22.jpe)

![](http://www.networkinginfoblog.com/contentsimages/SR%20operation.JPG)

### Go-Back-N vs. Selective Repeat
#### Go-Back-N
* Retransmits all the frames sent after a lost or corrupted frame
* Window size is N-1
* Sorting is not required by sender or receiver
	* If a packet is recieved out of order all packets after last ACK'd are resent
* Receiver does not store out of order packets
* Iteration of frames not required
* NAK number is next expected frame
* Less complex and cheaper

#### Selective Repeat
* Retransmits only those frames that are suspected to lost or corrupted
* Window size is <= (N+1)/2
	* If the window size is too large we do not know if a packet is new or a retransmittion 
* Receiver must be able to sort as it has to maintain the sequence of the frames
* Receiver stores the frames received after a corrupted frame in the buffer
* The sender must be able to search and select only the requested frame
* NAK number refer to the frame lost
* Complex and costly



# Chapter 4: Network Layer

## Router
![](https://electronicspost.com/wp-content/uploads/2016/05/4.6.png)
* *Input Port:* Terminates incoming physical link, performs link layer functions, perform lookup function
* *Switching Fabric:* Connects input ports to the output ports
* *Output Ports:* Stores packets recieved from switching fabric and transmits them to outgoing link. Performs link-layer and physical-layer functions 

## Switch Fabric
![](http://sirbonbac.com/images/Computer_Networking/Switching_Techniques.PNG)

*Switching via Memory:* Done using a traditional computer CPU. Input and output ports were functioned as I/O devices in an OS. Incoming packets signaled the routing processor via an interrupt. Each packet must be written into and read from memory, meaning only one packet can be forwarded at a time. 

*Switching via a Bus:* Input ports foward the packets without the routing processor. This is done by the input port pre-pepending a label onto the packet. All outgoing packets receive the packet but only the one with the matching label keeps it. The outgoing port removes the label. Only one packet can cross the bus at once.

*Switching via an Interconnection Network:* A *crossbar switch* consists of *2N* buses to *N* input ports and *N* output ports. The vertical buses intersect the horizontal buses at a *crosspoint*, which can be opened or closed. When a packet needs to forwarded the relevant *crosspoint* closes, so only the correct ouput port recieves the packet. The *crossbar switch* is **non-blocking**, so more than one port can recieve a packet at a time. However, only 1 packet can be sent over a given bus, meaning that if diffrent input ports want to send a packet to the same output port then the others will have to wait while one gets sent.

**forwarding:** Maps packet from input link to output link using forwarding table

**routing:** determines path from sender to receiver. They make the forwarding table

**longest prefix matching:** a packet will match the entry with the longest prefix, in cases of 2 or more matches

**ATM protocol** provides services such as throughput, delay, and jitter guarantees.

### Ideal Buffering 
*Flow control buffering* = RTT * Transmission rate

*Buffer* = RTT * trans rate / sqrt(# of TCP flows)

### Packet Dropping
If the query is full we must drop a packet 

**Drop-tail:** Drop arriving packets

**Random early Drop (RED):** Randomly drop incoming packets before Queue becomes full
* Used in Active Queue Managment

**Head of line blocking:** A packets output port is open but the packet in front is waiting

## Addressing 

**Interface:** Boundary between host and physical link
* IP addresses are associated with routers' interfaces; typically a router will have multiple IP addresses
 
**Dotted Decimal Notation:** The format IP addresses use. Each byte of the address is written in its decimal form and is separarted by a period (dot) from the other bytes. 
* *Example: 193.32.216.9*

* The binary notation is 11000001 00100000 11011000 00001001

### Subnet

**Subnet:** A part of a larger network.

IP addressing assigns an address block to subnets, such as 223.1.1.0/24

**Subnet Mask:** "/24" of 223.1.1.0/24 is a subnet mask. It indicates the leftmost bits of the 32 bit address that identify. Thus the last 24 bits of 223.1.1.0/24 are addressable by the subnet.

Subnets refer not just to the connection of computers to router but can also be applied in differnt instances, such as between routers. Below the interfaces that connect R1 & R2, R1 & R3, and R2 & R3 are subnets.

![Network Image](http://www.networkinginfoblog.com/contentsimages/Three%20routers%20interconnecting%20six%20subnets.JPG)

**Classless Interdomain Routing (CIDR):** The internet's address assignment strategy  

**Prefix:** The first x bits of the a.b.c.d/x address (subnet address)
* IP address of devices within an orginization will share a common prefix 
* Only the prefix will be considered by routers outside the orginization
* The remaining bits will be considered when forwarding packets *within* an organization

It is possible for an orginization to contain additional subnets

**Classful addressing** was problematic because it required subnet masks to be 8, 16, or 24 bits in length. These were refered to classes A, B, and C respectively. Class A supported up to 254 hosts, while the next-up class B supported up to 65,534 hosts. Class A may be too small and class B too big, effectively wasting IP addresses. 

**Broadcast Address:** The IP address 255.255.255.255. Packets sent to this destination are forwarded to all hosts on the same subnet.

**Obtaining a block of addresses:** An orginization contacts its ISP to obtain the block of addresses. The ISP provides the addresses from a larger block of addresses allocated to it. The ISPs obtain their address block from the *Internet Corporation for Assigned Names and Numbers (ICANN)*. ICANN also manages the root DNS servers, assigns domain names, and settle domain name disputes. 

#### Obtaining a host address:
Assigning IP address can be done manually by a System Administrator or 

**Dynamic Host Configuration (DHCP)** can automatically assigns IP addresses to hosts.

**_Steps to connect to DHCP:_**

1. Host sends a *DHCP discover message* to the **broadcast address**
2. Server sends a *DHCP offer message* to the **broadcast address**
3. Host accepts a one of possible multiple offers and sends a *DHCP request message* to chosen server
4. Server confirms with a *DHCP ACK message* and assigns the IP address

# Chapter 5

## Link-State (LS) Routing Algorithm

This algorthim is a **centralized routing algorthim**, meaning it requires complete knowledge about the network.

Worst-case complexity: O(n²) 

**Link-state broadcast:** Causes all nodes to have an identical and complete view of the network. Each node then runs the LS algorthim and computes set of least cost paths to every other node.

*Dijkstra's algorthim:* Visits the unvisted node with the smallest path cost until the shortest path is found. 

![](https://upload.wikimedia.org/wikipedia/commons/5/57/Dijkstra_Animation.gif)

---
 
 ![](http://www.dcs.bbk.ac.uk/~ptw/teaching/IWT/network-layer/abstract-graph-model.gif)
 
 ![](http://www.networkinginfoblog.com/contentsimages/Running%20the%20link-state%20algorithm%20on%20the%20network.JPG)
 
 * *D(v):* Cost of the least-cost path from the source node to destination *v* so far
 * *p(v):* Previous node (neighbor of *v*) along the current least-cost path
 * *N':* Set of vistited nodes 
 
 
## Distance-Vector (DV) Routing Algorithm

Each router knows its own address and
the cost to reach each of its directly connected neighbors.

The algorthim is:

* *Distributed:* Nodes receive information from one or more of its *directly-attached* neighbors, perform calculations, and then distribute the info back to its neighbors. 
* *Iterative:* The prior process continues until no more information is passed between neighbors.
* *Asynchronous:* Nodes are not required to operate in step with each other.
 
 **Bellman-Ford equation:** *dₓ(y) = minᵥ{c(x,v) + dᵥ(y)}*  
 
 * *dₓ(y)*: Cost of the least-cost path from node *x* to node *y*
 * *minᵥ:* Equation taken over all of x's neighbors
 * *c(x,v):* Cost of x to directly attached neighbor v
 * *dᵥ(y):*  Distance vectors for each of its neighbors for each neighbor v of x

 
 ![](http://www.networkinginfoblog.com/contentsimages/Distance-vector%20algorithm.JPG)
 
 ### Link Cost Change and Link Failure  
 
 **Routing-Loop:** Example, Y routes to X through Z. Z in turn routes to X through Y. 
 
**Count-to-infinity:** Y updates cost to X and informs Z. Z in turn updates cost to X and informs Y. The loop continues for possibly many iterations. This happens because Y and Z point to each other to get to X.

**Poisoned Reverse:** If a path becomes unreachable or too far tell neighbor path is infinite.

# Chapter 6
## ALOHA Protocol

*Assumptions:* 
* All frames consist of *L* bits
* Time is divided into slots of size *L/R* seconds (slot is time to transmit one frame)
* Nodes start to transmit frames only at the beginnings of slots
* Nodes are syncronized so each node knows when a slot begins
* In case frames collide in a slot, then all nodes detect the collision event before the slot ends

*Operation of slotted ALOHA:*
* When node has a fresh frame it waits until the beginning of the next slot and transmits entire frame into the slot
* In case of no collision the transmittion was a success and there is no retransmittion
* In case of a collision the node retransmitts in the next slot with a probability between 0 and 1 until it is successful. All nodes in the collision have a seperate probability

**Efficiency of a 'Slotted Multiple Access' Protocol:** Long-run fraction of successful slots/total slots

**Efficiency of Slotted ALOHA:** *NP(1-p)^(N-1)*, Max is 0.37
* N is active nodes
* p is probability to retransmit
* Transmittion rate then is 0.37 * R

![](http://www.networkinginfoblog.com/contentsimages/Nodes%201%202%20and%203%20collide%20in%20the%20first%20slot.JPG)

## Carrier Sense Multiple Access (CSMA)

**Carrier Sensing:** Node listens for other nodes transmitting and waits until no others are transmitting

**Collision Detection:** Transmitting node litens to channel while transmitting. If another starts transmitting stop for a random amount of time and then go into carrier sensing.

Even with **carrier sensing** collisions still occur because of **channel propagation delay**, that is even though node B could be currently transmitting bits node D may not detect them before it begins to transmitt.

*Operation:* 
* Adapter obtains a datagram, prepares a link-layer frame, and puts the frame adapter buffer
* If channel is idle start transmittion of frame, if not wait until it is
* While transmitting watch channel for signal energy 
* If the adapter has transmitted the entire frame it is finished. If instead it senses signal energy abort transmission
* After abortion wait a random amount of time and return to step 2

**Binary Expontional Backoff:** At a collision a node chooses *K* value at random from {0,1...2ⁿ-1}, with n being the number of collisions of the node's current frame

**Efficiency:** 1 / (1 + 5 * dProp/dTrans)

## Link-Layer Switches

### Switches Versus Routers
Some packet switches, called **link-layer switches** base their forwarding decision on values in the fields of the link-layer frame; switches are thus referred to as link-layer (layer 2) devices.

Other packet switches, called **routers**, base their forwarding decision on the value in the network-layer field. Routers are thus network-layer (layer 3) devices, but must also implement
layer 2 protocols as well, since layer 3 devices require the services of layer 2 to implement their (layer 3) functionality. 

*Routers:*
* Forward packets using network-layer addresses
* Layer 3 packet switch
* Since network addressing is hierarchical, most packets do not cycle through routers  
* Since packets are not restricted to a spanning tree, it can choose past path between source and destination
* Provide firewall protection agianst layer-2 broadcast storms
* Not plug-and-play, connected hosts need to to have IP addresses configured
* Longer packet processing times since must be proccessed through layer 3 

*Switches:*
* Forward packets using MAC addresses
* Since MAC address are flat may result in cycling
* Restricted to a spanning tree
* Layer 2 packet switch
* Plug-and-play
* High filtering and forwarding rates
* Require large ARP tables
* Susceptible to broadcast-storms 
	* If one host begins to transmits endless broadcast frames, switches will forward all the frames
 

*Both:*
* Store-and-forward packet switches

![](https://image.slidesharecdn.com/ethernetvietnguyen-150404074840-conversion-gate01/95/ethernet-networking-presentation-20-638.jpg?cb=1428151802)
