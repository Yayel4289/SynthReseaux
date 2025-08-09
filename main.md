---
title: Réseaux
markmap:
    spacingVertical: 15
    spacingHorizontal: 100
    colorFreezeLevel: 3
    color:
        - cyan
        - white
        - red 
        - orange 
        - yellow 
        - green 
        - blue
---

## ![](images/layers.png)

### Application

- Exemples : vie de tous les jours
    - e-mail (voir tp 3)
    - web 
    - jeux 
    - streaming
- Termes techniques 
    - URI $\Rightarrow$ Uniform Resource Identifier
    - URL $\Rightarrow$ Uniform Resource Locator 
        - Composed of URI's
- Principes des applications réseaux
    - Client-Server model
        - Server process : **wait** for communication
        - Client process : **initiates** communication 
    - Socket 
        - interface OS/network stack (Transport, Network, ...)
        - choix de protocole de transport (TCP, UDP)
        - Socket programming (tp)
    - Adressing remote process
        - IP adress + Port number
    - Potential Requirements (motivate choice TCP or UDP)
        - ex: reliable data transfer, timing, security, debit, ...

- Protocols
    - Hyper Text Transfer Protocol (Secure) $\Rightarrow$ HTTP(S)
        - Uses URL to 
            - Address Objects
            - Send parameters (ex : "search=lorem", ...)
        - Stateless 
            - Server maintains no info about past clients
        - Client-Server model
        - Uses TCP (important order)
            - Server listens TCP connections
            - Client initiates TCP connection to Server
            - Server Accepts
            - Client sends HTTP request
            - Server sends HTTP response
            - TCP closes
        - Messages
            - ASCII Encoding
            - 2 types 
                - request
                    - Methods (most important ones)
                        - `GET` $\Rightarrow$ retrieve object from Server
                        - `HEAD` $\Rightarrow$ `GET` for debugging, only headers
                        - `POST` $\Rightarrow$ send object to server
                        - `PUT`, `DELETE`, ...
                - response 
                    - Status codes ("X" means 1 digit)
                        - 1XX $\Rightarrow$ Info
                        - 2XX $\Rightarrow$ Success
                        - 3XX $\Rightarrow$ Redirection
                        - 4XX $\Rightarrow$ Client Error (ex : 404)
                        - 5XX $\Rightarrow$ Server Error
            - End delimiter 
                - Content-length header $\Rightarrow$ know where to stop
                - Close connection (when data is transmitted)
                - Transfer-encoding : chunked header $\Rightarrow$ message split in chunks with length + content
                - Implicit length $\Rightarrow$ responses without content, usually error msgs
        - Performances 
            - Def RTT : Round Trip Time $\Rightarrow$ temps pour aller-retour client-server
            - $T$ = time to transmit file
            - Non-persistent HTTP (HTTP/1.0 default behavior)
                - 1 object / TCP connection 
                - <img src="images/application/non_persistent.png" height="200"/>
                - $t = \sum (2 \cdot RTT + T_i)$
            - Persistent HTTP (HTTP/1.1 default behavior)
                - several objects / TCP connection
                - <img src="images/application/persistent.png" height="200"/>
                - $t = RTT + \sum (RTT + T_i)$
                - better with pipelining 
                    - <img src="images/application/pipelining.png" height="200"/>
                    - $t = 2 \cdot RTT + \sum T_i$

        - Cookies 
            - Keep state identification
                - How ? ID in HTTP request/response (+ database)
        - Proxy Server / Web Cache
            - Installed by local ISP
            - Acts as Client & Server
            - Browser forced to access Proxy 
            - Proxy works as an OS cache
                - if object in cache and not modified since <last date> : response to Client
                - else : request to origin Server
            - Avoid request to Server 
                - Objective : reduce RTT
                - Solution : optional expiration date in header
            - Avoid full response to Client
                - Objective : reduce bandwidth usage
                - Solution : conditional GET $\Rightarrow$ If-modified-since <date>
        - HTTP/2.0 
            - Multiplex concurrent requests in 1 TCP connection + resource prioritization
                - <img src="images/application/multi_prior.png" height="150"/>
            - Compress headers $\Rightarrow$ Binary Encoding
            - Supports Server Push
                - If 1 pushed object requires other objects $\Rightarrow$ Push Promise
                - Ex : `index.html` pushed and requires `style.css`, `script.js`, ...

    - Simple Mail Transfer Protocol $\Rightarrow$ SMTP 
        - Uses TCP
        - Commands ($\approx$ requests) in ASCII
        - Responses : status code + phrase
        - Transfer 
            - Handshake 
            - Message transfer 
            - Closure
        - Mail access protocols 
            - <img src="images/application/mail_access_protocols.png" height="75"/>
            - Post Office Protocol $\Rightarrow$ POP 
                - POP3 tp
            - Internet Mail Access Protocol $\Rightarrow$ IMAP
            - HTTP

    - Domain Name System $\Rightarrow$ DNS
        - Servers and Structure
            - <img src="images/application/dns.png" height="200"/>
            - Exists because a centralized DNS is bad 
                - Single point of failure 
                - Traffic Volume 
                - Distant from everything,
                - ...
            - Top-Level Domains $\Rightarrow$ TLD
                - com
                - edu 
                - org 
                - ...
        - Resource Records (also RR)
            - = entry in DNS database
            - Format : domain name, value, type, TTL (time-to-live)
            - Allows future evolution
        - Each Server maintains cache
        - Local Name Server 
            - Each local ISP has at least one
            - $\approx$ Proxy
        - Map : IP adress $\Leftrightarrow$ Hostname
            - Hostname $\Rightarrow$ IP adress : Name Resolution
                - Recursive Name Resolution
                    - <img src="images/application/recursive_name_resol.png" height="100"/>
                - Iterative Name Resolution
                    - <img src="images/application/iterative_name_resol.png" height="100"/>
            - IP address $\Rightarrow$ Hostname : Reverse Lookup
                - IP addresses translated to string and searched (from end always)
                    
        - DNS tool $\Rightarrow$ `dig` (tp)


