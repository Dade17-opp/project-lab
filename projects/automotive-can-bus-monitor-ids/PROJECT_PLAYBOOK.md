# Automotive CAN-Bus Monitor & IDS

**Stato:** Pianificato · **Focus:** acquisizione passiva, analisi e rilevamento interpretabile · **Stack:** C/C++, Linux SocketCAN/`vcan`, ESP32 TWAI come estensione.

## 1. Anatomia & Studio del Sistema

### Obiettivo e confini

Costruire un monitor che riceve frame CAN, normalizza timestamp e identificatore, mantiene contatori e segnala deviazioni da un profilo di traffico noto. Il primo target è esclusivamente `vcan`, una rete virtuale locale. Il progetto non controlla attuatori e non trasmette su reti veicolo. L'ESP32 è una fase successiva su banco isolato, con transceiver e terminazione corretti.

### Teoria e livelli

- **Fisico:** CAN_H/CAN_L differenziale, stato dominante/recessivo, terminazione tipica alle estremità del bus. Un ESP32 non si collega direttamente al bus: serve un transceiver CAN adeguato.
- **Collegamento:** arbitraggio CSMA/CR, frame standard 11 bit ed esteso 29 bit, DLC, payload, CRC, ACK, error frame e gestione degli errori. CAN classico porta fino a 8 byte; CAN FD ha regole e capacità diverse e va trattato esplicitamente.
- **Trasporto applicativo:** CAN non definisce da solo il significato dei segnali. Database DBC e protocolli superiori possono fornire semantica, ma in questo progetto iniziale si analizzano identificatori, periodicità, lunghezza e distribuzione dei payload.
- **SocketCAN:** interfaccia Linux a frame CAN tramite socket `PF_CAN`, filtro kernel, interfacce nominali e virtuali (`vcan`). Le capacità dipendono da kernel, driver e tool installati.
- **IDS:** separare raccolta, estrazione feature, baseline, decisione e reporting. Iniziare con soglie spiegabili, non con un classificatore opaco.

### Normative e fonti da studiare

Consultare le versioni correnti e il materiale didattico disponibile per ISO 11898 (livelli e data link CAN), ISO 14229/15765 solo se si estende lo studio a diagnostica/UDS, documentazione Linux SocketCAN e datasheet del transceiver scelto. Per sicurezza automotive, usare ISO/SAE 21434 come riferimento di processo e ISO 26262 per il contesto safety; non presentare questo prototipo come conforme o certificato. Verificare licenze e condizioni d'uso di ogni specifica.

### Domande da risolvere

Quale baseline rappresenta il traffico legittimo? Come distinguere avvio, idle e carico? Come gestire clock monotonic vs realtime? Quale falsi positivi per ora è accettabile nel dataset di laboratorio?

## 2. Attrezzi & Banco di Lavoro

### Fase virtuale

- PC Linux o VM Linux con supporto SocketCAN; kernel con driver `vcan`.
- `iproute2` (`ip`), `can-utils` (`candump`, `cansend`, `cangen`), GCC/Clang, CMake o Make, Git.
- Test unitari per decoder e feature; sanitizers AddressSanitizer e UndefinedBehaviorSanitizer durante sviluppo.
- Dataset sintetico versionato: scenari normali, burst, ID inatteso, payload variabile, frame troncato/invalidato dal parser.

### Fase hardware opzionale

- ESP32 con periferica TWAI/CAN supportata dal modello, transceiver CAN compatibile e alimentazione stabile.
- Banco separato dal veicolo: due nodi di test, cablaggio corto, terminazione verificata, alimentatore con limite di corrente.
- Analizzatore logico per segnali UART/debug; oscilloscopio e sonda differenziale solo se disponibili e usati correttamente.
- Verificare datasheet di MCU/transceiver, tensioni logiche, pinout, isolamento e stato di fail-safe prima di collegare.

### Toolchain e debug

Linux C++17/20 (scegliere e documentare lo standard), `gdb`, `strace`, `perf` per misure successive, `clang-tidy` e formatter. Usare `candump` come confronto indipendente. Non effettuare prove su veicoli o bus pubblici.

## 3. Disassemblaggio Logico

