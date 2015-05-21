---
layout: post
title: Sulla mistica dello storage enterprise
excerpt: "Una delle mie mitologie preferite."
tags: [storage]
modified: 2015-05-21
comments: true
image:
  feature: storage.jpg
---


Quello dello storage non-consumer è sicuramente uno dei settori dell'IT più floridi per quanto riguarda la nascita di miti e leggende: cosa favorita, oltre dal marketing dei big players, dall'oggettiva complessità di alcuni concetti in gioco e dalla diffusa ignoranza degli uomini-IT in merito.

Vediamo di semplificare un po', dando delle descrizioni "operative".

## DAS
Né più né meno dello storage interno del vostro server, ma ospitato in una enclosure esterna; da vedere come una "unità di espansione dischi" o "moltiplicatore di porte SAS esterno". Generalmente il collegamento alle macchine avviene tramite cavi SAS (gli stessi che collegano i dischi locali), lo storage viene visto esattamente come uno o più dischi interni alla macchina. 

Come un qualsiasi controller SAS interno, sono muniti di funzionalità RAID e permettono di esporre partizioni, "array" dell'aggregato di dischi fisici, presentati alla macchina come normalissimi dispositivi a blocchi AKA dischi interni.

## SAN
È qui che si concentrano la maggior parte degli equivoci, quindi diciamolo subito: la SAN non è altro che un DAS esposto ai servers tramite un protocollo di rete (iSCSI, FC, FCoE). Fine.
Più precisamente la Storage Area Network è un insieme di dispositivi, composta oltre dalla parte al quale viene generalmente attribuito il nome SAN, anche dagli switch e dagli HBA presenti sui servers; è giustappunto un **network**, non un singolo dispositivo.

È quindi possibile fare switching e routing del flusso di dati riguardanti lo storage, oltre a poter utilizzare il tutto in direct-attach similmente al DAS.
Cosa? Routing dello storage? Pazienza Iago, pazienza.

## NAS
Questo lo conosciamo bene tutti; un NAS è un fileserver che permette la condivisione dello storage a livello del filesystem, ovvero incorpora tutto lo stack necessario a presentare ai servers files e cartelle. Questo è l'unico dispositivo fra quelli presi in esame a permettere l'accesso allo storage a livello di files e non di blocchi; cioè, non è permesso accedere direttamente all'array di dischi fisici per partizionarlo, formattarlo ecc. Inoltre è l'unico, e sottolineo l'unico, ad incorporare la logica necessaria a gestire l'utilizzo in concorrenza di più macchine ad uno stesso pezzo di storage.

## E allora?
Come, la SAN non serviva per "condividere lo storage" fra più macchine? NO.
La SAN nasce principalmente per semplificare e rendere più efficiente la gestione dello spazio su disco di installazioni molto dense con parecchi servers, poiché è parecchio più semplice assegnare una LUN ad un server piuttosto versus spostare fisicamente (con ricostruzione array e quant'altro) dischi fra più macchine. In questo modo, è possibile assegnare dinamicamente "dischi" di dimensione variabile al server, senza interventi fisici e interruzioni dell'operatività: il punto che si possa fare su più macchine contemporaneamente, condividendo i dischi fisici, è la ragion
d'essere della SAN. 

In realtà, anche il DAS può fornire funzionalità di questo tipo se le macchine sono poche e abbastanza vicine allo storage, collegando direttamente i servers allo storage.
Nella caso della SAN, sia essa FC, FCoE o iSCSI, è addirittura possibile collegare lo storage e i servers a degli switch e persino a dei router in configurazioni simili a quelle di una tradizionale LAN, che permettono ad esempio di realizzare configurazioni che hanno particolari caratteristiche di resistenza ai guasti (utilizzando più SAN in replica, ad esempio).

Andiamo oltre adesso; collegate due macchine fisiche alla stessa LUN di una SAN (con dei dati all'interno, magari): vi ritroverete con un filesystem irrimediabilmente corrotto dopo qualche secondo, e la totale distruzione dei dati sarebbe cosa inevitabile dopo pochi tentativi di scrittura.
Perché? Perché collegare una SAN a due macchine, è come collegare un ideale hard disk fisico con due cavi SAS a diversi computers: alla prima operazione contemporanea, vi ritrovate con un ammasso di bit insignificanti.
C'è un solo modo per gestire un hard disk condiviso fra due macchine: un filesystem distribuito, detto anche *clusterizzato*.

## SAN != File Sharing
L'errata considerazione della SAN come "storage di files condiviso" a mio parere nasce dalla grande facilità d'uso di VMFS di VMware, che permette l'uso "trasparente" della SAN, presentando gli array agli utenti proprio come farebbe un NAS, cioè come spazio formattato nel quale è possibile effettuare operazioni in concorrenza sui files. Ma attenzione: come nel caso del NAS, è il filesystem che si occupa di gestire la concorrenza, questo nulla ha a che vedere con le caratteristiche della SAN. Provate a configurare un filesystem clustered come GPFS o GFS, se volete capire profondamente come funzioni la cosa; è sempre bene sporcarsi un po' le mani per arrivare ad una comprensione piena di certi meccanismi.

## Performance
La tipologia di storage più performante in assoluto continua ad essere quello interno ai servers, ancor di più con il recente affermarsi di ssd PCI-E e memory-channel. Lo storage di rete, sia
esso SAN o NAS (e in una certa misura anche il DAS) è comunque limitato dall'overhead del protocollo di rete e dalla sua larghezza di banda.
L'ultima versione del FC quota 16Gbit/s, appena uscito e dai prezzi folli… Il PCI-E 3.0 16x fa circa 126Gbit/s, oggi, per slot, e sono già uscite le specifiche del 4.0 che avrà esattamente il doppio di banda; cambia poco negli ordini di grandezza se consideriamo iSCSI.
Il DAS ha generalmente poco overhead, scala in termini di multipli interi di porte SAS a 12Gbit/s.

# Keep it simple, stupid!
Tutto questo può essere letto come un invito a semplificare il più possibile la topologia dello storage: si guadagna in prestazioni, si risparmia,  si ottiene un'affidabilità maggiore; la crescita di complessità può essere giustificata solamente quando non è assolutamente possibile soddisfare i requisiti progettuali senza aggiungere componenti o layer di astrazione.

In generale, lo storage locale è sempre da preferire a tutte le altre alternative; la "catena cinematica" è la più corta, ciò porta a meno connettori/interfacce che possono guastarsi (o fare contatto intermittente, che è una delle cause più comuni del fail nei raid con parità), minore latenza e
prestazioni massime, oltre che semplicità di gestione.
La crescita di dimensioni dello storage può urtare i limiti fisici del singolo server, ed ecco che abbiamo bisogno di un DAS per la scalabilità.
Gli ambiti dove è opportuno utilizzare NAS e SAN enterprise dopo l'avvento dei meccanismi di replica software block-level e file-level, sono sempre più ristretti; in particolare, nel caso della SAN è necessario avere del personale iniziato ai misteri della condivisione block-level ed una
infrastruttura abbastanza ridondante, cosa che spesso si traduce in esborsi non indifferenti e difficilmente giustificabili nel mondo delle PMI.