### Transport

- Termes techniques
    - Port number 
        - identifies process communication end point (HTTP, ...)
        - Used by Transport Protocols

    - Checksum
        - let $n_i$ the components of a segment 
        - the segment keeps $CS = \neg \sum n_i \Rightarrow CS + \sum n_i =$ only 1's
        - so can tell if corrupted bit
    
- Reliable Data Transfer (RDT) Principles
    - try to understand / redo some state machines $\Rightarrow$ potential exam question
    - Assumptions
        - Everything works fine $\Rightarrow$ RDT1.0
            - Ideal (not realistic) data transfer 
        - Bits may flip $\Rightarrow$ RDT2.0
            - Receiver verify msgs with Checksum and send : 
                - ACK(nowlegments) $\Rightarrow$ pkt OK
                - NAK (Negative AcK) $\Rightarrow$ pkt corrupted
            - <img src="images/transport/rdt20.png" height="100"/>
        - ACK / NAK may be corrupted $\Rightarrow$ RDT2.1
            - Alterning sequence number 0, 1 to recognize duplicates (only 2 bc stop & wait)
            - Checksum on ACK / NAK too
            - <img src="images/transport/rdt21.png" height="100"/>
        - NAK-free $\Rightarrow$ RDT2.2
            - Always ACK the sequence number
            - <img src="images/transport/rdt22.png" height="100"/>
        - Pkt loss may happen $\Rightarrow$ RDT3.0
            - timeout for ACK's
            - <img src="images/transport/rdt30.png" height="100"/>
            - base perf stinks $\Rightarrow$ solution :
                - sequence number : 0 $\rightarrow$ n
                - pipelining : 2 ways 
                    - Go-Back-N
                        - discard out of order pkts if pkt loss
                            - <img src="images/transport/go_back_n.png" height="100"/>
                        - ACK loss but sequence number $\nearrow \  \Rightarrow$ Sender knows Receiver ACK'd 
                            - <img src="images/transport/go_back_n2.png" height="100"/>
                        
                    - Selective repeat
                        - just timeout for pkt / ACK loss
                            - <img src="images/transport/selective_repeat.png" height="100"/>
                            - <img src="images/transport/selective_repeat2.png" height="100"/>
                            
                        - but problem : cyclic number can be misinterpreted
                            - <img src="images/transport/selective_repeat_dilemma.png" height="100"/>
                            - solution $\Rightarrow 2^M \geq 2N$ (bc worst case is Receiver one window in advance of Sender)

