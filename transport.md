# Transport Layer
1. [Internet Control Message Protocol (ICMP)](#ICMP)
2. [Reliable Transmission](#ReliableTransmission)
3. [UDP](#UDP)
4. [TCP](#TCP)
<a name="ICMP"></a>
## 1. Internet Control Message Protocol (ICMP)
<center>

![](./assets/ICMPpacket.png =400x400)

</center>

Also known as **Protocol 1**, defined in **RFC 792** (where standards are documented and managed by the **Internet Engineering Task Force (IETF))**.  
    
ICMP communicates information about the network (e.g. any issues and where they occurred).    

Appears in the Layer4 portion of an IP packet but it's usually considered as a part of Layer3 since you need a functioning ICMP for Layer3 to work.    
    
ICMP Packet consists of:
* Header 
  * 1b `Type`: indicate type of problem detected by the sender
  * 1b `Code`: indicate type of problem detected by the sender
  * 2b `Checkum`: protects entire ICMP from transmission error
  * 4b `Data`: contains additional information about ICMP messages
* Payload

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
<a name="ReliableTransmission"></a>
## 2. Reliable Transmission
Accomplished using
* **Acknowledgements (ACK)**: frame a protocol sends to its peer saying it received the earlier frame
* **timeouts**: if the sender does not receive an ACK from its peer in a reasonable amount of time, it will **retransmit** the original frame

ACKs and timeouts are used to implement **Automatic Repeat Requests (ARQ)**
* **Stop-and-Wait**: after transmitting a frame, the sender waits for an ACK before transmitting the next frame. If a timeout occurs, the sender retransmits the original frame
  * Potential issue of retransmitting the same frame b/c either the ACK was lost or timeout occured too early. This causes duplicate copies of the frame to be delivered. To remedy the issue, stop-and-wait protocol includes a 1-bit **sequence number** (either 1 or 0 if it's a retransmit)
  * Shortcoming: sender can only transmit 1 frame at a time through the link, which may be lower than the link's capacity
<center>

![](./assets/StopAndWait.png =400x400)

</center>

* **Sliding Window**: 
  * Sender assigns a **sequence number** to each frame and maintains 3 variables:
    * **Send Window Size (SWS)**: upper bound of number of unackowledged frames the sender can transmit
    * **Last Acknowledgment Received (LAR)** sequence number
    * **Last Frame Sent (LFS)** sequence number
    * Variables must satisfy `LFS - LAR <= SWS`
    * When the sender receives an ACK, LAR is moved to the right, allowing the sender to transmit another frame.
    * A timer is associated with each frame transmitted and the sender retransmits the frame if an ACK is not received in time
    * Sender must be able to buffer up to SWS frames, since they might retransmitted if timeout occurs
  * Receiver maintains 3 variables:
    * **Receive Window Size (RWS)**: upper bound of number of out-of-order frames the receiver can accept
    * **Largest Acceptable Frame (LAF)** sequence number
    * **Last Frame Received (LFR)** sequence number
    * Variables must satisfy `LAF - LFR <= RWS`
    * If a frame arrives with `SeqNum <= LFR` or `SeqNum > LAF`, the frame is discarded
    * If the frame has `LFR < SeqNum <= LAF` the frame is accepted
      * Let `SeqNumToAck` be the largest sequence number not ACK'd such that all frames with sequence numbers $\leq$ `SeqNumToAck` have been received. Then `SeqNumToAck` is accepted and
        * `LFR = SeqNumToAck`
        * `LAF = LFR + RWS`
  * Shortcomings: if a timeout occurs, the link pipe isn't full and the sender can't advance its window because of the invariant it must satisfy.
  * Sliding window can play 3 different roles
    * Reliable deliver frames over a unreliable link (discussed above)
    * Preserve order in which frames are transmitted (by using the sequence numbers)
    * Support **flow control**: prevents sender from over-running the receiver using `RWS`

<center>

![](./assets/SlidingWindow.png =300x250)

</center>
<a name="UDP"/>

### 3. UDP
Simplest transport protocol extends host-to-host delivery of the underlying network into proceses-to-process communication and should allow for **demultiplexing** since there are multiple process running on a given host.

UDP has processes **indirectly identify** each other using **ports**.
* The source process will send a message to a port and the destination process receives message from a port.
* Demultiplexing key consists of a `(port, host)` pair.
* For the server, it figures out the client's port by looking at the packet `SrcPort` field.
* For the client, it usually sends to a **well-known** port (e.g. `53`).
* UDP has **no flow-control**: ports are usually implemented as a message queue where messages are appended to the end of the queue.
  * If the queue is full, message is dropped

<center>

![](./assets/UDPHeader.png =300x300)

</center>

<a name="TCP"/>

### 4. TCP
Features of TCP:
* Reliable
* Connection-oriented
* Byte-stream service
* Flow-control mechanisms: prevents senders from over-running receivers (end-to-end issue)
* Congestion-control mechanism: prevents too much data from being injected into the network (conerned with how hosts and networks interact)

Differences between **TCP Sliding Window** and the normal Sliding Window:
* TCP supports logical connections between processes running on any two computers in the Internet, meaning that both sides of the connection must agree to exchange data. 
* TCP has a **teardown** phase: once a connection is lost, we can free state
* TCP has varying RTTs because connection isn't over a physical link, so the timeout mechanism that retriggers retransmissions must be adaptive
* Packets may be reordered as they cross the Internet
* Packets have a **Maximum segment Lifetime (MSL)** that specifies how long a packet can live on the Internet.
* Has flow control by having each side learn how much buffer space is on the other side
* Issue of **Network congestion**: packets might be sent over multiple types of links with varying bandwidth and many sources might traverse through the same slow link.

TCP Segment Format:
* Although TCP supports byte-stream service, it actually forms packets by buffering enough data on the sender side to create a packet and then sends this packet to its peer. The destination host empties the packet content into a receive buffer and reads the data at its leisure.
* Packets exchanged between TCP peers are called **seegments** and has the following header fields
  * `SrcPort`
  * `DstPort`
  * `Ackwowledgment`: carry information about flow of data in the other direction
  * `SecuenceNum`: sequence number for the first byte of data carried in that segment
  * `AdvertisedWindow`: carry information about flow of data in the other direction
  * `Flags`: relay control information between peers
    * `SYN`: establish TCP connection
    * `FIN`: terminates TCP connection
    * `RESET`: signifies that the receiver has become confused (e.g. unexpected segment)
    * `PUSH`: Tells sender to invoke push operation (indicates receiving side that it should notify the receiving process of this fact)
    * `URG`: segment contains urgent data
    * `ACK`: set anytime `Acknowledgement` field is valid
  * `checksum`
  * `HdrLen`: length of header in `32-bit` words
  * **Note**: TCP demux key is given by a 4-tuple: `(SrcPort, SrcIPAddr, DstPort, DstIPAddr)`

<center>

![](./assets/TCPHeader.png =300x300)

</center>

Connection Establishment and Termination:
* Client (caller) does an **active open** to a server (callee) that does a **passive open**. The two sides then exchange messages to establish a connection, and then begin exchanging data. once a participant is done, the connection is closed which initiates **termination messages**.
* Uses the **Three-way Handshake**: 
  1. Client (active participant) sends a segment to the server (passive participant) stating the initial sequence number (`Flags = SYN, SequenceNum = x`)
  2. Server responds with a segment acknowledging client's sequence number (`Flags = ACK, ACK = x + 1`) and states its own beginning sequence number (`Flags = SYN, SequenceeNum = y`).
  3. Client responds with a 3rd segment acknowleding server's sequence number (`Flags = ACK, ACK = y + 1`)
  4. If anything timesout, that segment is retransmitted

<center>

![](./assets/ThreeWayHandshake.png =400x350)

</center>

* Two ways of trigger state transitions in TCP
  * Segment arrives from a peer
  * Application process invokes an operation (e.g. active open)

<center>

![](./assets/TCPState.png =500x500)

</center>

TCP Sliding Window:
* Similar to link level sliding window except that the receiver **advertises** a variable size window (`AdvertisedWindow` header field) to help with flow control. TCP Sliding Window also guarantees:
  * Reliable and Ordered Delivery: buffers on both the sending and receiving side are used to buffer unacknowledged segments and reorder out of order data. To do so, we maintain
    * `LastByteAcked <= LastByteSent`
    * `LastByteSent <= LastByteWritten`
    * `LastByteRead < NextByteExpected`
    * `NextByteExpected <= LastByteRcvd + 1`
  * Flow Control: use `MaxSendBuffer` and `MaxRcvBuffer` to throttle data
    * `LastByteRcvd - LastByteRead <= MaxRcvBuffer`
    * `AdvertisedWindow = MaxRcvBuffer - ((NextByteExpected - 1) - LastByteRead)`
    * `LastByteSent - LastByteAcked <= AdvertisedWindow`
    * `EffectiveWindow = AdvertisedWindow - (LastByteSent - LastByteAcked)` (`EffectiveWindow` limits how much data it can send)
    * `LastByteWritten - LastByteAcked <= MaxSendBuffer`
    * If `y` bytes are written, `(LastByteWritten - LastByteAcked) + y > MaxSendBuffer` then TCP blocks the sending process
    * If `EffectiveWindow > 0`, can send data. Otherwise need to wait for an ACK to free up space
    * **Note**: the sender usually sends a 1-byte message called **Zero Window Probes** to notify the sender when `AdvertisedWindow` is non-zero
  * Issue 1: Fast sender, slow receiver (`LBR` increases slowly, `NBE` increases quickly):`(NBE - 1) - LBR` increases quickly so `AdvertisedWindow` and `EffectiveWindow` decreases to `0`, so the sender **blocks**.
    * `ACK` is sent when receiving any data segment. `ACK` will include (`NBE, AdvertisedWindow`). However, if `AdvertisedWindow == 0`, the sender sends no data and receiver returns no `ACKs`, so the sender can't learn when `AdvertisedWindow` increases.
      * Solution: if `AdvWin == 0`, periodically send a 1B segment to trigger an `ACK` response to trigger sender to expand `AdvWin` and start sending data again.
  * Issue 2: Finite Sequence Numbers:
    * 32b sequence number and 16b `AdvWin` so there are a lot more sequence numbers (good for sliding window)
    * Segments survive at most MSL (120s) so wrap-around within MSL can cause conflicts
  * Issue 3: Small Advertised Windows:
    * `delay * bw = throughput` which is the **largest useful AdvWin**

TCP Congestion Control: what happens when we send `AdvWin` when there is **network congestion**
* Get congestive packet loss, leading to retransmissions. However this just causes more congestion, leading to **congestion collapse**.
* Hosts try to estimate capacity and ACKs trigger new data segments using Nagle's Algorithm. However, the available bandwidth changes as other connections (flows) come and go

AIMD: Additive Increase, Multiplicative Decrease:
* **Congestion Window** (`cwnd`): source-based limit of data-in-flight. This value is based on observations. If there's congestion, then `cwnd` decreases. Otherwise increases.
* `AdvWin = awnd`
* Max unAck'd data: `maxWin = min(cwnd, awnd)`
* `EffWin = maxWin - (LBS - LBA)`
* If timeout for an ACK occurs, we assume it's a congestion-based drop and `cwnd = max(cwnd/2, MSS)`
* When ACK is received, there aren't any congestions and `cwnd = cwnd + MSS*MSS/cwnd`

Slow-start: idea is to ramp up quickly until AIMD is near-optimal
* Start: `cwnd = MSS`
* When recv ACK: `cwnd = cwnd + MSS`
* When packet loss: switch to AIMD
* New connections begin in SS
* If a connection goes dead, then we enter SS until `cwnd` reaches SSthreshold

Fast retransmit: 
* TCP uses duplicate ACKs. If we see the same duplicate 3 times, then retransmit the next segment after the ACK, even without a timeout.

Fast Recovery:
* Uses continued ACKs to clock new sends. So when a fast retransmission occurs, `cwnd = cwnd/2`. Leads to **congestion avoidance** and avoids going back to SS.

TCP Tahoe: after 3 dup ACKs
* do a fast-retransmit
* set `SSThresh = cwnd/2`
* `cwnd = MSS`
* Enter SS

TCP Reno: after 3 dup ACKs
* do a fast-retransmit
* `SSTresh = cwnd/2`
* `cwnd = cwnd/2`
* Start fast recovery

TCP Vegas: RTT-base congestion avoidance
* `ExpectedRate = cwnd/minRTT`
* `D = ExpectedRate-ActualRate`
* If `D < a` then increase `cwnd` linearly
* If `D > b` then decrease `cwnd` linearly

TCP New Reno: higher-throughput version of Reno's fast recovery
* Dup ACKs causes new segment to be sent
* If the ACK makes progress, we assume it points to a hole from loss

TCP CUBIC: function-based congestion control
* If packet loss is found, `cwnd = b*cwnd` where `b < 1`
* Otherwise `cwnd` grows as a cubic function of time since the last back-off
