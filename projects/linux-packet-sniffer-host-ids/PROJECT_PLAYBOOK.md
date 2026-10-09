# Linux Packet Sniffer & Host-IDS

**Stato:** Pianificato · **Focus:** sensore passivo e allarmi spiegabili · **Stack:** Linux `AF_PACKET`, C++20, eBPF/libbpf come fase avanzata.

## 1. Anatomia & Studio del Sistema

### Obiettivo e modello

Implementare un collector Linux per pacchetti e segnali locali, con parser robusti, eventi normalizzati e regole configurabili. Il primo prototipo è passivo su loopback o rete di laboratorio autorizzata. Non implementa blocco, spoofing, scansione o modifica del traffico. L'esecuzione iniziale privilegia privilegi minimi; un percorso eBPF privilegiato è separato e opzionale.

### Teoria, kernel e protocolli

- **AF_PACKET:** socket di livello link (`SOCK_RAW` include header L2, `SOCK_DGRAM` fornisce frame cooked, secondo configurazione/kernel), binding a interfaccia, protocollo di rete e filtro. Richiede capability/permessi adeguati, ad esempio `CAP_NET_RAW`; non eseguire come root per comodità.
- **Percorso pacchetto:** NIC/driver → stack NAPI/kernel → packet socket → buffer applicativo. VLAN offload, GRO/LRO, loopback e driver possono cambiare ciò che il collector osserva. Documentare interfaccia e offload del test.
- **Parsing:** Ethernet, VLAN, IPv4/IPv6, ARP e TCP/UDP/ICMP. Non fidarsi di lunghezze dichiarate: validare ogni offset e gestire frammentazione, extension header IPv6, pacchetti troncati e checksum offload.
- **C++20:** RAII, tipi bounded/span, ownership chiara, thread e code limitate. Evitare copie premature e accessi non allineati o reinterpret cast insicuri.
- **eBPF:** programmi verificati dal verifier, mappe, hook disponibili (TC/XDP/tracepoint/socket filter) e compatibilità legata a kernel/configurazione. Verificare GPL/licensing e API della versione libbpf. Introdurlo solo dopo aver definito il collector userspace.
- **Host-IDS:** distinguere osservazione rete da telemetry host; correlare eventi senza promettere copertura completa. Allarmi basati su indicatori espliciti, rate limit e audit trail.

### Normative e fonti

Consultare documentazione kernel Linux per packet sockets, capability, eBPF e networking; RFC pertinenti a Ethernet, IPv4, IPv6, TCP, UDP e protocolli implementati. Considerare NIST SP 800-94 come contesto storico IDS/IPS e MITRE ATT&CK per descrivere tecniche, non per dichiarare copertura. Per dati di rete reali, definire minimizzazione, retention e autorizzazione coerenti con policy e normativa applicabile (incluso GDPR dove pertinente). Non dichiarare certificazioni.

## 2. Attrezzi & Banco di Lavoro

- Linux con kernel registrato, interfaccia loopback e rete di test isolata; VM/container per generare traffico innocuo.
- C++20, CMake, GCC/Clang, `gdb`, `strace`, `perf`, `tcpdump`/Wireshark come riferimento, `iproute2` e `ethtool` per ispezionare interfaccia/offload.
- Librerie standard iniziali; introdurre libbpf solo nella fase eBPF con dipendenze/versioni documentate.
- Test unitari per parser su buffer costruiti; corpus PCAP sintetico senza dati personali. Sanitizers ASan/UBSan/TSan secondo componenti.
- CI per build, static analysis e test senza privilegi; test privilegiati documentati e manuali su VM dedicata.
- Output JSONL o formato stabile con timestamp, interfaccia, tuple normalizzate e motivazione. Evitare payload completi nei log per default.

## 3. Disassemblaggio Logico

```text
interfaccia autorizzata
        │ Ethernet frame / metadata kernel
        ▼
AF_PACKET receiver ── bind, timeout, filtro, buffer bounded
        │
        ▼
parser L2 → L3 → L4 (controllo rigoroso dei bounds)
        │
        ├──► flow/event normalizer ── coda bounded
        │                              │
        │                              ▼
        │                       regole/statistiche IDS
        │                              │
        └── eBPF opzionale             ▼
           (hook + mappe)        allarmi JSONL/metriche
                                      │
                                shutdown e report
```