- Protocols 
    - Transmission Control Protocol $\Rightarrow$ TCP 
        - Segment structure
            - Maximum Segment Size $\Rightarrow$ MSS 
                - restrained by MTU (Maximum Transmission Unit)
                    - <img src="images/transport/mss.png" height="70"/>
                - MSS Trade-off
                    - if MSS too large $\Rightarrow$ fragmentation (IP datagrams might be too large for certain links)
                    - if MSS too low $\Rightarrow$ too much TCP/IP header overhead
                - MSS negotiation 
                    - TCP can announce MSS to incoming TCP segments to readjust
                - Nagle's algo 
                    - solution for small-pkt problem (small msgs apps)
                    - functioning
                        - if no coming ACK : send immediately 
                        - else : send when 
                            - previous pkt is ACK'd 
                            - segment size reached MSS
            - Content 
                - source port nbr 
                - dest port nbr 
                - sequence nbr 
                - ACK nbr 
                - flags (ACK, etc)
                - checksum 
                - options (SACK, MSS, etc)
                - receive windown `rwnd` (flow control)
                - ...
        - Reliable data transfer 
            - Retransmission TimeOut $\Rightarrow$ RTO 
                - Depends of RTT $\Rightarrow$ need to constantly estimate RTT
                    - $R_i$ = time between seg transmission and its ACK
                    - Smoothed RTT $\Rightarrow SRTT$
                    - Exponentially Weighted Moving Average $\Rightarrow$ EWMA
                        - weight $\Rightarrow \alpha$ ( = 0.125 usually)
                        - inductive form 
                            - $SRTT_0 = R_0$
                            - $SRTT_i = (1 - \alpha) SRTT_{i-1} + \alpha R_i$
                        - general form (ok to retrieve from inductive)
                            - $SRTT_i = \sum_{k=0}^i \alpha (1 - \alpha)^{i - k} R_k = \alpha \sum_{k=0}^i (1 - \alpha)^{i - k} R_k$
                        - not enough to follow RTT !
                    - Safety margin 
                        - weight $\Rightarrow \beta$ ( = 0.25 usually)
                        - inductive form 
                            - $DevRTT_0 = R_0 / 2$
                            - $DevRTT_i = (1 - \beta) DevRTT_{i - 1} + \beta | SRTT_i - R_i |$
                        - general form (as ok as SRTT)
                            - $DevRTT_i = \sum_{k = 0}^i \beta (1 - \beta)^{i - k} |SRTT_k - R_k| = \beta \sum_{k = 0}^i (1 - \beta)^{i - k} |SRTT_k - R_k|$
                    - $RT0_i = SRTT_i + 4 DevRTT_i$
                        - if timer expires, $RTO \leftarrow 2 RTO$
            - Fast retransmit
                - <img src="images/transport/fast_retransmit.png" height="100"/>
                - Detect lost segments via duplicates (3) ACKs
                - Directly resend 
        - Flow control
            - Matches sender speed and receiver drain
            - How ? receive window `rwnd` var sent by receiver
                - if `rwnd = 0` : stop sending
                - if `rwnd > 0` : send `rwnd` segment
                - sender might not receive `rwnd` info
                    - send window probe (= sonde) every *persist timer* time
            - Silly Window Syndrome 
                - Risk of slowing down a lot 
                - Ex : `rwnd = 1` and 1 byte to be sent constantly
                - Solutions 
                    - Avoid annoucing `rwnd < MSS` (receiver part)
                    - OR
                    - Avoid sendind too small segments (sender part) $\Rightarrow$ Nagle's algo
            - $Throughput \approx \frac{rwnd}{RTT}$
            - Options to scale `rwnd` to `rwnd`$\cdot 2^N$
        - Connection Managment 
            - 3-way handshake
                - <img src="images/transport/three_way.png" height="100"/>
            - closing 
                - <img src="images/transport/closing.png" height="100"/>
            - TCP connection management state machine 
                - <img src="images/transport/tcp_sm.png" height="100"/>
        - Congestion Control
            - To avoid to overwhelm the network (routers, link, ...) (!= flow ctrl)
            - let `cwnd` adapting var for congestion window
                - Slow Start
                    - `cwnd = 1*MSS`
                    - `cwnd += nbr_ACK*MSS` **every RTT** (exponential)
                - Additive Increase, Multiplicative Decrease (AIMD)
                    - let `ssthresh` slow start threshold(=seuil)
                        - at beginning : set to large value
                        - after loss : `ssthresh = cwnd / 2`
                    - AIMD activates when `cwnd >= ssthresh` $\Rightarrow$ Congestion avoidance
                    - `cwnd += 1*MSS` **every RTT**
                    - if loss (detected by 3 dup ACKs) $\Rightarrow$ Fast recovery
                        - `cwnd = ssthresh = cwnd / 2`
                    - if timeout (no ACK for too long = congestion) 
                        - `ssthresh = cwnd / 2`
                        - `cwnd = 1*MSS` (Slow start again)
                    - Throughput (congestion avoidance)
                        - <img src="images/transport/tcp_debit.png" height="100"/>
        - Selective ACK (SACK)
            - reduces retransmission duplicates
            - ex 
                - pkts sent : 1 2 3 4 5 6 7 but 3 4 lost ! 
                - Receiver sends ACK(2) AND SACK 5-7
                - Sender retransmits 3 4 instead of 3-7
        - Fair due to AIMD
    - User Datagram Protocol $\Rightarrow$ UDP
        - connectionless $\Rightarrow$ no handshake
        - uses Checksum
        - \+ 
            - fast (ex : no handshake)
        - \- 
            - unreliable data transfer
        - why if tcp exists ? video games, streaming $\Rightarrow$ when time is more important

