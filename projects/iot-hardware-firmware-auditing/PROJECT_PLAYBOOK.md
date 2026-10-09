# IoT Hardware Security & Firmware Auditing Station

**Stato:** Pianificato · **Focus:** acquisizione autorizzata e analisi offline · **Stack:** UART, SPI flash, hash, Binwalk, reverse engineering.

## 1. Anatomia & Studio del Sistema

### Obiettivo e confini

Allestire una postazione ripetibile per identificare componenti e interfacce su dispositivi di proprietà o per cui si ha autorizzazione esplicita, acquisire firmware senza alterare l'originale ove possibile e condurre triage offline. Non tentare accesso a dispositivi di terzi, bypass di credenziali o estrazione di segreti non autorizzata. Non collegare debugger o alimentatori finché pinout e livelli non sono verificati.

### Teoria e architettura

- **PCB e componenti:** leggere marcature, datasheet, package e pin-1; individuare MCU/SoC, memoria, regolatori, test pad e connettori. Tracce visive sono ipotesi, non conferma.
- **UART:** livelli logici tipicamente TTL, baud/parità/stop bit, TX/RX/GND. UART non è RS-232; non presumere 3,3 V o 5 V senza misura/datasheet. Collegare GND comune e incrociare TX/RX solo dopo aver verificato la direzione.
- **SPI NOR/NAND:** segnali CS, CLK, MOSI/IO, MISO/IO, capacità, tensione e protocollo dipendono dal chip. Verificare se la memoria è alimentata dal target; evitare back-powering e contention durante clip/in-circuit.
- **Acquisizione:** dump grezzo, dimensione attesa, più letture, hash SHA-256, provenienza e condizioni. Per memoria NAND considerare bad block/ECC e differenze tra dump grezzo e immagine logica.
- **Firmware:** bootloader, kernel, filesystem, configurazione, firme, metadati e componenti open source. Binwalk aiuta a identificare firme e segmenti; la classificazione automatica non prova la sicurezza o l'eseguibilità.
- **Reverse engineering:** disassembly e decompilazione dipendono da ISA, endianness, base address e simboli. Preferire prima stringhe, filesystem e metadati; analizzare codice in sandbox.

### Normative e riferimenti

Consultare datasheet dei componenti, documentazione dei programmatori e standard SPI/UART applicabili, documentazione Binwalk e del framework di analisi scelto. Per il contesto di sicurezza prodotto, studiare IEC 62443-4-1/-2-1 e, in UE, il Cyber Resilience Act e requisiti applicabili aggiornati: verificare sempre testo e applicabilità con fonti ufficiali. Un audit didattico non costituisce valutazione di conformità.

## 2. Attrezzi & Banco di Lavoro

### Hardware

- ESD mat, bracciale e contenitori antistatici; illuminazione e lente/microscopio.
- Multimetro; alimentatore da banco con limite di corrente. Oscilloscopio/analizzatore logico per misure appropriate.
- Adattatore USB-UART con selezione di livelli documentata; nessun adattatore RS-232 diretto.
- Programmer SPI supportato dal chip, clip SOIC solo con pitch compatibile; adattatori di tensione/regolazione quando previsti dal datasheet.
- Dispositivo di test proprio, documentazione d'acquisto e foto della configurazione iniziale.

### Toolchain software

Linux isolato o VM; `screen`/`picocom` per seriale, `flashrom` o utility del programmer quando supportati, `sha256sum`, `file`, Binwalk e strumenti di estrazione in ambiente usa-e-getta. Ghidra o equivalente per reverse engineering. Container/VM senza accesso a credenziali host per contenuti non fidati. Registrare versioni e hash degli strumenti/dump.

### Sicurezza del banco

Non alimentare una scheda via USB-UART e programmer contemporaneamente senza conoscere il circuito. Misurare alimentazione, orientamento e massa; rispettare i limiti ESD. Usare dump di riferimento e non scrivere mai sulla flash durante la fase di sola acquisizione.

## 3. Disassemblaggio Logico

```text
dispositivo autorizzato
   ├── osservazione visiva + identificazione chip/datasheet
   ├── UART ── adattatore a livello corretto ── terminale solo lettura
   └── SPI ── programmer compatibile ── immagine grezza immutabile
                                             │ SHA-256 + metadati
                                             ▼
                                      copia di lavoro offline
                                             │
                        identificazione → carving → filesystem
                                             │
                      triage stringhe/config → reverse engineering
                                             │
                      findings con evidenza, rischio e limiti
```

