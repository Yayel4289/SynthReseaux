# Réseaux

## <img src="images/layers.png" height="200"/>

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

    - Domain Name System $\Rightarrow$ DNS
        - Resource Records (also RR)
            - = entry in DNS database
            - Format : domain name, value, type, TTL (time-to-live)
            - Allows future evolution
        - Each Server maintains cache
        - Local Name Server 
            - Each local ISP has at least one
            - $\approx$ Proxy
        - Top-Level Domains $\Rightarrow$ TLD
            - edu 
            - org 
            - ...
        - Map : IP adress $\Leftrightarrow$ Hostname
            - Hostname $\Rightarrow$ IP adress : Name Resolution
                - Recursive Name Resolution
                    - <img src="images/application/recursive_name_resol.png" height="100"/>
                - Iterative Name Resolution
                    - <img src="images/application/iterative_name_resol.png" height="100"/>
            - IP address $\Rightarrow$ Hostname : Reverse Lookup
                    
        - Servers and Structure
            - <img src="images/application/serv_struct.png" height="100"/>
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
    - Assumptions
        - Everything works $\Rightarrow$ RDT1.0
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
                - pipelining : 2 ways 
                    - Go-Back-N
                        - sequence number : 0 $\rightarrow$ n
                        - discard out of order pkts if pkt loss
                        - <img src="images/transport/go_back_n.png" height="100"/>
                    - Selective repeat
                        - $\approx$ Go-Back-n but doesn't discard out of order pkts
                        - <img src="images/transport/selective_repeat.png" height="100"/>
            
- Protocols 
    - Unavailable Services 
        - delay guarantees
        - bandwidth guarantees
    - De/Multiplexing $\Rightarrow$ by header
    - Concrete Protocols
        - Transmission Control Protocol $\Rightarrow$ TCP 
            - Nagle's algo 
                - MSS $\Rightarrow$ Maximum Segment Size
                - algo $\Rightarrow$ fill full segment before sending 
                    - full segment sent = send window $\Rightarrow$ "sliding window"
                    - exception with unACK'd segments : push immediately
            - Constant estimation of RTT for RTO (Retransmission TimeOut)
            - Reliable data transfer 
                - Fast retransmit 
                    - Detect lost segments via duplicates (3) ACKs
                    - Directly resend
            - Flow control
                - Match Edge service speed (no overwhelm to receiver)
                - How ? receive window `rwnd` var sent by receiver
                    - if `rwnd = 0` : stop sending
                    - if `rwnd > 0` : send `rwnd` segment
                - Silly Window Syndrome 
                    - Risk of slowing down a lot 
                    - Ex : `rwnd = 1` and 1 byte to be sent *constantly*
                    - Solutions 
                        - Avoid too small `rwnd` (receiver part)
                        - Avoid sendind too small segments (sender part)
            - Connection Managment 
                - 3-way handshake
                - <img src="images/transport/three_way.png" height="100"/>
            - Congestion Control
                - != Flow control !
                - To avoid to overwhelm the network (routers, link, ...)
                - Ex : bottleneck link
                - let `cwnd` adapting var for congestion window : how ?
                    - Slow Start
                        - Fast Actually $\Rightarrow$ exponential
                        - `cwnd += 1*MSS` every **ACK**
                    - Additive Increase, Multiplicative Decrease (AIMD)
                        - let `ssthresh` slow start threshold(=seuil)
                            - at beginning : set to large value
                            - after loss : `ssthresh = cwnd / 2`
                        - AIMD activates when `cwnd == ssthresh`
                        - Linear
                        - `cwnd += 1*MSS` every **RTT**
                        - `cwnd /= 2` if loss
                            - detected after 3 dup ACKs $\Rightarrow$ Fast Recovery
                    - if timeout : `cwnd = 1` and Slow Start

            - Selective ACK (SACK) : if 1 pkt loss but next ones received $\Rightarrow$ notify

            - Fair due to AIMD

            - Multiplexing : `header = (srcIP, dstIP, srcPort, dstPort)`

            - $\overline{Throughput} = \frac{\overline{W}}{RTT}$ ($\overline{W}$ is avg window size)
            
            - mnemo : Tu Communique Précautionneusement

        - User Datagram Protocol $\Rightarrow$ UDP
            - connectionless $\Rightarrow$ no handshake
            - uses Checksum
            - \+ 
                - fast (ex : no handshake)
            - \- 
                - unreliable data transfer
            - why if tcp exists ? video games, streaming $\Rightarrow$ when time is more important

            - Multiplexing : `header = (dstIP, dstPort)`

            - mnemo : Ultra Direct Pratique

