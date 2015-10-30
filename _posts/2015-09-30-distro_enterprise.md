---
layout: post
title: Perché usare distro enterprise?
excerpt: "Quando la babele deve farsi da parte"
tags: [Linux]
comments: true
image:
  feature: distro.jpg
---

Uno dei consigli più gettonati nei vari forum **per principianti** riguardati Linux è: 

> *Non ti funziona $nuova_versione_ambiente_grafico? <BR> Prova $distribuzione_misconosciuta!*

La multipolarità della comunità OSS ha permesso la nascita e l'evoluzione di progetti importanti, e la loro gestione "poco business-oriented" ha consentito che si sviluppassero in direzioni spesso più interessanti e funzionali rispetto ad equivalenti commerciali.
Dando uno sguardo alla situazione attuale dell'ecosistema Linux però, non si può ignorare come sia il kernel, sia l'ambiente GNU, siano essenzialmente mantenuti da una moltitudine di aziende; gli ultimi [rapporti a riguardo](http://www.linuxfoundation.org/news-media/announcements/2015/02/linux-foundation-releases-linux-development-report) parlano di come oltre l'80% degli sviluppatori siano pagati per il loro contributo dalle oltre 1200 aziende attivamente coinvolte: fra i top contributors spiccano Intel, Red Hat, Samsung, IBM, SUSE e Google.

È possibile trovare la concretizzazione di questi contributi professionali e di qualità in una terna (anche se ho delle riserve sul nuovo player) di distribuzioni che chiameremo **distro enterprise**, ovvero distribuzioni di Linux destinate ad ambienti di produzione per le quali viene offerto supporto commerciale e lo sviluppo/QA viene curato internamente; ci stiamo riferendo naturalmente a Red Hat Enterprise Linux (RHEL per gli amici), SUSE Enterprise Linux e Ubuntu Server.

In questo post volevo esplorare alcuni loro punti di forza che secondo me le rendono una scelta irrinunciabile quando si deve *lavorare*, in senso esteso, con il computer. <BR>
Perché le macchine nascono per fare lavoro al posto dell'uomo, right?

## Supporto
La possibilità di poter chiamare qualcuno che risolve qualsiasi problema entro le SLA di contratto e che all'occorrenza rilascia anche patch customizzate è **im-pa-ga-bi-le**. Fesserie bloccanti che a volte tengono fermi progetti per settimane, di solito vengono risolte in poche ore. 

Poiché chi fornisce la distribuzione da assistenza e mantiene/sviluppa *tutti* i pacchetti presenti nella distro, si riceve supporto anche su Apache, mySQL ecc; a differenza del mondo Windows dove viene garantito solamente il sistema operativo, e si gioca molto allo scaricabarile per qualsiasi software di terze parti. In genere, è possibile mettere su un sistema di produzione con discrete garanzie di fattibilità e tempistiche.

Supporto vuol dire anche manutenzione assicurata a lungo termine: la maggior parte delle distribuzioni succitate offre **dieci anni** di supporto che comprende patch di sicurezza (e non), chiamate di assistenza, nuovo software… insomma, una piattaforma stabile dove poter sviluppare il proprio workflow senza la paura che ci cambi sotto i piedi.


## Compatibilità Hardware e Software
Software di modellazione 3D, schede video high-end, praticamente tutto l'hardware enterprise come HBA, controller RAID e interi servers/soluzioni/infrastrutture… devo aggiungere altro? <BR>
Sì: RHEL è una delle poche distro ad implementare correttamente e di default gli attributi estesi per il filesystem (ovvero, le ACL di Windows) e SUSE vanta una compatibilità spettacolare con Active Directory.

## Possibilità certificazioni / Prospettive lavorative
Un conto è "*so installare Linux, beh, sì*", un altro è poter affermare "*sono certificato come system architect per quell'architettura e blablabla*"; può fare la differenza. <BR> È costoso, è una mezza mafia, ma può fare la differenza. <BR>
Sicuramente conoscere una distribuzione affermata nel panorama enterprise da un marcia in più a parità di curriculum: in generale CentOS è praticamente obbligatoria, ultimamente anche una conoscenza Debian-ish non guasta.


## Stabilità
Inutile negarlo, i bugs bastardi esistono anche su Linux, e spesso le soluzioni non sono né triviali né facilmente attuabili *at all*. <BR> Un ciclo di sviluppo più lento e orientato alla produzione, nonché QA sul codice documentato e funzionalità davvero **funzionanti** sono il miglior biglietto da visita.

Un esempio? Provate a configurare SELINUX o i pacchetti per l'HA su una distro *rolling release*. <BR> E poi ditemi. <BR>
Ogni pacchetto è generalmente testato-integrato per funzionare bene con tutti gli altri, la distribuzione è un *unicum* più che essere un'accozzaglia di pacchetti in un calderone, con alcuni presenti solo perché "fa community", "dobbiamo averli tutti". <BR> Di contro, il parco software è in genere limitato rispetto ad altre distro; il prezzo del rigore.

## Interoperabilità fra versione commerciale e non
La user base delle versioni gratuite (e non solo FLOSS!) di Linux è immensamente più grande di quella delle versioni a pagamento/supportate.

Una dei grandi vantaggi di RHEL e Ubuntu però, è la possibilità di passare in modo assolutamente indolore dalla versione *commerciale* e supportata a quella *community*. Ubuntu vende solo il supporto, ergo il problema non esiste. In CentOS/RHEL basta cambiare i repo, le due hanno compatibilità binaria (cambiano loghi e RHN). SUSE/openSUSE pecca un po' a riguardo, non esiste difatti un equivalente 1:1 di SUSE enterprise. 

È possibile scaricare la versione binaria (oltre ai sorgenti, chiaramente) di SUSE e utilizzarla con i corrispondenti repo community, ma la procedura non è esattamente lineare né "caldeggiata"; tutto il contrario di Ubuntu e CentOS, che hanno guadagnato una gran fetta del loro market share proprio grazie all'ecosistema cantinaro e comunitario che si è creato attorno a loro. 
<BR> SUSE, quando ci svegliamo? <BR>
La spinta all'integrazione con prodotti Microsoft ha curiosi riflessi sulle strategie aziendali…

## Documentazione
Questo è probabilmente il punto più importante della rassegna, il maggiore *selling point*; la documentazione di queste distribuzioni, in generale è **OTTIMA**. 
<BR> Non parlo di "link al forum, link al tutorial, come ha fatto l'utente XY, prova Z": **ogni** funzionalità è completamente documentata e la documentazione è fornita insieme alla distro.

Non mi riferisco al sempreverde *man*, ma ad interi portali dedicati solamente a spiegare, fornire esempi, aiutare a costruire configurazioni funzionanti. 
<BR> Aggiornati, corretti, utilizzabili in ogni momento come *reference* quando ci si trova in difficoltà. Devo dire che sono anche utili per scoprire funzionalità e pacchetti dei quali non si sospettava l'esistenza, semplicemente confrontando "come lo fa SUSE" *vs* "come lo fa RHEL".

## Un consiglio?
Per lavorare, fintantoché potete e non sorgono delle difficoltà insormontabili, usate CentOS. La maggior parte delle soluzioni enterprise sono sviluppate sopra di essa, dai firmware delle SAN; è bene farsi le ossa sugli ambienti realmente utilizzati in produzione.
Se si resta sui repo standard, fatta salvo ovviamente una configurazione corretta, è incredibilmente stabile: la uso correntemente sia come hypervisor (KVM) che come host.

OpenSUSE/SUSE ha molto senso come ambiente desktop: ottima con KDE4, grande integrazione di default con eventuali sistemi Microsoft o altre terze parti. Inoltre, ultimamente sta prendendo molto piede come piattaforma di storage grazie al supporto per CEPH. La suite di High Availability e supercalcolo dicono funzioni molto bene, ma generalmente sono campi dove si customizza parecchio.

Ubuntu… sta guadagnando molta popolarità in installazione di OpenStack. Per il resto, la documentazione è poco affidabile/aggiornata/completa rispetto agli altri contenders, e la selezione del software spesso è fatta in modo scellerato, solamente per includere features consumer a scapito della stabilità; come si può includere un kernel **NON** *lts* in una release *lts*?! Di contro, è molto aggiornata e alcuni progetti proposti da Canonical sono interessanti (penso a LXD), ma la consiglierei solamente se è impossibile raggiungere il vostro scopo utilizzando le altre due alternative.