Separare archivio originale read-only, working copy e output di analisi. Ogni finding deve puntare a hash del campione, offset/percorso, tool/versione e passaggi di riproduzione. Rimuovere seriali, token, MAC e dati personali dai materiali pubblici.

## 4. Sequenza Operativa Step-by-Step

1. **Autorizzazione e scheda caso:** registrare proprietario, scopo, modello/revisione, permessi, limiti e trattamento dei dati.
2. **Fotografare e ispezionare:** documentare PCB, marcature e connettori senza alimentare; cercare datasheet e schema affidabili.
3. **Mappare i rischi elettrici:** identificare GND e rail con riferimenti certi; misurare tensioni solo con procedura sicura. Fermarsi se i livelli restano incerti.
4. **Identificare UART:** determinare pinout e livello, collegare GND/TX/RX in sola ricezione iniziale, osservare boot log a baud comuni senza inviare comandi.
5. **Identificare la flash:** leggere marcatura e datasheet, capacità, tensione e protocollo; scegliere programmer/adattatore compatibile.
6. **Acquisire in lettura:** fotografare cablaggio, leggere il chip almeno due volte secondo capacità del tool, salvare output separati e confrontare dimensione e hash.
7. **Conservare l'originale:** archivio immutabile fuori dal repository se il file è grande o sensibile; annotare SHA-256, data, metodo e strumento. Nel repository mettere solo manifest e hash quando opportuno.
8. **Creare copia di lavoro:** analizzare esclusivamente una copia; non eseguire binari estratti sul sistema host.
9. **Triage statico:** identificare tipo/architettura, eseguire Binwalk per firme, estrarre in VM usa-e-getta, inventariare filesystem e stringhe. Validare manualmente i risultati rilevanti.
10. **Approfondire:** valutare boot chain, aggiornamento, configurazioni, certificati e componenti; usare Ghidra con ISA e base address documentati. Evitare di pubblicare credenziali o exploit riutilizzabili.
11. **Scrivere il report:** evidenze, gravità motivata, impatto, riproducibilità, falsi positivi/limiti e remediation suggerita. Separare fatti da ipotesi.
12. **Pulizia e ripetibilità:** salvare manifest/tool versions, smontare il banco, proteggere o eliminare copie temporanee secondo autorizzazione e policy.

### Manifest minimo di acquisizione

```text
case_id, device_model, hardware_revision, chip_marking, acquisition_method,
tool_and_version, image_size_bytes, sha256, timestamp_utc, operator_notes
```

## 5. Tolleranze, Collaudo & Rodaggio

- **Integrità:** confrontare letture multiple byte-per-byte; identificare offset instabili, letture corte e chip ID incoerente. Non assumere che hash uguali dimostrino una procedura completa: conservare anche metadati.
- **Ripetibilità:** far ripetere l'acquisizione seguendo solo la checklist e confrontare risultati. Documentare clip in-circuit vs chip-off e possibili differenze.
- **Edge case:** memoria protetta, dump parziale, firmware compresso/cifrato, filesystem non riconosciuto, bad block, regioni vuote, UART assente o log disabilitato.
- **Sicurezza elettrica:** monitorare corrente, temperatura e reset; interrompere in caso di assorbimento anomalo, riscaldamento o comportamento inatteso. Verificare assenza di back-powering.
- **Robustezza pipeline:** nomi file ostili, archive traversal, decompression bomb e file malformati vanno processati in VM con limiti CPU, memoria e spazio. Non estrarre su directory condivise fidate.
- **Leak e risorse:** verificare che script e tool chiudano file, cancellino temp secondo policy e non copino dati sensibili nei log. Lavorare su immagini grandi senza caricarle tutte in RAM.
- **Criteri di completamento:** immagini tracciate e confrontabili, catena dei tool annotata, almeno un'analisi riproducibile e report senza segreti. Stabilire soglie quantitative solo dopo conoscere il programmer e il chip.
- **Limiti da dichiarare:** identificazione automatica incompleta, cifratura non equivale a vulnerabilità, dump non sempre rappresentativo del contenuto logico e risultati non generalizzabili a dispositivi non testati.
