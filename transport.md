# Transport Layer
1. [Internet Control Message Protocol (ICMP)](#ICMP)
2. [Reliable Transmission](#ReliableTransmission)
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