### Network

- Protocols
    - Internet Protocol $\Rightarrow$ IP
        - Format 
            - <img src="images/network/ip_header.png" height="100"/>
        - Fragmentation 
            - flags in header 
                - Don't Fragment $\Rightarrow$ DF 
                - More Fragment $\Rightarrow$ MF 
                - unused
            - for exercises, remember IP header = TCP header = 20B
        - Addressing
            - Classful addressing
                - <img src="images/network/ip_addr_class.png" height="100"/>
            - Subnetting 
                - \>\< Classful addressing 
                - idea : take an IP address and share it
                - prefix-length (bc no class)
                    - distinguish net and host 
                    - ex : `171.18.32.36/19`
                        - `mask = 255.255.224.0`
                        - `net = 171.18.32.0`
                        - `host = 0.0.0.36`
                - Formules 
                    - `net = addr & mask`
                    - `host = addr & !mask`
                    - `(other_addr & mask) == (addr & mask)` (to see if `addr` and `other_addr` are in the same subnet)
                    - `(other_addr ^ addr) & mask == 0` (same)
                - the subnet is made in the host part
                - when subnetting, the host part can't have 
                    - the lowest addr $\Rightarrow$ network addr
                    - the highest addr $\Rightarrow$ broadcast
                    - doesnt apply for links since smart routers on ends
        - Forwarding
            - Forwarding table
                - <img src="images/network/forwarding_table.png" height="100"/>
            - Longest Prefix Matching (LPM) (self exp)
                - Software Implementation with b-tree (trie)
                    - bit = 0 $\Rightarrow$ Left
                    - bit = 1 $\Rightarrow$ Right
                    - <img src="images/network/software_lpm.png" height="100"/>
                - Hardware Implementation with Ternary Content-Addressable Memory (TCAM)
                    - <img src="images/network/hardware_lpm.png" height="100"/>
                    - Ternary
                        - 0, 1 or x (might be both)
                    - Content-Adressable Memory 
                        - Search to all entries in parallel $\Rightarrow$ perf++
        - How to get IP address ?
            - networks 
                - <img src="images/network/hierarchy.png" height="100"/>
                - bottom network gets allocated addr space from above
                - easier forward IP addr (unless user changes IPS for example)
            - users 
                - static config 
                    - by network admin
                    - OS can handle
                - dynamic congig 
                    - link-local addresses
                        - automatic random addr given
                    - DHCP (best)
        - Network Address Translation $\Rightarrow$ NAT
            - Map network (green) and local (red) IP addresses 
                - <img src="images/network/nat.png" height="100"/>
            - Problem 
                - the local address isnt visible externally
                - Solutions but controversial
                    - violates end-to-end
                    - break some application layer protocols
        - IPv6
            - motivation 
                - ipv4 almost completely allocated 
                - perf++
            - how to transition ? 
                - Tunneling 
                    - <img src="images/network/tunnel.png" height="100"/>
                    

    - Dynamic Host Configuration Protocol $\Rightarrow$ DHCP
        - allows host to dynamically optain IP address
        - uses UDP
        - functioning 
            - client asks for IP addr to servers
                - <img src="images/network/dhcp_discover.png" height="100"/>
            - 3 timers starts if connected : `T1 < T2 < Lease`
                - `T1 expires` $\Rightarrow$ request to same server
                - `T2 expires` $\Rightarrow$ request all accessible servers ('needs help' bc its serv left it)
                - `Lease expires` $\Rightarrow$ deconnection
        - can use relays
        - relies a lot on options encoded in TLV

    - Internet Control Message Protocol $\Rightarrow$ ICMP 
        - error / reachability reporting
        - multiple pkts sent with TTL = 1,2,3,... until dst reached
        - <img src="images/network/icmp.png" height="100"/>
        - can also discover MTU and notice src




