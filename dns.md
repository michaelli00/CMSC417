# Domain Name Service
**Name space**: defines set of possible names hosts can have
* **Flat**: names are not divisible
* **Hierarchical**: names are divisible

Naming system maintain a collection of **bindings** of names to values (usually the address)
**Resolution mechanism**: when invoked with a name, returns the corresponding value
* **Name server**: specific implementation of a resolution mechanism. It is available on a network and can be queried

**DNS Process**:
1. User presents a hostname to an application
2. Program engages the naming system to translate the given name to a host address
3. Application opens a connection to this host by some protocol (e.g. TCP) using host IP address

**DNS Hierarchy**:
* DNS names are read right to left, using periods as separators
* DNS maps domain names into values (not strictly host names into host addresses)

**Name Servers**:
* Domain hierarchy is partitioned into **zones**. Information in each zone is implemented in 2 or more other name servers, so DNS can be treated as a hierarchy of name servers that a client can query for.
* **resource records**: `(Name, Value, Type, Class, TTL)`
  * `Type` specifies how `Value` should be interpreted.
    * Type `A` indicates that `Value` is an IP address
    * Type `NS` indicates that `Value` gives the domain name for a host that is running a name server that knows how to resolve names within the specified domain
    * Type `CNAME` indicates that `Value` gives the canonical name for a particular host (used to define aliases)
  * `Class` allows entities to define record types. (e.g. Class `In`)
  * `TTL` shows how long the resource record is valid

**Name Resolution**
* Client sends several queries to several name servers until it finds a match `NS` record then recurses down until it finds an `A` record.

**Other Terminology**:
* **Resolver**: client that asks questions to the nameserver about hostnames

**DNS Query Flow**:
1. OS tries to resolve address locally. If not found then a request is made to the recursive server
2. Recursive server checks cahce. If it doesn't find the record, then request to root servers is made.
3. If root server doesn't know the answer but it sends a referral to one of the recursive name server in the form of `NS` records
4. Recursive namespace picks a random nameserver and sends off a query to find an answer
5. Answer is found and send to the client

**Resolver Implementation**:
1. Transform client request into a search specification
