---
layout: post
title: Sopravvivere al velociraptor che cancella la tua tesi di laurea lanciando banane
excerpt: "Ovvero, come combinare insieme GIT e SSH per rendere (quasi) impossibile perdere un file."
tags: [GIT]
modified: 2015-05-21
comments: true
image:
  feature: velociraptor.png
---

In questi giorni c'è stato tanto rumore mediatico riguardo alla violazione di account iCloud contenenti dati personali, e non sono mancate brecce in altri noti sistemi poco tempo addietro.

I sistemi cloud proposti al mercato consumer per l'archiviazione dei dati da qualche altra parte hanno conosciuto una crescita incredibile negli ultimi anni, grazie e sopratutto al mercato mobile che ha posto l'accento sulla possibilità di guasti/furti del dispositivo: **un cellulare è intrinsecamente più rubabile e danneggiabile di un PC desktop**.

Mi sembra incredibile e sopratutto *inaccettabile* che a 42 (!) anni dalla nascita di **[SCCS](http://en.wikipedia.org/wiki/Source_Code_Control_System)** si possano ancora perdere dei dati per l'ignoranza e/o l'incuria dell'operatore; ho visto parecchi miei conoscenti disperarsi di fronte a files corrotti, modifiche involontarie ma ormai salvate…

Gli attuali sistemi di cloud storage come Dropbox, iCloud, Google drive ed altri (mi riferisco sempre a prodotti consumer) limitano molto la problematica, ma permangono delle criticità.

#### Privacy
Dove sono i miei dati? Chi ha i miei dati? Qualcuno fa analisi statistica/rivende metadati sui miei files? È possibile che io non possa più fisicamente accedere ai miei dati?

#### Limitazioni/Costi
Molti servizi offrono account gratuiti, ma in genere lo spazio disponibile è parecchio limitato… inoltre, vengono imposte delle limitazioni come il numero di repliche, la dimensione massima dei files, la tipologia di client (pensate ad iCloud!), l'accessibilità e la completa gestione gerarchica delle informazioni memorizzate. Esistono servizi enterprise-grade che superano buona parte di queste problematiche, ma i costi sono spesso proibitivi; Amazon S3 in questo è forse il miglior compromesso.

#### Condivisione
La possibilità di far partecipare altri utenti al proprio repository di files è spesso parecchio limitata, a meno di non condividere direttamente l'account (ORRORE!). In generale, questi servizi si prestano poco a lavori in team perché non hanno modo di gestire modifiche concorrenti ad uno stesso file, spesso è necessario fare una copia locale delle informazioni interessate e poi processare nuovamente tutto a cose fatte… parecchio inefficiente! Si può fare di meglio.

## Retrospettiva
La mia risposta a questi requirements, viene dal passato: GIT, SSH, un po' di shell aliasing. Strumenti affidabili, collaudati da generazioni di sviluppatori nel tempo, e sopratutto adatti ad un uso professionale… che oggi possiamo utilizzare anche per esigenze personali, dati i giganteschi passi avanti fatti da elaboratori alla portata di tutti. 

È molto significativo notare come si spaccino per *novità* delle conquiste tecnologiche vecchiotte: il "cloud computing" nasce negli anni '60 con i mainframe IBM (avete presente *tn3270* che accede ad ISPF? No?), i sistemi di controllo versione datano prima del 1975, telnet era disponibile già nel '68. 
Oggi è tutto parecchio più comodo, maturo e sicuro; il problema è semmai che l'allargamento della base di utenza anche ad analfabeti (non solo) informatici ha portato ad un deciso livellamento verso il basso delle interfacce utente, che richiedono sempre meno apprendistato e al contempo nascondono la grandissima parte delle tecnologie sottostanti alle finestre sbrilluccicose.

## Requisiti
Parlando di GIT, molti sviluppatori che mi leggeranno lo assoceranno al servizio GitHub; bene, quel che propongo io e di fare quello che fa GitHub lato server: **diventare** GitHub, diventare il servizio, per avere pieno controllo sul posizionamento e la gestione dei nostri dati.

Ecco la lista della spesa:

* Due o più macchine con sistema POSIX o quasi-POSIX (Linux, UNIX, BSD, Mac OS X vanno benissimo); se siete dei temerari, anche Windows con CGYWIN. Nelle macchine dovranno essere installati ssh server e client, oltre ad ovviamente GIT ed una shell POSIX o similare a vostra scelta (io uso ZSH). 
* Una configurazione del network che vi permetta di raggiungere mutuamente le macchine su una porta a vostra scelta, anche attraverso NAT e reindirizzamenti vari va bene. Anzi, è consigliato: cambiate la porta di default per ssh. Fatelo. Punto. Riconfigurate le regole del firewall di conseguenza. 
* La capacità di non cominciare a scrivermi commenti stile "cosa è Linux?", "non ho mai usato GIT, come si fa un commit? Cosa è un branch?", "Ma io ho android", "eh la peppa, io uso razzomissileOS e funziona tutto più meglio assai di quello che scrivi tu": per informarvi su queste cose ci sono DOC e wiki ufficiali, io sto scrivendo di una metodologia per combinare insieme degli strumenti in un determinato scenario… per creare uno strumento nuovo.

> Che lo spirito del RTFM vi accompagni sempre.

## Uno sguardo d'insieme
Supposto quindi che sappiate usare GIT e siate anche dei sysadmin UNIX/Linux discretamente in allenamento, passiamo alla descrizione di alto livello di ciò che faremo. 
Presa una cartella contenente dei files nella nostra macchina, l'idea è che grazie a GIT ogni cambiamento del quale faremo un "commit" verrà registrato come "istantanea" di tutti i file presenti nella cartella (sto semplificando, possiamo aggiungere file che si trovano ovunque ovviamente): questa istantanea sarà anche corredata di un commento esplicativo dei cambiamenti effettuati, così da avere una storyline di tutto il nostro progetto.

Inoltre, ed è questo il pezzo interessante, tutto questo repository contenente sia i file all'ultima versione sia tutte le versioni degli snapshots precedenti all'attuale sia tutti i commenti esplicativi alle varie revisioni, verrano replicati su *N* macchine remote. Naturalmente, sarà anche possibile clonare tutto il repository da queste macchine per generare sia delle nuove cartelle di lavoro, sia dei nuovi repository master dai quali poter clonare tutto, ecc ecc. E questo, su qualsiasi dispositivo che abbia GIT ed accesso ad uno qualsiasi fra i servers. 

La connessione fra server e client, sulla quale opererà GIT, verrà incapsulata dentro il protocollo SSH utilizzato tramite scambio di chiavi asimmetrico di chiavi RSA (niente ID-password, grazie!). E tutta questa procedura verrà automatizzata, in modo da ridursi a un semplice comando, quantomeno per l'upload dei cambiamenti; leggi: "per pararsi il culo da perdite di dati". 
Essendo il flusso di dati criptato tramite RSA, a meno di non fare delle fesserie nella config e di cambiare periodicamente le chiavi RSA (la crittografia asimmetrica è soggetta ad attacchi di tipo statistico, se le chiavi si usano a lungo), il sistema è utilizzabile anche da sistemi separati dalla WAN (su internet, ecco). Pensate al computer di casa e quello dell'ufficio, per esempio. O un altro pc da qualche altra parte del mondo, cambia nulla.

## Dettagli implementazione
Passiamo alla realizzazione pratica: chiameremo **A**-server la macchina server e **B**-client la macchina client (indicherò con **A**-server e **B**-client anche gli IP/nomi_dns delle macchine): il server conterrà il repo sul quale pusheremo le modifiche, il nostro backup insomma, che potremmo anche clonare in altre macchine; il repository sul server sarà di tipo **bare repo**, cioè conterrà i files codificati nei vari snapshots, ma nessuna cartella di lavoro per l'utente.

Il client, oltre alla propria copia locale del repository, contenente tutto quanto è presente nel **bare repo** (ma nella cartella locale nascosta .git), e ANCHE la cartella di lavoro (che tecnicamente è una copia dell'ultimo snapshot).
Usiamo repository **bare** sul server per risparmiare spazio manca l'ulteriore clone dell'ultimo snapshot, detto "working tree". 

#### Warning! 
Questo non è un howto per utenti della prima ora, quindi molti passaggi che sono dipendenti dall'implementazione e del vostro sistema operativo verranno omessi: descrivo **COSA** fare, ma non **COME** fare; se avete letto una RFC nella vostra vita (o un libro di analisi matematica), non dovreste avere problemi.

### SSH pairing
Per prima cosa, dobbiamo permettere alle due macchine di comunicare fra loro, naturalmente via ssh, senza dover inserire ogni volta username e password: a tale scopo, permetteremo il collegamento tramite pairing di chiavi RSA. Creiamo le chiavi sul client; accediamo ad essa e nel terminale digitiamo:

{% highlight css %}
user@B-client: ssh-keygen -t rsa
{% endhighlight %}
L'output confermerà la creazione. Creiamo quindi la cartella di destinazione sul server
{% highlight css %}
user@B-client: ssh user@A-server mkdir -p .ssh
{% endhighlight %}
e trasferiamo la chiave appena creata
{% highlight css %}
user@B-client: cat .ssh/id_rsa.pub | ssh user@A-server 'cat >> .ssh/authorized_keys'
{% endhighlight %}
assegnando i permessi corretti
{% highlight css %}
user@B-client: ssh user@A-server "chmod 700 .ssh; chmod 640 .ssh/authorized_keys"
{% endhighlight %}
verificate che sia possibile effettuare il login senza inserire credenzialia.
### Creazione dei repo
Creiamo quindi il repo in una cartella della macchina A-server; dopo esserci loggati in essa e aver creato la cartella /path/to/repo_server/, eseguiamo:
{% highlight css %}
user@A-server: git init --bare /path/to/repo_server/
{% endhighlight %}
Questo sarà il nostro repository remoto. La configurazione sul server, ammesso di aver configurato correttamente ssh, firewall e il resto del networking, finisce qui… quindi, passiamo sul client! Creiamo il repository sul client, esattamente come fatto sul server:
{% highlight css %}
user@B-client: git init /path/to/repo_client/
{% endhighlight %}
e adesso, recidiamo il nodo gordiano: aggiungiamo l'origine remota al nostro repository. Ciò risponde alla domanda, "dove verrano mandati dati" da parte del 
client (che, ricordiamolo, è il sistema sul quale lavoriamo e del quale vogliamo avere un backup distribuito). Notare che nella riga sottostante sto usando una porta diversa dalla standard per ssh:
{% highlight css %}
user@B-client: git remote add A-server ssh://user@A-server:60001/path/to/repo_server/
{% endhighlight %}
Adesso proviamo a creare un file, fare un commit e poi l'output di
{% highlight css %}
user@B-client: git push A-server
{% endhighlight %}
dovrebbe confermare il funzionamento di quanto configurato, trasferendo gli elementi dal repo locale a quello remoto. Anche git status da ora in avanti terrà traccia dello status del repository remoto, segnalandovi quandi commit non avete ancora uploadato ecc. Una ulteriore conferma può essere data provando a clonare il repository remoto in locale:
{% highlight css %}
git clone user@A-server:60001/path/to/repo_server/
{% endhighlight %}
### Automazione
Adesso che abbiamo un repository distribuito, possiamo aggiungere altri nodi-server o iniziare a cooperare con altri soggetti (anche non in LAN!) utilizzando il bare-repo come piattaforma per sviluppare qualsiasi progetto; ma sopratutto, possiamo rendere più ergonomico il funzionamento del sistema con dei piccoli aliasing (da mettere nel .bashrc, o ciò che usa la vostra shell in proposito) ad esempio possiamo syncare dei repo multipli remoti e/o locali, reindirizzando l'output dell'avvenuta (o meno) sincronizzazione ad un file di log:
{% highlight css %}
alias syncall='git push A-server 2>> /path/to/log.txt && git push B-server 2>> /path/to/log.txt && git push C-server 2>> /path/to/log.txt`
{% endhighlight %}
un'altra cosa interessante, possono essere dei log arricchiti:
{% highlight css %}
alias glog='git log --stat --max-count=10'
{% endhighlight %}
## Qualche appunto pratico sulla sicurezza
Cambiate SEMPRE la porta di default di ssh. SEMPRE. È il servizio più attaccato su sistemi UNIX-like. Anche se siete su CentOS e dovrete metter mano alle policy di SELINUX, fatelo. E mettete una porta alta, non standard. Non disattivate SELINUX o il firewall, per carità; sono cose che si configurano una tantum e poi funzionano sempre, mentre un'intrusione potrebbe rovinarvi, per sempre.

Cambiate spesso le chiavi ssh se lavorate sulla rete internet. Ovviamente, non usate la stessa chiave per uploadare la nuova: fatevi un account a parte solo per queste operazioni, che usere solamente per cambio chiavi e compiti amministrativi: per i push in generale passa molta più roba, ed è attaccabile senza esagerate difficoltà se vi sniffano con una certa costanza. Magari un giorno scriverò anche di questo.

Data la delicatezza dello scambio chiavi, sarebbe opportuno eseguire la prima sincronizzazione stando fisicamente collegati (o almeno in LAN) con il server, se il repo contiene dati sensibili.
Tenere tutto su ambienti virtualizzati con vm generalmente scollegate/spente che si attivano alla bisogna è sempre consigliabile.