- Router Architecture 
    - <img src="images/network/router_arch.png" height="100"/>
    - Switching fabrics 
        - Memory (1st gen)
            - Architecture like computers 
            - pkts copied to memory 
            - CPU controls switching
        - Bus (2nd gen)
            - Fwding cache / line + CPU $\Rightarrow$ faster
            - <img src="images/network/bus.png" height="100"/>
        - Crossbar (3rd gen)
            - <img src="images/network/crossbar.png" height="100"/>
            - can be difficult to addess $N \times M$ points
    - Head-Of-Line (HOL) blocking 
        - <img src="images/network/hol.png" height="100"/>
        - solution $\Rightarrow$ split input into queues for != outputs
                
- Routing algorithms 
    - Link State Routing Protocol
        - routers 'know' the whole map
        - Link State Packet (LSP) Flooding
            - Each router sends an `LSP`, containing 
                - its `id`
                - the costs to reach its neighbors
                - `seq_nbr` (increment every LSP flood)
                    - allows to deal with link failures / changes
                - `age` (decrement regularly)
                    - `if age == 0: del LSP`
                    - allows to deal with router failures / reboot
                    - LSP flooding required sometimes
        - Link State DataBase (LSDB)
            - ```py
                # when router receives LSP
                if (LSP.id not in LSDB) or (LSP.seq_nbr > LSDB[LSP.id].seq_nbr):
                    LSDB[LSP.id] = LSP
                    send_to_neighbors(LSP)
            ```
        - Dijkstra
            - variables 
                - `c(x, y)` : cost from `x` to `y`
                - `D(v)` : current cost (estimation) from src to `v`
                - `p(v)` : current predecessor (estimation) to reach `v`
                - `N'` : set of nodes whose min cost path is def known
            - relaxation 
                - ```py
                    def relax(u, v):
                        if D(u) + c(u, v) < D(v):
                            D(v) = D(u) + c(u, v)
                            p(v) = u
                ```
            - algo 
                - ```py
                    def dijkstra(G=(V,E), src):
                        D(src) = 0
                        N' = set()
                        for v in V - {src}:
                            D(v) = inf 
                            p(v) = None

                        while (N' != V):
                            find (u not in N') with D(u) is minimum
                            N'.add(u)
                            for v in u.neighbors:
                                relax(u, v)
                ```

    - Distance Vector Protocol
        - routers only 'know' their neighbors
        - Distance Vector (DV) : costs / dest obtained by neighbors
        - Bellman-Ford equation 
            - $d_x(y) = \min _v (c(x, v) + d_v(y))$
            - $d_v(v) = 0$ (base)
        - each router $x$
            - maintains
                - its own DV $D_x = D_x(y) | y \in V$
                - its neighbors' $v$ DVs $D_v(y) | y \in V$
            - update its own DV using Bellman-Ford when 
                - neighbors send their updated DV
                - local links fail / change cost ($c(x, v)$)
            - share its own DV after update
        - count-to-infinity problem 
            - illustrations 
                - <img src="images/network/count_to_inf/1.png" height="100"/>
                - <img src="images/network/count_to_inf/2.png" height="100"/>
                - <img src="images/network/count_to_inf/3.png" height="100"/>
                - <img src="images/network/count_to_inf/4.png" height="100"/>
            - explanations
                - $y$ announces to $z$ a cost for a route to $x$ that goes through $z$ !
                - so $z$ updates its DV and sends it to $y$
                - ...
            - solution 
                - poisoned reverse 
                    - <img src="images/network/count_to_inf/pr.png" height="100"/>


