# Backlog — Cyber-Physical Security & Engineering

Registro iniziale di progetti proposti. Tutti sono **pianificati**: non rappresentano implementazioni o risultati già conseguiti. Per ciascuno, il playbook descrive un percorso graduale e verificabile.

## 1. Automotive CAN-Bus Monitor & IDS

- **Obiettivo:** acquisire e analizzare traffico CAN in un ambiente di test, identificando comportamenti anomali con regole interpretabili e una baseline statistica.
- **Tecnologie:** C/C++, ESP32/TWAI o Linux SocketCAN con `vcan`; parser e pipeline di eventi.
- **Risultati attesi:** monitor in sola lettura, generatore di traffico sintetico, regole di rilevamento, test riproducibili e report di valutazione.
- **Domande di ricerca:** come distinguere variazioni legittime da anomalie? Qual è il costo in latenza e risorse? Quali limiti derivano dall'assenza di semantica dei segnali?
- **Stato:** Pianificato.
- **Playbook:** [Apri il piano tecnico](projects/automotive-can-bus-monitor-ids/PROJECT_PLAYBOOK.md).

## 2. IoT Hardware Security & Firmware Auditing Station

- **Obiettivo:** costruire una postazione di laboratorio per identificazione, acquisizione controllata e analisi statica di firmware appartenente a dispositivi propri o autorizzati.
- **Tecnologie:** UART a livelli elettrici verificati, SPI flash programmer, dump con hash, Binwalk, strumenti di reverse engineering.
- **Risultati attesi:** checklist di acquisizione non distruttiva, archivio tracciabile, pipeline di analisi offline e scheda dei risultati.
- **Domande di ricerca:** come rendere integro e ripetibile il processo di acquisizione? Quali euristiche di analisi producono falsi positivi? Come documentare firmware cifrato o compresso senza forzare l'accesso?
- **Stato:** Pianificato.
- **Playbook:** [Apri il piano tecnico](projects/iot-hardware-firmware-auditing/PROJECT_PLAYBOOK.md).

## 3. Linux Packet Sniffer & Host-IDS

- **Obiettivo:** realizzare un sensore di rete Linux con acquisizione AF_PACKET e un'estensione eBPF opzionale, per classificare eventi e produrre allarmi locali con overhead misurato.
- **Tecnologie:** C++20, Linux `AF_PACKET`, `PACKET_MMAP` come estensione, eBPF/libbpf come fase avanzata.
- **Risultati attesi:** sensore passivo, parser robusti, output JSONL, regole configurabili, test con traffico sintetico e confronto prestazionale.
- **Domande di ricerca:** quali compromessi tra copia, latenza, perdita pacchetti e complessità? Quali segnali host sono osservabili senza privilegi eccessivi?
- **Stato:** Pianificato.
- **Playbook:** [Apri il piano tecnico](projects/linux-packet-sniffer-host-ids/PROJECT_PLAYBOOK.md).

## Priorità suggerita

1. Avviare il monitor CAN con `vcan`: è possibile imparare acquisizione, parsing e test senza esporre un veicolo.
2. Sviluppare lo sniffer su loopback o rete di laboratorio, con acquisizione passiva e dataset sintetico.
3. Allestire la stazione firmware dopo aver acquisito componenti di laboratorio e procedure sicure per segnali e alimentazione.

Rivalutare la priorità in base a corsi, hardware disponibile e obiettivi di apprendimento; non considerare l'ordine come una scadenza.
