<link href="style.css" rel="stylesheet"/>

# Réseaux

## <img src="images/layers.png" alt="desc" height="200"/>

### Application

- Exemples : vie de tous les jours $\Rightarrow$ e-mail, jeux, streaming, etc
- Principes 
    - communique par network
    - Client process : init communication
    - Server process : wait for communication
    - Socket 
        - interface entre OS et network stack (Transport, Network, ...)
        - choix de transport (TCP, UDP)


### Transport

### Network

- Network Edge : Connexion par signaux
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
        - <img src="images/intro/digital_modulations.png" alt="desc" height="100"/>
        - <img src="images/intro/qpsk.png" alt="desc" height="100"/>
        - QAM <img src="images/intro/qam.png" alt="desc" height="100"/>
        - plus efficace (avec noise) : PSK, QPSK

    - Transmission impairments
        - Limited Bandwidth :
            - $f$ max imposée $\Rightarrow$ déformation signal
            - <img src="images/intro/limited_bandwidth.png" alt="desc" height="100"/>
        - Attenuation (with distance)
            - $P_{tx}$ et $P_{rx}$, resp. puissance des signaux transmis et reçus
            - $Attenuation = \frac{P_{tx}}{P_{rx}}$
            - $Attenuation_{dB} = 10 log_{10}(\frac{P_{tx}}{P_{rx}})$
        - Delay distortion 
            - <img src="images/intro/delay_distortion.png" alt="desc" height="100"/>
        - Noise (Bruit)
            - Origines
                - Thermal noise : mvmt électrons
                - Intermodulation : interférence avec modulateurs alentours (gérable easy)
                - Impulse noise
            - Signal-to-Noise Ratio, $SNR = \frac{S}{N}$ où S, N puissance resp. du signal et du noise
            - SNR++ $\Rightarrow$ perf++
            - impact QPSK
                - <img src="images/intro/noise_qpsk.png" alt="desc" height="100"/>

    - Theorical channel capacity
        - Noise free channel 
            - $R = 2B$ (où B est la Bandwidth)
            - $C = 2B log_2(M)$ (général)
        - Noisy channel
            - $C = B log_2(1 + \frac{S}{N})$

- Network Core :
    - Comment transférer data ?
        - Circuit Switching : Réservation d'un circuit pour transférer
            - Resources : link bandwidth + switch capacity (not ability)
            - Ressources dédiée / 1 circuit $\Rightarrow$ perf++
            - Multiplexing
                - Frequency Division Multiplexing (FDM)
                    - <img src="images/intro/fdm.png" alt="desc" height="100"/>
                    - Comment ? Signal porteur $f_c$
                - Time Division Multiplexing (TDM)
                    - <img src="images/intro/tdm.png" alt="desc" height="100"/>
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
                    - $t_{trans} = \frac{L}{R}$ (transmission delay)
                    - $t_{propa} = \frac{d}{s}$ (propagation delay)
                    - $t_{proc}$ (processing delay, often negligible)
                    - $t_{queue}$ (queuing delay)
                    - $t_{nodal} = t_{trans} + t_{propa} + t_{proc} + t_{queue}$
            - queueing
                - M/M/1 queueing model (theorical)
                    - M/M/1 $\Rightarrow$ Markovian x 2, 1 infinite q
                    - $\lambda$ avg pkt **arrival** rate (pkts/s)
                    - $\mu$ avg pkt **departure** rate (pkts/s)
                    - $\rho = \frac{\lambda}{\mu}$ intensity (stable if < 1, 'inf' delay else)
                    - $\overline{N} = \rho (1 - \rho)$ avg nbr of pkts in q
                - delay depends on pkt arrival $\Rightarrow$ exos
                - packet loss (q has finite capacity)
            - debit : $R \leq min(R_i) \Rightarrow$ bottleneck 
            - store and forward $\Rightarrow$ long transmission delay
                - solution : <img src="images/intro/pipelining.png" alt="desc" height="100"/>
                - $d_{trans} = (\sum \frac{L}{R_i}) + (N - 1) \frac{L}{R_{min}}$
            - packet max size (bc pipelining)
                - msg size = $M$ bits
                - header size = $H$ bits
                - max packet size = $K_{max}$ bits
                - Number of bits to transmit = $M + H \lceil \frac{M}{K_{max}} \rceil$
                - Time $T$ 
                    - without pipelining : $T = N (\frac{H + M}{R})$
                    - with pipelining : $T = \frac{1}{R} ((N - 1) K_{max} + M + H \lceil \frac{M}{K_{max}} \rceil)$ où N is nbr of links
                - Average number of packets = $\frac{\overline{M}}{K_{max}} + \frac{1}{2}$
                - $\overline{T} = \frac{1}{R} ((N - 1) K_{max} + \overline{M} + H (\frac{\overline{M}}{K_{max}} + \frac{1}{2}))$
                - $K_{max} \approx \sqrt{\overline{M} \frac{H}{N - 1}}$
    - Network Structure
        - ISP : Internet Service Provider (Fournisseur d'accès internet)
        - Tier 1,2,3 $\Rightarrow$ chaque tier paye au-dessus (tier 3 = Proximus, etc)

### Link

### Physical
