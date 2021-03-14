# Transport Layer
From the header fields, the `next proto` identifies the layer 4 protocol. Furthermore, the layer 4 header and payload make the layer 3 payload.
* L4 header length is dependent on the protocol
* L4 payload = total length of L3 payload - L4 header length
## Internet Control Message Protocol (ICMP)
Also known as **Protocol 1**, defined in **RFC 792** (where standards are documented and managed by the **Internet Engineering Task Force (IETF))**.  
    
Appears in the L4 portion of an IP packet but it's usually considered as a part of L3 since you a functioning ICMP for L3 to work.    
    
ICMP Packet
* Header 
  * 1b type
  * 1b code
  * 2b checksum
* payload

ICMP communicates information about the network (e.g. any issues and where they occurred).    
    
ICMP Types:
* Ping
  * 8: echo request
  * 0: echo reply
* 3: destination is unreachable
* 11: time exceeded (usually TTL = 0)
* 4: source quench

Codes are type-specific:
* Types 0 and 8 are always code = 0
* Type 3 has several possible code:
  * 0: network was unreachable
  * 1: host unreachable
  * 2: protocol unreachable (destination doesn't support protocol)
  * 3: port unreachable (can reach host but no process listening at the port)
  * 4: fragmentation required, but DF bit was set
  * 5: source routing failed 
## UDP
  Protocol 17 (0x11) and is the simplest transport protocol for data.
  * No reliability
  * Best effort

These requirements allow for **multiplexing** at hosts. However, only 1 message can only be assigned to 1 packet. Header includes
* **source port** and a **dest port** for transportation purposes
* length
* checksum

Advantages of UDP
* Connectionless (no sessions)
  * No negotiation with remote host
  * If we have data to send, we store it in a packet and send it
  * Able to receive responses from its port
* Instead of sessions, we have 4-tuples (source_addr, dest_addr, source_port, dest_port)
  * Usually enough information for a higher protocol to use
* Very lightweight
* Good for single-message protocols
* Good for streaming: we don't have to attempt to retransmit lost packets
## Reliable Transmission
Use **ackowledgements (ACKs)**:
* If we receive an ACK, that means that the destination received the packet
* If we don't receive an ACK, either packet didn't reach destination or the ACK was dropped (can't distinguish).
  * Use timeouts to determine if we didn't get an ACK. Then the packet will be **retransmitted** (transmit = xmit)
* INSERT WATERFALL DIAGRAM

If we have ACKs and timeouts, it is called **Automatic Repeat Request (ARQ)**.
* **Stop-and-wait**: implementation that
  * sends packet
  * waits for ACK (if there is a timeout then we re-transmit the packet)
    * attach a sequence number to the packet and if the destination has already seen this sequence number, ignore it
    * set a flag that says the pack transmission is a repeat
    * **Stop-and-wait**: 1-bit alternating sequence number so there can only be 1 outstanding packet and the timeout implies the maximum usage of the channel.
  * send next packet

**Throughput**: roundtrip delay * bandwidth
## Sliding Window
**Send Window Size (SWS)**: maximum number of unacknowledged packets)   
**LAR**: Last ACK Received     
**LFS**: sequence number of the Last Frame Send   
We need to satisfy: LFS - LAR $\leq$ SWS
* As we receive more ACKs, LAR increases with timeouts for re-transmission as necessary (must hold a buffer of size SWS).     

**RWS**: Receive Window Size (maximum number of acceptable out-of-order packets)    
**LAF**: Sequence number of the Largest Acceptable Frame      
**LFR**: Last Frame Received      
Need to satisfy: LAF - LFR $\geq$ RWS     
How it works:
* Source sends the Destination a packet
* At the Destination:
  * If packet.seq $\leq$ LFR[s] or p.seq $>$ LAF[s] $\rightarrow$ discard p
  * Otherwise
    * accept the packet
    * shift LFR[s] if needed
    * ACK(S, LFR[S}) if LFR[S] changed
    * LAF[S] = LFR[S] + RWS
## Variation on ACK
Early loss detection:
* **Negative ACK (NACK/NAK)**: indicates missing packets but is unnecessary if timeouts are being used.   
* **Duplicate ACK (DACK)**: when 7 + 8 are received, we ACK 5 again which allows the sender to know that the data was received but data is missing (6 is missing and needs to be reset).    
* **Selective ACK (SACK)**: ACK out of order packets.
  * Improves utilization but increases complexity
## Choosing Window Sizes
If we know the round trip time and bandwidth, can compute an optimal SWS for a given packet size:     
SWS = throughput/(bits/pkt)   
    
RWS = 1 $\implies$ receiver won't buffer out of order packets.    
RWS = SWS $\implies$ receiver can buffer anything the send transmits.   
    
Sequence numbers are finite in size so they will eventually run out and will have to wrap.  
b bits of sequence numbers $\implies$ SWS $< 2^n$   
For stop-and-wait: $b = 1$ and SWS = 1.   
RWS = SWS $\implies$ SWS < $2^{b-1}$ so S does not send a packet with seq number in range that D has buffered.    

    
Sliding window gives:
* Reliable delivery
* Preservation of packet order for higher layers
* Support for **Flow Control**
## TCP
Protocol 6    
    
For sliding window, we assume we have **point-to-point** links.   
TCP connects hosts across the Internet and has the following properties:
* Reliable
* Connection oriented
* In-order delivery
* Full-duplex
* Flow-control (receiver limits the sender's rate)
* Congestion control (feedback from network limits sender's rate)

How TCP works
* **Connection Establishment Phase**: Set up state for sliding window
* **Explicit Teardown**: Clean up state
* Wide range of RTTs so retransmission timeouts must be **adaptive**
* Lots of reordering so we define the **Maximum Segment Lifetime (MSL)**: 120 seconds
* Learn the remote host's resource limits
* Sender's queue doesn't reflect what's occurring in the network.
* Link bandwidths vary, leading to **congestion**
* Endpoints guarantee the properties that they need (don't try to build properties into the network)
  * If we want to push functionality to lower layers, we must ensure that it can be implemented correctly and completely
  * E2E guarantees are usually more effective since the end points handle everything and reduces the burden on the network

TCP is a **byte-stream protocol**
* We write bytes to a socket and read bytes from a socket
* Buffers bytes and groups them together into **segments** (packets)
* Demultiplex streams via (sport, saddr, dport, daddr)

TCP Header
* 5 words (20 bytes) in the basic header. More can be added with options
