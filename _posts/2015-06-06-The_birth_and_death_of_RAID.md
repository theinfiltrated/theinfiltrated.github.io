---
layout: post
title: The birth and death of RAID
excerpt: "Quando c'era il controller, ci ricostruivano in orario!"
tags: [storage]
comments: true
image:
  feature: storage2.jpg
---

Le prime notizie relative al RAID risalgono alla fine degli anni '70, quando le macchine UNIX midrange iniziarono a dotarsi di quello che oggi chiameremmo "RAID Software"; prima c'erano solo sistemi per la correzione di errori in memoria, ma di tipo offline.

## Pubblicità progresso
Fa sempre bene ricordare come il RAID non sia una soluzione che riguarda la **sicurezza** dei dati: il RAID è una soluzione che riguarda la *continuità del servizio*. Ovvero, un array RAID permette di *continuare a lavorare* durante un guasto, ma è implicito che ci si debba immediatamente prodigare per sostituire il componente guasto e non si possa fare a meno di un sistema di backup.

Il RAID non sostituisce il backup, **mai**: sono due sistemi concettualmente ed operativamente diversi; se il RAID è online (cioè utilizzato per l'accesso ai dati di lavoro), e garantisce la continuità del servizio, il backup è invece offline, off-site e fornisce archiviazione, conservazione e sicurezza dei dati.
Il backup in genere è una soluzione di *disaster recovery*, il RAID è *risk mitigation*.

## SCSI Golden Age
L'utente medio ricorderà probabilmente i controller entry level Adaptec o LSI dei primi anni '90, nati principalmente per supplire allo scadente RAID software integrato in Windows… ancora oggi parecchio deficitario rispetto all'esemplare implementazione di **md** su Linux.
Le tipiche configurazioni prevedevano RAID 5, tre-cinque dischi (rigorosamente *dispari*, per agevolare il calcolo della parità)… fino all'inizio degli anni 2000, i dischi costavano parecchio e non c'erano alternative in quella fascia di prezzo.

Inoltre, le CPU x86 erano ancora pressapoco dei giocattoli, che mal sopportavano l'overhead dei livelli RAID con parità. I SO avevano ancora una gestione delle code di I/O abbastanza primitiva; l'offloading del calcolo all'ASIC del controller era una manna dal cielo. Da metà degli anni '90 il RAID hardware in qualsiasi macchina server viene considerato praticamente obbligatorio, e in una decade diventa *commodity*.

Ogni array che si rispetti ha i suoi dischi SCSI da 73Gb a 10kRPM, il RAID 5 *regge* perché il fallimento dovuto ad [URE](http://en.wikipedia.org/wiki/RAID#URE) è abbastanza improbabile date le dimensioni degli HDD.
Da segnalare anche la diffusione nelle motherboard consumer dei cosidetti "*FakeRAID*", implementazioni puramente software e supportate quasi unicamente da Windows. Sono generalmente non troppo prestazionali o affidabili, e poco impiegate in ambito enterprise.

## Per dipingere una parete grande, ci vuole un pennello grande
Saltiamo avanti di due decenni, siamo negli anni 2010 (chissà che convenzione tireranno fuori riguardo queste decadi, al momento lo scrivo per esteso): i dischi raggiungono capacità di svariati Tb, il costo dello storage è in caduta libera, il fallimento per URE mette fuori dai giochi i livelli RAID con singola parità.

Il RAID 10 viene universalmente adottato in ambito enterprise, affiancato dal RAID 6 o livelli proprietari (aventi comunque almeno doppia parità) per l'archiviazione massiva.
Assistiamo inoltre ad una ulteriore evoluzione del RAID software: le prestazioni delle CPU rendono **md** più che adeguato per la maggior parte degli array-use_case; inoltre, con la comparsa di *ZFS* l'utente può usufruire di un filesystem che fa anche RAID (con tripla parità!) e *volume management* avanzato… permette anche di impostare dispositivi per il caching!
Ciononostante, il RAID hardware viene ancora preferito in sistemi di produzione per alcune caratteristiche avanzate:

* possibilità di fare caching con analisi *euristica* dei blocchi più utilizzati su cache molto grandi (anche decine di Gb);
* blind swap, ovvero sostituzione dei dischi a caldo e ricostruzione dell'array senza nessun altro intervento da parte dell'operatore (spostando la gestione dell'array dal system admin all'operatore hardware);
* batterie interne, che anche in caso di totale failure dell'alimentazione permettono al controller di terminare le scritture o almeno memorizzarle su dispositivi a stato solido fino al successivo recovery;
* replica remota su altri controller con latenze molto basse;
* OS-agnosticità, tanto da poter essere montati su dispositivi come SAN, NAS, DAS ecc.

Il RAID è una pratica ormai assolutamente standardizzata e **implicita** in ogni installazione di un certo livello.

## …and thanks for all the bits

E allora, perché nel titolo parlo di *morte* del RAID? 
La domanda da porsi è se il concetto di unità di storage in RAID (per come lo intendiamo oggi, assistito da un controller hardware in particolare) abbia ancora senso con l'avvento degli SSD di nuova generazione, per i seguenti motivi:

#### Prestazioni del bus
Sfruttando le connessioni PCI-E (o, ultimamente, gli slot della [RAM](http://www-03.ibm.com/systems/x/options/storage/solidstate/exflashdimm/index.html)), gli SSD permettono prestazioni teoricamente equivalenti alle massime ottenibili da un controller RAID, poiché sfruttano lo stesso bus di sistema per comunicare con la cpu.

#### Parallelismo/Prestazioni
Le maggiori prestazioni ottenuti dai controllers compiendo I/O in parallelo su dischi rotazionali, oggi sono replicabili leggendo o scrivendo in contemporanea (striping) da un numero *praticamente arbitrario* di NAND, solamente "limitato dal controller"; questa volta, quello dell'SSD. 

Da qui la crescita esplosiva nelle prestazioni degli ultimi SSD messi in commercio, che arrivano a picchi di quasi 4Gb/s (GigaBYTE, non Bit) R/W, con latenze bassissime.
E se ci sono ancora obiezioni sulla capacità, basta guardare agli [ultimi ritrovati](http://www.fusionio.com/data-sheets/iomemory-sx300-atomic-series/).

#### Parallelismo/Affidabilità
La maggior affidabilità del RAID di dischi rotazionali (RAID 10, ovviamente) è data dal poter eseguire letture e scritture in parallelo (stavolta, cloning) su "pezzi di hardware" distinti, e grazie ad uno strato software di recuperare eventuali fallimenti… esattamente ciò che viene fatto *NAND per NAND* dagli attuali SSD enterprise, che spesso espongono una capacità effettiva minore (anche del 30%) per potere, grazie alla logica del controller, tenere alcuni settori *spare* che generalmente vengono utilizzati come **settori di parità** a la RAID con parità (multipla!).

Questa architettura sembra funzionare molto bene per via dell'enorme numero di *nodi* per stripes e il grande "dominio di parità"; il controller riesce a tenere conto con discreta precisione dello stato di salute delle singole componenti, e generalmente rimpiazza le NAND che si avvicinano a fine vita con quelle precedentemente tenute *a riposo* prima che ci possa essere un sostanziale degradamento di prestazioni o affidabilità.
Un esempio di queste funzionalità viene molto chiaramente descritto [qui](http://www.fusionio.com/blog/adaptive-flashback).

# So what?
A mio parere, ci avviciniamo ad un cambio di paradigma nel quale il supporto di memorizzazione a stato solido sarà affidabile e prestazionale quanto un controller RAID, rendendone di fatto superfluo l'utilizzo, limitatamente almeno allo storage locale.
La battaglia si sta già svolgendo, specialmente fra gli algoritmi per il wear-leveling e livelli di RAID-ridondanza "interni"; ne parlano molto gli ultimi papers di IBM sui loro nuovi storage all-flash e altri di Intel e FusionIO.

Costano troppo? Space-wise, ancora per qualche anno. IOPS-wise, c'è già stato il sorpasso. Per fare gli IOPS di un SSD enterprise ci vogliono una batteria di dischi SAS più un controller RAID con una cache mica da ridere… e se non li puoi hostare dentro la macchina singola, ti serve anche una SAN, che presa da sola peggiora l'affidabilità del sistema. Altrimenti ne servono due, con due switches FC, HBA ridondanti, ecc; ed ecco che la soluzione con storage locale torna ad essere concorrenziale sia sul prezzo che nella semplicità di gestione, magari assistita da una replica block-level.

Naturalmente, lo use-case che immagino è quello di workload mission-critical; non certo di quello di consolidamento dello storage nel quale latenza, prestazioni pure e semplicità dell'infrastruttura passano in secondo piano.
