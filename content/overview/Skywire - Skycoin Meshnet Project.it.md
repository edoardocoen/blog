+++
title = "Skywire - Progetto Skycoin Meshnet"
tags = [
    "Skywire",
    "Meshnet",
]
bounty = 15
date = "2017-08-29"
categories = [
    "Skywire",
    "Overview",
]
+++

![Skywire: The New Internet](https://i.imgur.com/9Jk0gLe.jpg)

<!-- MarkdownTOC autolink="true" bracket="round" -->

- [Introduction](#introduction)
- [Routing: Overview](#routing-overview)
- [Incentives: Payment Protocol](#incentives-payment-protocol)
- [Source Routing: Link Layer Encryption](#source-routing-link-layer-encryption)
    - [Example Protocol: Nodes `A` and `B`](#example-protocol-nodes-a-and-b)
    - [Possible Improvements:](#possible-improvements)
- [IPv4 Gateway: Bypassing Existing ISPs](#ipv4-gateway-bypassing-existing-isps)
    - [Example One](#example-one)
    - [Example Two](#example-two)
- [Skywire Daemon Services Architecture](#skywire-daemon-services-architecture)
    - [Example Service: Blockchain Sync](#example-service-blockchain-sync)
        - [Finding Peers](#finding-peers)
        - [Sending and Receiving Messages](#sending-and-receiving-messages)
- [Multi-Home Routing and Link Aggregation](#multi-home-routing-and-link-aggregation)
- [Meshnet Routing: Store and Forward](#meshnet-routing-store-and-forward)
- [Store and Forward: Capacity Utilization](#store-and-forward-capacity-utilization)
- [Store and Forward: Examples](#store-and-forward-examples)
    - [Example of normal operation](#example-of-normal-operation)
    - [Example with congestion](#example-with-congestion)
    - [Example with packet loss](#example-with-packet-loss)
- [Store and Forward: Bandwidth Latency Product](#store-and-forward-bandwidth-latency-product)
- [Store and Forward: Capacity Utilization, Quality and Service](#store-and-forward-capacity-utilization-quality-and-service)
- [Source Routing: Multi-Route Mobile Connectivity](#source-routing-multi-route-mobile-connectivity)
- [Source Routing: Guard Nodes](#source-routing-guard-nodes)
- [Source Routing: Limitations of BGP](#source-routing-limitations-of-bgp)
- [Virtual Routes: Skywire Network Topology at Scale](#virtual-routes-skywire-network-topology-at-scale)
- [Source Routing: Virtual Routes, SONET Topology](#source-routing-virtual-routes-sonet-topology)
- [Source Routing: Asymmetric Connectivity](#source-routing-asymmetric-connectivity)
- [Source Routing: Route Discovery](#source-routing-route-discovery)

<!-- /MarkdownTOC -->

## Introduzione

Obbiettivi di Skywire:

* Aumentare la concorrenza a banda larga. Fornire un'alternativa agli ISP esistenti. Estendersi almeno un miglio.
* Consentire alle community di creare ISP in esecuzione su un'infrastruttura gestita dall'utente.

Skywire è un nuovo protocollo darknet.

* Bassa latenza (veloce come TCP / IP e teoricamente più veloce sulla rete nativa)
* Prestazioni elevate (progettate per video, condivisione di file e applicazioni ad alto utilizzo di banda)
* Conservazione della privacy
* Supporta l'operazione Over Wifi (Meshnet)
* Supporta l'operazione di Clearnet (Darknet / Overlay)

Skywire risolve il problema degli incentivi e di leeching per l'implementazione della rete.

* Gli utenti ricevono Skycoin per fornire risorse di rete
* Gli utenti consumano monete per consumare risorse di rete

Skywire è ad accesso libero.

* Chiunque abbia un client è in grado di connettersi a qualsiasi nodo Skywire
* L'obiettivo è creare una meshnet aperta ad accesso globale

Skywire è "tutela della privacy".

* Il traffico che passa attraverso il tuo nodo non può essere ricondotto al tuo indirizzo IP
* I nodi che inoltrano il traffico possono vedere solo l'hop precedente e quello successivo
* Le terze parti che osservano passivamente non possono collegare singoli pacchetti a un flusso o utente
* I terzi e i nodi di inoltro non possono leggere il contenuto del traffico

## Routing: Panoramica

Lo Skywire meshnet utilizza un protocollo store-and-forward source-routed.

Il nucleo della rete di sovrapposizione è una serie di nodi.

* Ogni nodo è identificato da un hash della chiave pubblica
* Ogni nodo riceve messaggi e inoltra messaggi
* I nodi ricevono monete per l'inoltro del traffico

Per comunicare tra i nodi `A` e` C`, tramite il nodo `B`:

* Il nodo `A` si collega al nodo` B` e stabilisce un percorso
* Il nodo `A` estende la rotta a` C`
* Il traffico inviato sul percorso su "A" arriverà a "C"

In una rotta `A -> B -> C -> D`:

* I nodi conoscono solo il passaggio precedente e successivo.
* `C` saprà che il messaggio è passato attraverso` B` ed è indirizzato a `D`. `C` tuttavia non conoscerà l'identità di` A`.
* `B` non può dedurre che` A` sia l'origine del percorso.
* `C` non può dedurre che` D` è la destinazione finale del messaggio
* `B` e` C` non possono leggere il contenuto del messaggio (crittografia end-to-end)
* Un osservatore passivo che non partecipa all'instradamento di un particolare messaggio non può ottenere alcuna informazione sul contenuto del messaggio (crittografia a livello di collegamento).
* È possibile raggruppare più messaggi da più percorsi verso la stessa destinazione, riducendo la capacità dell'osservatore passivo di eseguire analisi del traffico.

Nella più semplice implementazione una rotta è un prefisso a 128 bit. 
Ogni nodo legge il prefisso e fa una ricerca in una tabella, per determinare 
il nodo successivo in cui inoltrare il pacchetto.

La fonte ha il controllo completo sul routing.

* Ogni nodo può aggiornare il proprio protocollo di routing in modo indipendente per soddisfare le proprie esigenze
* La sorgente può ottimizzare i percorsi di rete per percorsi di rete a bassa latenza per VOIP o giochi
* La sorgente può ottimizzare i percorsi di rete per il throughput per le applicazioni di condivisione di video e file
* La sorgente può raggruppare più percorsi verso la destinazione per ridondanza, latenza ridotta e velocità effettiva

Alcune applicazioni raggrupperanno più percorsi paralleli a livello di applicazione per:

* privacy (nodi gatekeeper, tipo tor gateway / servizi anonimi)
* throughput
* latenza ridotta
* ridondandza

Questo è il cuore della rete di overlay Skycoin. È molto semplice, ma estremamente potente.
I dettagli tecnici e di implementazione saranno discussi in seguito.

Skywire ha semplicemente prefisso un pacchetto con un ID percorso.

* Il routing è una ricerca di tabelle molto semplice
* L'overhead per pacchetto è costante e non aumenta per i percorsi lunghi

Note:

* La destinazione non conosce l'identità dell'origine. L'identità non è più a livello di routing, ma a livello di applicazione. L'identità deve essere confermata attraverso l'infrastruttura a chiave pubblica.
* Gli attacchi Man-in-middle non sono possibili. Una fonte può verificare la destinazione tramite la propria chiave pubblica.
* La privacy è significativamente migliorata da IPv4, dove chiunque gestisce un pacchetto può vedere la destinazione, l'origine e il contenuto del pacchetto.
* Le prestazioni sono superiori a IPv4 / BGP perché gli ISP utilizzano il routing hot potato.
* La crittografia end-to-end elimina gli attacchi di iniezione di pacchetti e lo spoofing. Il traffico di spoofing richiede le chiavi private per ciascuna estremità del tunnel di connessione.
* La crittografia è veloce. L'obiettivo è un throughput di 10 Gb / s su hardware FPGA, 200 Mb / s su ARM.

## Incentivi: protocollo di pagamento

![Skywire miner](https://i.imgur.com/2zj4CUV.jpg)

*[Skywire "miner"](/statement/skywire-miner-hardware-for-the-next-internet/)*

I nodi vogliono inoltrare il traffico e ricevere monete. Questo è l'equivalente di "mining" in Skycoin e quanti utenti otterranno le loro prime monete.

I pagamenti per il transito non devono rivelare l'identità del nodo di origine. Skycoin utilizzerà un intermediario cieco per i pagamenti attraverso terzi fino a quando non verrà sviluppato un protocollo migliore.

Ogni nodo nella rotta registra il traffico e il nodo di origine registra il traffico. Periodicamente stabiliscono i pagamenti della larghezza di banda.

L'origine tiene le monete in deposito con terzi. Un account pseudonimo viene creato con la terza parte. Ogni nodo può verificare la reputazione dell'origine e l'abilità di pagamento attraverso terzi, senza conoscere l'identità della parte. Per le terze parti, ciascuna origine apparirà come più account pseudonimi non collegati. Ogni nodo di transito apparirà come più account di pseudonimi non collegati.

Piccole transazioni saranno regolate internamente, in transazioni fuori blocco. Le transazioni fuori blockchain possono essere ritirate in un indirizzo appena generato, mai usato prima che il saldo superi una soglia (attualmente 1 Skycoin). Questo riduce la blockchain e incoraggia le microtransazioni ad essere eseguite fuori blocco.

## Source Routing: Link Layer Encryption

Esiste una link layer encryption predefinita tra gli hop e la crittografia end-to-end predefinita. Un'applicazione tipica utilizzerà la crittografia a livello di collegamento, la crittografia end-to-end e quindi la crittografia a livello di applicazione appropriata.

La crittografia tra i nodi dovrebbe essere veloce. Le implementazioni FPGA devono supportare il funzionamento a 10 Gb / s alla velocità di linea. I processori ARM devono essere in grado di supportare 250 Mb / s.

L'attuale miglior candidato è ChaCha20 con scambio di chiavi effimere ECC secp256k1.

ChaCha20 only uses simple arithmetic operations, is faster than AES for embedded devices and is more resistant to timing channel attacks than AES.

ChaCha20 utilizza solo operazioni aritmetiche semplici, è più veloce di AES per i dispositivi embedded ed è più resistente agli attacchi dei canali temporali rispetto ad AES.

La chiave della sessione precedente deve essere accumulata nel segreto ricevuto tramite ECDH.

La chiave di sessione, stabilita tramite crittografia a chiave pubblica (ECC), viene utilizzata per crittografare le comunicazioni utilizzando un algoritmo di crittografia asimmetrico più veloce (AES, ChaCha20). Questa è la crittografia di base del livello di collegamento tra i nodi.

### Protocollo di esempio: Nodi `A` e` B`

- Il nodo `A` vuole generare una chiave di sessione per l'invio di dati crittografati al nodo` B`
- Il nodo `B` ha la chiave pubblica` P`, con chiave privata `p`. `P` è un punto sulla curva EC25 sep256k1. `p` è un numero intero a 256 bit. `P` è il punto base b, elevato alla potenza di p con l'operazione di aggiunta della curva.
- Il nodo `A` genera una chiave pubblica effimera` Q`, con chiave privata `q`. (Il nodo `A` genera a caso un intero di 20 byte.Questa è la chiave privata` q`. Il nodo `A` solleva il punto base alla potenza di` q`, per generare la chiave privata `Q`, che è un punto sulla curva).
- Il nodo `A` manda,` P` * `q` (il punto sulla curva` P`, che è la chiave pubblica `B`, alzata al potere di` q`)
- Il nodo `A` invia` P` al nodo `B`
- Il nodo `B` riceve` P` e calcola `P * q`, il nodo` A` può calcolare `p * Q`, che sono uguali. Questo è il segreto condiviso, che viene sottoposto a hash per generare la chiave di sessione.
- `P = b * q`, quindi` P * q` è uguale a `(b * p) * q`. `P * q = (b * p) * q = (b * q) * p = Q * p`, poiché` Q = b * q`. `A` sa` q, Q` e `P`, e` B` conosce `p, P` e` Q`. Quindi `A` e` B` possono entrambi calcolare `P * q` e` Q * p` e usare questo come loro segreto. Tuttavia, una terza parte non conosce le chiavi private `q` per` A` né conosce la chiave privata `p` per` B`, quindi una terza parte non può calcolare questo "segreto" e quindi non può leggere nulla crittografato sotto il segreto .
- Il nodo `B` conferma la ricezione dell'aggiornamento della chiave di sessione. Il nodo `A` inizia a trasmettere sotto la nuova chiave di sessione non appena la conferma viene ricevuta da` B`.
- Il nodo `A`, invia messaggi al nodo` B` crittografandoli usando ChaCha20 usando la chiave di sessione, come chiave di crittografia asimmetrica.

### Possiblili miglioramenti:

- Aggiornamenti frequenti delle chiavi di sessione. Scambio tasti ECDH ogni pochi secondi o minuti.
- Inserisci la vecchia chiave di sessione con il nuovo segreto ECDH per generare una nuova chiave di sessione.
- Aggiungi nonces a pacchetti e hash secret in nonce per generare la chiave per ogni messaggio. La stessa chiave non viene mai riutilizzata. Riduce l'impatto dell'attacco con testo in chiaro noto.
- Elimina il testo normale noto nei messaggi.
- Pad messaggi a multipli di 16 o 32 byte.

## Gateway IPv4: esclusione degli ISP esistenti

Molte persone hanno solo una scelta per gli ISP. Questo descrive brevemente in che modo Skywire può aumentare la concorrenza.

Alcune applicazioni possono essere eseguite in modo nativo nello spazio degli indirizzi Skywire. Alcune applicazioni come Bittorrent, la sincronizzazione dei file e le applicazioni di comunicazione beneficiano fortemente dell'infrastruttura Skywire e saranno modificate per essere eseguite in modo nativo su di essa.

Le applicazioni legacy, come Netflix, Facebook, Twittter richiedono un gateway di rete che interfaccia la rete di sovrapposizione Skywire con reti IPv4 e IPv6.

Un utente sceglie un gateway Skywire in esecuzione su un server in un centro di colocation locale. Il traffico IPv4 degli utenti verrà tunnelato attraverso il gateway (simile a una VPN). L'IP dell'utente apparirà come IP del server gateway. Il server ha una connessione gigabit a internet backbones a multiple velocità, su provider che non valutano il limite Netflix. L'utente ha più opzioni di provider per i gateway IPv4 di Skywire. Il fornitore gateway sarà pagato in Skycoin su base misurata.

Il nodo Skywire nella home dell'utente si connette al gateway su tutti i percorsi possibili. Il nodo Skywire esegue il tunneling del traffico IPv4 da un router al gateway nel centro di colocation. L'indirizzo IP del nodo gateway è l'indirizzo IP che appare all'utente.

### Esempio uno

Un utente ha un modem via cavo a 10 Mb / s. Installano un router Skywire. Il router è collegato al computer, a un nodo Skywire Wifi e al modello di cavo. Il router è configurato con Skywire come tunnel IPv4. Collegano il loro computer al router.

Il nodo Wi-Fi Skywire si collega al nodo Skywire dei vicini tramite Wi-Fi, che è collegato a un modem via cavo a 10 Mb / s. Il vicino ha anche un wifi a 5 GHz a 200 Mb / s con un'antenna direzionale point-to-point collegata a un'azienda che gestisce un nodo Wi-Fi Skywire in fondo alla strada.

Il router Skywire degli utenti effettua una ricerca ricorsiva prima dei nodi con connettività clearnet e stabilisce percorsi

* Il suo modem via cavo
* Wifi -> il suo modem via cavo vicino
* Wifi -> 5 GHz punto a punto -> 100/30 Mb / s calo di business / fibra

L'utente sarà in grado di accedere e aggregare la larghezza di banda su tutti i percorsi, collegandosi al tunnel IPv4. Nelle comunità in cui la larghezza di banda e l'affidabilità aggregate hanno raggiunto un determinato livello, l'utente non ha più bisogno del modem via cavo per la connettività.

### Example Two

The business down the street has a 100/30 Mb/s fiber drop with an SLA. The business pays a fixed rate for internet. Any bandwidth they do not use is lost. The business hooks up a Skywire router. The router has 3 ports. The first port is their WAN connection, the second port is their internal network, the third port goes to their Skywire wifi nodes on roof. The router buffers and prioritizes traffic on the internal network and allocates unused capacity to the Skywire traffic. The operator receives Skycoins for transit which, subsidizes the cost of the fiber drop.

## Skywire Daemon Services Architecture

* Each Skycoin node has a Secpk256k1 pubkey.
* Each Skycoin node has a Skycoin address identifying it. The address is a hash of the node’s public key. This public key hash is the equivalent of an IP address on the network.
* Each Skycoin node has a connection pool of peers it is connected to. These can be peers over TCP, UDP clearnet connections, physical connections through direct Ethernet and Wifi peer (meshnet operation). A connection can also be a “virtual connection” which is tunneled through a physical or clearnet connection and will be described later.
* Each connection instance with a peer has “channels”. A channel is a 16 bit integer which is similar to a “port” in TCP.
* All messages sent and received have a 32 bit length prefixed and 16 bit channel.
* Channel 0 is reserved for communication between Skywire Daemons,  exposing meta-information about the services running on the Daemon and other data required for network operation.
* A Skywire daemon may expose “services” on a channel. A service is a process that handles data messages received on a channel and originates data messages addressed to remote peers and services.

### Example Service: Blockchain Sync

*This example is refers to a Golang implementation, but the daemon architecture is language agnostic.*

You want to sync two different personal block chains of people with public keys A and B. You initiate two “blockchain sync service” instances, configure them with the respective public keys and associate them with the Skycoin Daemon. These services run on your local daemon, each on a particular channel.

#### Finding Peers

The blockchain sync daemons the hash the public key and do a DHT (Distributed Hash Table) lookup to find other peers syncing the blockchain. Once peers are found, the peers can be introduce each other to additional peers through PEX (Peer Exchange).

#### Sending and Receiving Messages

Services on creation register a list of messages they respond to and can send. Messages are Golang structs. The message struct data is filled out and then sent. The data arrives and the .Handle() method is called on the corresponding message struct.

## Multi-Home Routing and Link Aggregation

If you have a 2 Mb/s cable modem and your neighbor has a 2 Mb/s cable modem and each of you are running Skywire nodes, then your Skywire node can connect to his node and aggregate the bandwidth across both connections. Packets may now take routes through your cable modem and routes through his cable modem. The cable modems are the choke point. To get a 4 Mb/s connection, traffic has to pass in parallel paths through both modems.

Applications like Bittorrent will be able to aggregate bandwidth across all available connections, because they natively open up a large number of connections, which will take district routes.

## Meshnet Routing: Store and Forward

For nodes at the edge of the network communicating over mesh, there are a few issues.

If you have an eight hop network over Wifi and 50% of packets drop on each hop, then only 1 in 256 packets will make it through. Packet drop is normal on Wifi, but traditional TCP treats packet drop as congestion and throttles the connection speed back.

At the network edge, Skywire will use store and forward along routes. This imposes a memory requirement on Skywire nodes, but significantly improves network performance.

For the route `A->B->C`

* Each route has a buffer.
* Each node keeps sending the messages until they are received and acknowledged.
* If the Buffer from `B->C` is full for the route, then `A` will know this and will stop transmitting data until the sent data has been acknowledged there is room in the buffer.

Therefore there are two acknowledgements between nodes at the link layer. One acknowledgement is an affirmative acknowledgement that transmitted data segments were received. Another is an acknowledgment that data from the buffer has been sent to and acknowledged by the next node in the route.

## Store and Forward: Capacity Utilization

In traditional IP networks, as network links are utilized towards capacity the efficiency drops. A network running at 80% capacity faces the risk that a short term burst of data causes a router to go over capacity and network packets are dropped.

TCP interprets dropped packets for any reason as congestion and throttles back speed. The dropped packets also require retransmission under TCP and introduce latency as the application waits for timeout and retransmission before it can process the rest of the packet stream.

With store and forward operation, the route buffers fill up and then nothing happens. When the buffers fill, the incoming node stop sending data until notified of space in the buffer.

This store and forward operation is especially important for practical Wifi meshnets. There are only three non-overlapping channels in the 2.4 GHz band. Packet loss increases very quickly and very early compared to the bandwidth saturation point for a Wifi networks. Wifi packet loss is inevitable and does not reliably indicate congestion or capacity limits.

Store and forward allows us to run the Wifi nodes at full capacity and saturate all available bandwidth, without triggering TCP congestion controls.

Practical networks will require:

* Software Defined Radio
* MIMO
* Beam Forming
* Directional Antennas
* Cooperative temporal and geophysical coordination of transmission time, broadcast power and frequency usage
* 801.11af whitespace frequencies

## Store and Forward: Examples

Each node, for each route keeps track of:

* Buffer size for receiving node for route
* Predicted buffer size (acked and unacked datasegments transmitted)
* Acked buffer size
* Offset, size and sequence of each transmitted message that has not been acked
* Circular buffer of bytes of outgoing datagrams that have not received an ack

A data segment on the link layer may contain concatenated messages, from multiple routes addressed to the same node. This frustrates traffic analysis and improves performance by allowing larger datagrams on networks which support higher MTUs.

For each transmitted message, there are two acks. The first ack is that the datagram has been received by the next node in route. This is an ack for the datagram, which may contain multiple messages, each corresponding to different routes. After receiving this ack, the node no longer needs to retain the datagram. If the datagram is not acked, then it needs to be resent.

The second ack are updates about remaining free bytes in the incoming buffer for a route. If the free bytes in the buffer for a route is large enough, then additional messages for that route can be transmitted.

Another possible approach, maintains a buffer per sender instead of per route, with the receiver sending block messages to the sender for congested routes. This reduces the number of route hash lookups required by sender and is something that may have to be experimented with.

### Example of normal operation

Route: `A->B->C`

* B has 1024 KB buffer for route
* A sends 512 KB to B
* B Acknowledges the 512 KB to A
* < A receives the Acknowledgement (and clears first 512 KB, no longer needs to be stored) >
* B forwards 512 KB to C
* C acknowledges receipt of the 512 KB
* C acknowledges to A that the 512 KB has been forwarded

### Example with congestion

Route: `A->B->C`

* B has 1024 buffer for route
* A sends 512 KB to B
* A sends 256 KB to B
* A sends 256 KB to B
* < A stops sending because pending, is already enough to fill buffer at B >
* B acknowledges to A the receipt of 512 KB and 512 KB
* B sends 256 KB to C
* C acknowledges receipt of 256 KB to B
* B acknowledges to B, forwarding of 256 KB to A
* < A can now send, up to 256 KB additional KB >


Data is assumed to be received in the order sent for Wifi and direct ethernet connection

### Example with packet loss

Route: `A->B->C`

* B has 1024 buffer for route
* A sends 512 KB to B
* A sends 256 KB to B
* B acks the 256 KB
* A infers that the 512 KB was not received
* A retransmits the 512 KB
* B acks the 512 KB
* < B can now continue sending stream in order to C >

## Store and Forward: Bandwidth Latency Product

In store and forward a storage requirement is imposed on the transmitting node equal to the product of the round trip latency and the transmission rate times the round trip latency. 1 GB of ram is enough for 8000 ms round trip latency at 1 Gb/s transmission rate.

Store and forward should be default, but optional.

## Store and Forward: Capacity Utilization, Quality and Service

Video, Audio and file downloads are buffered. Absolute averaged throughput over a time window of seconds matters, while latency is irrelevant. Other traffic such as website requests, video game and voip is real time and should be delivered as quickly as possible.

With two quality of service levels, “Real Time” and “Bulk” we can transmit VOIP, website and video game traffic first, reducing latency for this traffic. Latency insensitive traffic such as video, music and file sharing would only flow over link after real time traffic buffer is empty.

We are able to utilize links at near 100% of capacity while, lowering latency for real time traffic. Therefore we propose supporting two quality of service levels for routes.

## Source Routing: Multi-Route Mobile Connectivity

If connections between nodes are stable, low latency and have high bandwidth then a single route is sufficient for most applications. Some applications like Bitorrent, open a large number of connections and natively can use bandwidth across all available routes.

If the links between nodes are slow, unreliable or connectivity is changing, reliability and performance requires traffic to be multiplexed over multiple redundant routes.

If a Skywire node running on a cell phone is in a car driving down the street, the networks that are accessible will change. Network nodes will come into range and other network nodes will leave range. The node should have continuous connectivity at the application layer, even as the physical connections are created and destroyed.

One approach is choosing a set of reliable nodes on the network backbone as termination points for a route and then proxying the traffic through these nodes, over a set of multiple short term routes.

## Source Routing: Multi-Route Reliability

If links are unreliable or have highly variable latency, it is desirable to encode application data over multiple paths, such that the data can be recovered if data from any of the paths is received. Fountain coding and other encoding methods exist which may be applicable here.

## Source Routing: Guard Nodes

For privacy, if a user wants to further weaken linkability between their Skywire node address (public key hash) to their IP address, they can destinate a fixed set of nodes that are advertised as being transit points for traffic destined for their address or act as required nodes on a route from their address.

## Source Routing: Limitations of BGP

Border Gateway Protocol, the current dominant routing protocol, handles the routing problem by not keeping any state for packets. Instead BGP, allows each network to create a series of ad-hoc rules for each of its routers which look at the source and destination of a packet and decide which network interface to forward the packet to. Routers message each other with connectivity information and another routing algorithm is used for routing within a network domain.

BGP is designed to interface a series of independent autonomous networks. BGP has a homogeneity assumption, the routing within an autonomous domain is assumed to be centrally managed and highly reliable with homogenous routing within the domain. Mesh network and community ISPs will be ad-hoc with heterogeneous device connectivity and routing.

Connectivity in mesh networks, ad-hoc configurations and densely interconnected networks with redundant multi-home routing paths completely violate the hierarchical assumptions BGP.

BGP has several issues that a next gen protocol should address:

* BGP is not self configuring. BGP based networks require extensive technical expertise to configure and operate
* BGP systems often require manual configuration to route around damage and are not resilient against bad configurations
* BGP requires manual creation of ad-hoc route filtering rules and increasing complexity for networks with multi-home connectivity
* BGP networks require highly centralized planning
* The NSA has exploited flaws in BGP to route targeted traffic to interception points
* The assumptions of BGP are becoming increasingly strained, especially for ad-hoc, mesh and mobile networks
* The hierarchical, single path assumptions of BGP make implementation of multi-homing and other next-gen networking requirements extremely difficult
* BGP suffers severe issues when network links are unreliable, such as route flapping.
* BGP routing table size grows exponentially as interconnected subnetworks proliferate.
* Multihoming causes a massive explosion in BGP routing table size.
* BGP has difficulty with load balancing and multi-home routing. BGP limits the ability in practical networks to take advantage of parallel connectivity between locations.
* BGP creates an incentive for ISPs to dump network traffic on to other networks as quickly as possible (“Hot Potato Routing”), reducing performance and increasing latency

There is no alternative to BGP. BGP is the best solution within its design constraints.

The successor to BGP must:

* Be non-hierarchical
* Be self-configuring (zero-conf)
* Operate well with dense ad-hoc, redundant interconnection between networks

## Virtual Routes: Skywire Network Topology at Scale

The Skywire routing implementation requires a node to maintain information for each route that passes through it. Individual nodes are unable to handle hundreds of thousands of individual routes and scalability is achieved through another mechanism.

Skywire is experimenting with a non-hierarchical, self organizing routing that natively supports multihoming and non-hierarchical network topographies while scaling efficiently.

Skywire minimizes network diameter as the network scales through the use of virtual routes. Virtual routes allow thousands of connections to be bundled over a high bandwidth backbone connection with the overhead of a single route.

A “virtual route” creates a tunnel over an existing route:

`A -> B -> C -> D`

The virtual route appears as A->D. B and C may be high bandwidth long distance connections. B and C only incur the overhead of a single route, while A and D incur the overhead of maintaining the routes on the A->D tunnel.

The virtual route may contain traffic from hundreds of bundled routes from A to D, while B and C only experience overhead of a single route. The virtual route may further, bundle multiple redundant network paths between the origin and destination for performance, throughput and redundancy.

Virtual routes allow network capacity to be clustered roughly hierarchically with nodes at each layer having a constant fan in and overhead.

Nodes at the network edge feed into aggregation nodes. Edge aggregation nodes are connected to high bandwidth intradomain transit and feed into gateway nodes which interface between networks. Gateway nodes feed into high bandwidth and long-haul transit.

Virtual routes are a representation of existing interdomain routing relationships, which natively support:

* Non-hierarchical routing (data centers)
* Multiple-homing
* Dense network interconnection between domains at different levels of hierarchy
* Multi-path routing within and between network domains

Virtual routes obey a triangle equality. If the cost of a route A->B, is C(A->B) then

`C(A->B->C) >= C(A->B) + C(B->C)`

Origin preference for low latency, low cost, and low hop routes, creates economic incentives to create an efficient network topology. The network is non-hierarchical and self-organizing. The virtual routes that are created are route summarizations that naturally reflect the flow of traffic.

In BGP, networks try to get rid of traffic as quickly as possible (hot potato routing). In Skywire, networks compete to provide transit (to receive coin incentives). Skywire clients will preference, low cost, low hop count and low latency routes. Networks with direct long haul capacity between source and destination have lower latency and lower hop count and therefore receive preference.

For efficiency, the bandwidth capacity and fan-in (number of routes each virtual route is bundling) at each level of the network hierarchy must be constant, in order to achieve a constant network diameter and logarithmic routing table growth in the number of hosts.

## Source Routing: Virtual Routes, SONET Topology

A multiple-input, multiple-output virtual route, may be physically implemented as a SONET ring, with Skywire nodes in each city the SONET topology passes through. The Skywire nodes act as a gateway router between the Skywire network and the SONET topology.

Nodes are able to queue up large datagrams concatenating multiple messages from the same source to the same destination for efficiency.

The message enters the Skywire node of the SONET ring at a colocation center in one city. The message destination or route is read and the message is encoded for transport over the SONET segment. The message arrives at the destination Skywire node on the SONET segment and continues on its path.

A multiple input, multiple output virtual route is therefore a list of Skywire nodes, with a transit cost, describing a SONET ring or fully connected topology, where any node in list has transit to any other node in the list.

## Source Routing: Asymmetric Connectivity

Next gen wifi systems will have 4x4 and 8x8 antennas in a phased array MIMO arrangement. Such systems are able to project highly focused directional beams. These systems significantly increase the power and signal strength at the receiver, but do not symmetrically improve antenna gain for return signals.

Similarly, a high powered, amplified wifi signal through a directional antenna may be received at a site fifteen miles away, but reception of the signal from the site cannot similarly be amplified as easily as power can be boosted in transmission.

We propose, Asymmetric routes, for situations where messages can be received by a node, but where the node cannot directly communicate back. In an asymmetric route, confirmation messages are relayed over the network by a route, enabling full utilization of asymmetric connectivity over one way communication channels.

Situations where this will become increasingly relevant

* Rural SONET arrangements with amplified Wifi over directional antennas
* Urban connectivity between highly directional and non-directional antennas broadcasting at the same power levels
* Concrete penetration in 802.11af systems
* Non-line of sight LiFi propagation can transmit over 200 Mb/s but its highly asymmetrical
* RONJA type Li-Fi systems have theoretical capacity limits over 10 Gb/s line-of-sight and there is cost/setup advantage for asymmetrical connectivity

Utilizing asymmetric connectivity and routes allowing only one way direct data transmission between nodes has several advances, especially for rural developments and reducing cost of integrating high capacity of next-gen technologies.

## Source Routing: Route Discovery

IPv4 gateway and meshnets for community ISPs, only require breadth first search over paths to clearnet connectivity. The best, most reliable, highest throughput routes have very small depth. Therefore we consider routing solved for this case. Will look at general routing later.
