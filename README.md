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



# Chapter 3: Transportation layer

**UDP:** multiplexing/demultiplexing

# Chapter 4: Network Layer
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

 
 
 
 
 
 
 
 
 
 
 