```text
vcan / bus di banco
        │ frame + timestamp
        ▼
SocketCAN receiver ── filtro per interfaccia/ID
        │
        ▼
validazione e normalizzazione
        │
        ├──► contatori e finestre temporali
        ├──► feature per ID (frequenza, DLC, payload)
        ▼
baseline + regole interpretabili
        │
        ├──► eventi JSONL con motivazione
        └──► metriche: perdita, latenza, CPU, memoria
```

Componenti: `can_source` incapsula la socket; `Frame` è un tipo con ID, flag esteso, DLC, bytes e timestamp; `FeatureWindow` aggiorna statistiche per intervallo; `RuleEngine` restituisce codici motivazione; `Reporter` emette console/JSONL. Isolare le dipendenze hardware così i test girano senza CAN fisico.

## 4. Sequenza Operativa Step-by-Step

1. **Definire il threat model:** monitor solo, asset protetto, attaccante ipotetico, limiti e criterio di successo. Registrare che il sistema non è safety-critical.
2. **Preparare Linux:** installare strumenti dai repository della distribuzione; registrare OS, kernel, compilatore e versioni.
3. **Creare `vcan0`:** caricare il modulo se necessario, creare e attivare l'interfaccia virtuale. Verificare la configurazione con `ip link`.
4. **Validare il banco software:** usare `candump` e inviare frame sintetici a `vcan0`; confermare che gli ID e payload attesi arrivino.
5. **Definire il formato dati:** distinguere CAN classico/FD, ID standard/esteso, lunghezze ammissibili, timestamp monotonic e gestione degli error frame.
6. **Scrivere un ricevitore minimale:** bind alla sola interfaccia scelta, gestire segnali di stop, errori socket e timeout senza blocchi indefiniti.
7. **Aggiungere test parser:** casi limite ID, DLC, frame FD, flag, input malformati e conversioni. Mantenere funzioni pure testabili.
8. **Implementare baseline:** calcolare intervallo di arrivo e frequenza per ID su finestre; esporre configurazione delle soglie e motivi degli allarmi.
9. **Generare scenari ripetibili:** traffico periodico normale, ID sconosciuto, burst e variazioni di lunghezza; salvare seed e parametri.
10. **Valutare:** precision, recall, falsi allarmi, latenza detection e uso risorse; separare i dati di calibrazione da quelli di verifica.
11. **Documentare:** diagramma, build/run, esempi sintetici, limiti e risultati effettivi. Aggiungere CI per build e test se disponibile.
12. **Solo dopo, valutare ESP32:** collegare esclusivamente il banco isolato, prima in ricezione, controllare tensioni/terminazione e confrontare gli stessi vettori di test.

Esempio di setup virtuale (Linux):

```sh
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
ip -details link show vcan0
```

Rimuovere l'interfaccia di test al termine: `sudo ip link delete vcan0`.

## 5. Tolleranze, Collaudo & Rodaggio

- **Correttezza:** ID standard/esteso, CAN FD se supportato, frame con DLC limite, error frame, socket chiusa, interfaccia assente e stop pulito.
- **Robustezza:** burst sostenuti, ID cardinality elevata, timestamp non monotoni simulati, buffer socket pieno, input non valido e rotazione log.
- **Memoria:** eseguire ASan/UBSan, verificare crescita delle mappe per ID ignoti e imporre limiti/TTL alle chiavi; controllare leak nel loop di ricezione.
- **Prestazioni:** misurare tasso di frame, CPU, RSS, latenza p50/p95/p99 e drop sotto carico. Riportare macchina, kernel, durata e metodo.
- **Qualità del rilevamento:** confusione per scenario, falsi positivi, falsi negativi e tempo al rilevamento; evitare di addestrare e valutare sugli stessi campioni.
- **Edge case:** ripartenza interfaccia, cambio configurazione, clock adjustment, ID legittimo raro, traffico a fasi e contatori overflow.
- **Criteri iniziali:** nessun crash per una sessione di stress definita, memoria stabilizzata entro limiti documentati, nessun allarme spurio sui casi normali del dataset e rilevazione degli scenari anomali dichiarati. I valori numerici vanno scelti e motivati dopo aver misurato il carico target.
- **Limiti da dichiarare:** payload opachi senza DBC, dataset sintetico non equivalente a flotta, possibili falsi positivi, nessuna garanzia di prevenzione o conformità automotive.
