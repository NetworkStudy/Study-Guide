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

*Switching via an Interconnection Network:* 


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


 
 
 
 
 
 
 
 
 
 
