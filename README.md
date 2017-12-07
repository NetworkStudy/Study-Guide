# Concurrent Server Code:


```python
# create socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# re-use the port
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
# bind socket to host and port
sock.bind((host, port))
# Start listening
sock.listen(5)
While True:
	# accept connection
	conn, address = sock.accept()
	# fork child
	pid = os.fork()
	# child
	if pid == 0:
		sock.close()
		handle(conn)
	# parent
	else:
		conn.close
```
 
 # Steps to create a Daemon:

Runs as a background process that requires no interaction with a user. Typically it will not have an associated terminal. 

**Steps:**
1. fork from parent process
2. Disassociate from controlling terminal and process group
3. Close standard file descriptors
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
 
 
 
 
 
 
 
 
 
 
 
