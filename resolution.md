## DNS Resolution
Goal of DNS is to resolve fully qualified domain names (FQDN) to an IP address
* Recursive query demands a name resolution or the answer "cannot be found". Query is between DNS client and its local DNS server.
* Iterative query doesn't demand a name resolution. This means that other DNS servers may provide a name resolution if they know or simply respond with a referral. Query is between local DNS server and other DNS servers.

#### Header Section
* `ID`: 16-bit request identifier
* `Flags`: 16-bit options
  * `QR`: 1-bit specifying if it's a query (`0`) or a response (`1`)
  * `Opcode`: 4-bit field specifying query type. For this project should be `0` for standard query
  * `AA`:
  * `TC`: 1-bit specifying if the message needs to be truncated.
  * `RD`: 1-bit specifying if recursion is desired
  * `RA`: 1-bit signifying that recursion is available
  * `Z`:
  * `Rcode`: 4-bit signifying any error codes
* `QDCOUNT`: 16-bit unsigned integer specifying number of entries in the question section
* `ANCOUNT`:
* `NSCOUNT`:
* `ARCOUNT`:

#### Question Section
* `QNAME`: variable-bit field that contains URL of IP address we want to find. Encoded as a series of labels (octets) in the format: `<len><octet>...00`. Note that `QNAME` is terminated by `00`
* `QTYPE`: 16-bit field that tells the DNS record type we're looking for
* `QCLASS`: 16-bit field that tells the DNS class we're looking for

#### Answer Section
* `Name`: variable-bit URL who's IP address this response contains
* `Type`: 16-bit DNS record type
* `Class`: 16-bit DNS class type
* `TTL`: 32-bit unsigned integer specifying time to live for this response. Before this time interval runs out, the result can be cached. Afterwards, it should be discarded
* `RDLength`: 16-bit containing byte length of `RDData`
* `RData`: variable-bit contains data we're looking for.