Moduli: `CaptureSource`, `PacketView` non owning e validato, parser puri, `Event`, `RuleEngine`, `Sink` e `Metrics`. Separare acquisizione da analisi consente replay da file e test senza privilegi. Il ramo eBPF deve avere contatori di perdita e percorso fallback esplicito.

## 4. Sequenza Operativa Step-by-Step

1. **Definire scope e threat model:** interfaccia, dati osservati, retention, ambiente autorizzato, regole e limiti del sensore.
2. **Preparare ambiente:** registrare distro, kernel, compiler e tool. Avviare VM/rete isolata; usare loopback per il primo test.
3. **Stabilire reference:** catturare traffico sintetico con tcpdump/Wireshark e documentare offload, MTU, VLAN e interfaccia.
4. **Definire schema eventi:** timestamp monotonic e wall-clock distinti, interfaccia, lunghezze, tuple, protocollo e flag di troncamento; minimizzare payload.
5. **Implementare parser puri:** bounds check per Ethernet/VLAN/IP/TCP/UDP; ritornare errore tipizzato su input incompleto o incoerente.
6. **Testare parser prima della socket:** pacchetti minimi, massimi, frammentati, troncati, VLAN multiple secondo supporto, IPv6 extension header e valori overflow.
7. **Aggiungere acquisizione AF_PACKET:** binding a una singola interfaccia, timeout, gestione EINTR/ENOBUFS, buffer limitato, stop signal e chiusura RAII.
8. **Least privilege:** sviluppo con capability minime su binario dedicato o configurazione controllata; documentare permessi senza eseguire tutto come root.
9. **Aggiungere pipeline:** code bounded, contatori drop e backpressure; serializzare solo eventi necessari e limitare rate degli allarmi.
10. **Definire regole baseline:** scansioni anomale nel dataset sintetico, volumi/soglie e correlazioni locali con motivazione esplicita. Evitare azioni automatiche.
11. **Misurare:** throughput, drop, CPU, RSS e latenza a vari packet rate e dimensioni; confrontare col riferimento indipendente.
12. **Valutare eBPF:** solo dopo aver definito kernel target e feature; prototipo minimale di conteggio/filtraggio, gestione mappe, verifier log, fallback e test di compatibilità.
13. **Pubblicare documentazione:** comandi di build/run, modello dati, minacce coperte, limiti, privilegi, dataset e risultati misurati.

Avviare il primo test su `lo`; per test di rete usare namespace o VM isolati. L'accesso packet socket e l'installazione di programmi eBPF possono richiedere privilegi: limitarli alla macchina di laboratorio e seguire documentazione della distribuzione/kernel.

## 5. Tolleranze, Collaudo & Rodaggio

- **Parser:** fuzzing e corpus per lunghezza zero, header parziali, IHL/DL errati, overflow, frammenti sovrapposti, IPv6 extension chain lunga e protocolli ignoti. Nessun panic, OOB o UB.
- **Socket:** interfaccia rimossa, permesso negato, socket chiusa, EINTR, burst, ring/buffer saturato e pacchetti troncati. Esportare drop reason e contatori.
- **Memoria e concorrenza:** RAII, code con cap, evitare crescita per flow mai chiusi; ASan/UBSan e TSan dove applicabile; verificare shutdown anche sotto carico.
- **Edge case kernel/NIC:** VLAN offload, GRO/LRO, MTU jumbo, loopback, timestamp differente, interfaccia rinominata e mismatch tra capture point e reference.
- **Privacy e logging:** payload non registrato per default; redigere identificatori sensibili nei fixture; limiti di retention e rotazione; proteggere PCAP perché possono contenere dati personali.
- **eBPF:** verifier rejection, map piena, attach fallito, feature assente su kernel target, permessi limitati, detach pulito e aggiornamento del programma senza perdita incontrollata.
- **Metriche di accettazione:** p50/p95/p99 latenza, packet drop, CPU/RSS, eventi persi e precision/recall su scenari dichiarati. Stabilire soglie dopo definizione dell'hardware e del carico.
- **Limiti da dichiarare:** visibilità dipendente dal punto di cattura e offload, traffico cifrato limita ispezione payload, eBPF varia tra kernel, IDS euristico non è garanzia di rilevamento e il prototipo non previene attacchi.