### Network

- Protocols
    - Internet Protocol $\Rightarrow$ IP
        - IP Adress $\Rightarrow$ identifies host device
            - ex : gaia.cs.umass.edu $\Rightarrow$ 128.119.245.12
        - Fragmentation 
            - links have a MTU $\Rightarrow$ Maximum Transfer Size
            - if datagram length > link MTU
                - datagram Fragmentation 
                - datagram reassembled at dest
        - Addressing and Subnetting $\Rightarrow$ TP
            - <img src="images/network/ip_addr_class.png" height="100"/>
            - Formules 
                - `net = addr & mask`
                - `host = addr & !mask`
                - `(other_addr & mask) == (addr & mask)`
                - `(other_addr ^ addr) & mask == 0`
        - Forwarding
            - Forwarding table
                - <img src="images/network/forwarding_table.png" height="100"/>
            - Longest Prefix Matching (LPM) (self exp)
                - Implementation with b-tree
                    - bit = 0 $\Rightarrow$ Left
                    - bit = 1 $\Rightarrow$ Right
                    - <img src="images/network/lpm.png" height="100"/>
            - Subnetting $\Rightarrow$ TP
            

- Router 
    - Routing algorithms 
        - Def : determining path taken by pkts from src to dest
        - 2 types 
            - Link State Routing Protocol
                - Link State Packet (LSP) Flooding
                    - Construction of Link State DataBase (LSDB)
                - Dijkstra algo (voir slides)
            - Distance Vector Protocol (voir projet)

- Switching 
    - Def : defines how a network element forwards data with its header
    - Needed by Routing
    - Types of networks
        - Virtual Circuit (VC) networks
            - virutal circuits (paths) must be setup before traffic
            - Pkt forwarding depends on header (VC number)
            - intelligence in network (dumb end systems)
            - guaranteed services : debit, latency
            - Ex : Circuit Switching
        - Datagram networks
            - no setup
            - pkt header contains dest
            - every router must knpw how to reach dest !
            - Uses IP
            - intelligence at edge
            - best effort service


    
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
    - Parity bits 
    - Checksum
    - Cyclic redundancy codes (de degré $k$)(CRC-$k$)
        - Maximize detection 
        - Minimize overhead
        - Principe 
            - nombre binaire (ex : 1010) $\Rightarrow$ Polynome binaire ($x^3 + x$)
            - Règles polynomes binaire 
                - \+, \- : XOR
                - \* : AND
                - / : identité (seulement def pour 1)
                - % : reste division entière
            - Posons 
                - $G(x)$ (Generator) un polynome de degré $k$ commun au receiver et au sender
                - $M(x)$ (Message) un polynome de degré $n$ (le msg à envoyer)
                - $R(x) = M(x) \cdot x^{16} \% G(x)$ un polynome
                - $C(x) = M(x) \cdot x^{16} + R(x)$ le polynome envoyé
                    - if $C(x)$ exact divisible par $G(x)$ : pas d'erreur
                    - else : erreur

- Medium Access Control (MAC) Protocols 
    - For sharing the channel
    - 3 types
        - Channel Partitionning (FDM, TDM, ...)
        - Random Access
            - allows collision and recover from them
            - ALOHA
                - works with ACKs, if no ACK (collision) : retransmit
                - Slotted ALOHA
                    - increase success rate with time slots
                    - require clock sync
                - Carrier Sense Multiple Access (CSMA)
                    - listen before transmit
                        - if channel idle : transmit 
                        - if channel busy : wait
                    - cons : time is required to detect business
                    - CSMA/CD (collsion detection)
                        - minimum frame size 
                        - no ACK
        - Taking turns
            - works with tokens

- Ethernet 
    - wired LAN tech
    - star topo > bus topo
    - no handshake
    - unreliable : no ACK
    - works with CSMA/CD

### Physical

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