- Switching 
    - Def : defines how a network element forwards data with its header
    - Circuit Switching or Packet Switching

    
- Network Structure
    - ISP : Internet Service Provider (Fournisseur d'accès internet)
    - Tier 1,2,3/local $\Rightarrow$ chaque tier paye au-dessus (tier 3 = Proximus, local can be univ, etc)

### Link
- Comment transférer data ?
    - Circuit Switching : Réservation d'un circuit pour transférer
        - Resources : link bandwidth + switch capacity (not ability)
        - Ressources dédiée / 1 circuit $\Rightarrow$ perf++
        - Multiplexing
            - Share 1 link among N circuits
                - Frequency Division Multiplexing (FDM)
                    - <img src="images/intro/fdm.png" height="100"/>
                    - Comment ? Signal porteur $f_c$
                - Time Division Multiplexing (TDM)
                    - <img src="images/intro/tdm.png" height="100"/>
                    - Comment ? RR-like algo
                - $R_{new} = \frac{R}{N}$
    - Packet Switching : division data en packets
        - 1 lien / packet
        - delay 
            - variables 
                - $L$ packet length (bits b)
                - $R$ link bandwidth (b/s)
                - $d$ physical link length (m)
                - $s$ propagation speed (m/s)
            - formules et données
                - $d_{trans} = \frac{L}{R}$ (transmission delay)
                - $d_{propa} = \frac{d}{s}$ (propagation delay)
                - $d_{proc}$ (processing delay, often negligible)
                - $d_{queue}$ (queuing delay)
                - $d_{nodal} = d_{trans} + d_{propa} + d_{proc} + d_{queue}$
        - debit : $R \leq min(R_i) \Rightarrow$ bottleneck 
        - store and forward $\Rightarrow$ long transmission delay
            - solution : pipelining 
                - <img src="images/intro/pipelining.png" height="100"/>
                - $d_{trans} = (\sum \frac{L}{R_i}) + (N - 1) \frac{L}{R_{min}}$
                - packet max size
                    - msg size = $M$ bits
                    - header size = $H$ bits
                    - max packet size = $K_{max}$ bits
                    - number of packets: $n = \lceil \frac{M}{K_{max}} \rceil$
                    - Number of bits to transmit = $M + H n$
                    - N links
                    - Time $T$ 
                        - without pipelining : $T = N (\frac{H + M}{R})$
                        - with pipelining : $T = \frac{1}{R} ((N - 1) K_{max} + M + H n)$
                            - why ? 
                                - $T = T_1 + T_2$ where 
                                    - $T_1 = \frac{1}{R} (M + H n) \Rightarrow $ time to transmit all bits 
                                    - $T_2 = \frac{1}{R} ((N - 1) K_{max}) \Rightarrow $ it "slides" bc of pipelining
                    - Average number of packets: $\overline{n} = \frac{\overline{M}}{K_{max}} + \frac{1}{2}$
                    - $\overline{T} = \frac{1}{R} ((N - 1) K_{max} + \overline{M} + H \overline{n})$
                    - $K_{max} \approx \sqrt{\overline{M} \frac{H}{N - 1}}$
        - queueing
            - M/M/1 queueing model (theorical)
                - M/M/1 $\Rightarrow$ Markovian x 2, 1 infinite q
                - $\lambda$ avg pkt **arrival** rate (pkts/s)
                - $\mu$ avg pkt **departure** rate (pkts/s)
                - $\rho = \frac{\lambda}{\mu}$ intensity (stable if < 1, 'inf' delay else)
                - $\overline{N} = \rho (1 - \rho)$ avg nbr of pkts in q
                - $\overline{d_{queue}} = \overline{N}L/R = \overline{N} d_{trans}$
                    - in exercises, $\overline{d_{queue}}$ should involve these vars
            - delay depends on pkt arrival $\Rightarrow$ exos
            - packet loss (q has finite capacity)

- Error detection 
    - Checksum (alreay seen in RDT)
    - Single-bit Parity checking 
        - focus on number of 1's
        - even-parity
            - makes the sequence even
            - ex
                - 001010010**1**
                - 001000010**0**
        - odd-parity 
            - makes the sequence odd
            - ex 
                - 001000010**1**
                - 001010010**0**
    - Cyclic redundancy codes (de degré $k$)(CRC-$k$)
        - detection++, overhead--
        - Principe 
            - binary seq (ex : 1010) $\Rightarrow$ binary polynomial ($x^3 + x$)
            - binary polynomials arithmetics 
                - \+, \- : XOR
            - let polynomials
                - $M(x)$ (Message) deg $n$ (msg to send)
                - $G(x)$ (Generator) deg $k$ sender and receiver agree on
                - $R(x) = M(x) \cdot x^k \% G(x)$ 
                - $C(x) = M(x) \cdot x^k + R(x) \Rightarrow$ the sent Codeword
                    - if $C(x)$ exactly divisible by $G(x)$ : ok
                    - else : error

- Medium Access Control (MAC) Protocols 
    - For sharing the channel
    - if frame collision at a station : unreadable
        - <img src="images/link/collision.png" height="20"/>
    - 3 types
        - Channel Partitionning (FDM, TDM, ...)
        - Random Access
            - allows collision and recover from them
            - ALOHA
                - works with ACKs, if no ACK (collision) : retransmit
                - collisions with frame overlaps
            - Slotted ALOHA
                - \+ increase success rate with time slots
                - \- require clock sync
            - Carrier Sense Multiple Access (CSMA)
                - also works with ACKs, if no ACK (collision) : retransmit
                - listen before transmit
                    - if channel idle : transmit 
                    - if channel busy : wait
                - cons : time is required to detect business
            - CSMA/CD (collision detection)
                - minimum frame size 
                - no ACK
                - $d_{trans} >= 2 d_{propa} \Rightarrow$ all collisions detected
        - Taking turns
            - master manages turns (bad)
            - token passing (better)

- Link layer address
    - = Ethernet address = MAC address
    - **only used within a Local Area Network (LAN)**
    - Format 
        - 48 bits long, hex for humans
        - 2 parts 
            - Oranizationnally Unique Identifier $\Rightarrow$ OUI
                - 1st bit $\Rightarrow$ I/G
                    - 0 = Individual (unicast)
                    - 1 = Group (broad/multi cast)
                - 2nd bit $\Rightarrow$ U/L
                    - 0 = Universal
                    - 1 = Local
            - Vendor-specific part 
    - Address Resolution Protocol $\Rightarrow$ ARP
        - IP $\Leftrightarrow$ MAC address mapping 
        - maintains cache (IP addr, MAC addr, TTL)
        - <img src="images/link/arp.png" height="100"/>

- Ethernet (protocol)
    - frame with src, dst, payload,... of course
    - Connectionless : no handshake
    - Unreliable (but very good)
    - uses CSMA/CD
        - but with exponential backoff 
            - after the $m^{th}$ collision, wait time at random up to $2^m-1$
            
- Switch (=commutateur)
    - used in a star topology (> bus topology)
    - maintains a switch table (MAC addr, Port, TTL)
    - no routing protocol, self-learning ! 
        - ```py
            def receive_frame(frame, input_port):
                # frame.src, frame.dst => MAC addr
                # port => switch I/O

                associate(frame.src, input_port) # only way to learn port/addr

                if output_port found for frame.dst:
                    if (input_port == output_port):
                        delete(frame) # filtering since src == dst
                    else:
                        forward(frame, output_port)
                else:
                    flood(frame) # forward on all except input_port
        ```
    - Switch vs Router 
        - Router 
            - Network layer 
            - Routing algo 
            - Between subnets
        - Switch 
            - Link layer 
            - self learning
            - in subnets

### Physical <!-- markmap: fold -->

- Termes techniques
    - Modem : modulateur - démodulateur de signaux (analogique $\Leftrightarrow$ digital)
    - Baseband : Données à transmettre, fréquence notée $f_b$
    - Carrier signal / Signal porteur : Signal servant à transporter des données, fréquence notée $f_c$
    - Passband / Bande passante : Signal modulé à partir de Baseband et Carrier signal, avec $f_b$ <<< $f_c$

- Modulations Digitales
    - M : nombre de symboles (QPSK, QAM)
    - N : nombre de bits encodés par symbole, $N = log_2(M)$
    - Symbol / Baud rate : $R_s = \frac{1}{T_s}$
    - Bit rate : $R = \frac{N}{T_s} = N R_s$
    - <img src="images/intro/digital_modulations.png" height="100"/>
    - <img src="images/intro/qpsk.png" height="100"/>
    - QAM <img src="images/intro/qam.png" height="100"/>
    - plus efficace (avec noise) : PSK, QPSK

- Transmission impairments
    - Limited Bandwidth :
        - $f$ max imposée $\Rightarrow$ déformation signal
        - <img src="images/intro/limited_bandwidth.png" height="100"/>
    - Attenuation (with distance)
        - $P_{tx}$ et $P_{rx}$, resp. puissance des signaux transmis et reçus
        - $Attenuation = \frac{P_{tx}}{P_{rx}}$
        - $Attenuation_{dB} = 10 log_{10}(\frac{P_{tx}}{P_{rx}})$
    - Delay distortion 
        - <img src="images/intro/delay_distortion.png" height="100"/>
    - Noise (Bruit)
        - Origines
            - Thermal noise : mvmt électrons
            - Intermodulation : interférence avec modulateurs alentours (gérable easy)
            - Impulse noise
        - Signal-to-Noise Ratio, $SNR = \frac{S}{N}$ où S, N puissance resp. du signal et du noise
        - SNR++ $\Rightarrow$ perf++
        - impact QPSK
            - <img src="images/intro/noise_qpsk.png" height="100"/>

- Theorical channel capacity
    - Noise free channel 
        - $R = 2B$ (où B est la Bandwidth)
        - $C = 2B log_2(M)$ (général)
    - Noisy channel
        - $C = B log_2(1 + \frac{S}{N})$


## Protocols

- Ensemble de règles permettant de communiquer (syntaxes, type de msg, ...)
- Encoding 
    - Should follow 
        - Space Efficiency 
        - Ease of Delimiting
        - Ease of Parsing (trad msg)
        - Data Transparency (must have the ability to carry **anything** $\Rightarrow$ /!\ Delimiting)

    - Examples 
        - Binary Encoding 
            - fixed group of bits for msg fields
            - <img src="images/application/binary.png" height="100"/>
            - compact++
            - flexible-
            - ex : DNS, HTTP/2.0, TCP, UDP, IP, Ethernet 
        - Type-Length-Value (TLV) Encoding
            - <img src="images/application/tlv.png" height="100"/>
            - compact+
            - flexible-
            - ex : IP, TCP, (DHCP), ... 
        - Match Tag (ASCII Encoding)
            - Just text with delims
            - <img src="images/application/match_tag.png" height="100"/>
            - compact-
            - flexible++
            - ex : HTTP/1.1 headers, ...

