## Transport Layer
### Reliable Transmission
Accomplished using
* **Acknowledgements (ACK)**: frame a protocol sends to its peer saying it received the earlier frame
* **timeouts**: if the sender does not receive an ACK from its peer in a reasonable amount of time, it will **retransmit** the original frame

ACKs and timeouts are used to implement **Automatic Repeat Requests (ARQ)**
* **Stop-and-Wait** algorithm: after transmitting a frame, the sender waits for an ACK before transmitting the next frame. If a timeout occurs, the sender retransmits the original frame
  * Potential issue of retransmitting the same frame b/c either the ACK was lost or timeout occured too early. This causes duplicate copies of the frame to be delivered. To remedy the issue, stop-and-wait protocol includes a 1-bit **sequence number** (either 1 or 0 if it's a retransmit)
  * Shortcoming comes from only allowing sender to transmit 1 frame at a time through the link, which may be lower than the link's capacity
