# Project Lab

Taccuino tecnico per idee, prototipi e progetti in corso nell'ambito della **Cyber-Physical Security** e dell'ingegneria dei sistemi. Ogni progetto ha una cartella autonoma con playbook, obiettivi, piano di validazione e spazio per documentare risultati e decisioni.

## Progetti

| Progetto | Ambito | Stato |
|---|---|---|
| [Automotive CAN-Bus Monitor & IDS](projects/automotive-can-bus-monitor-ids/PROJECT_PLAYBOOK.md) | CAN, C/C++, ESP32/Linux vcan, rilevamento anomalie | Pianificato |
| [IoT Hardware Security & Firmware Auditing Station](projects/iot-hardware-firmware-auditing/PROJECT_PLAYBOOK.md) | UART, SPI flash, firmware, reverse engineering | Pianificato |
| [Linux Packet Sniffer & Host-IDS](projects/linux-packet-sniffer-host-ids/PROJECT_PLAYBOOK.md) | AF_PACKET, eBPF, C++20, telemetria host | Pianificato |

## Come usare questo repository

1. Scegli un progetto dal [backlog](BACKLOG.md) e aggiorna il suo stato quando inizi.
2. Segui il playbook come piano di studio e sviluppo; registra ipotesi, risultati, misure e problemi incontrati.
3. Mantieni riproducibili setup e test. Non inserire credenziali, dump di terzi o dati personali.
4. Aggiungi una nuova cartella sotto `projects/` per ogni nuova idea, con un `PROJECT_PLAYBOOK.md` e una breve sezione nel backlog.

## Principi di lavoro

- **Sicurezza e autorizzazione:** test solo su hardware proprio, reti isolate o sistemi per cui si dispone di autorizzazione esplicita.
- **Misure prima delle affermazioni:** pubblicare risultati con ambiente, metodo, limiti e dati riproducibili.
- **Progressione didattica:** iniziare con simulatori e banchi controllati; passare all'hardware reale solo dopo aver compreso i limiti elettrici e i rischi.
- **Portfolio trasparente:** distinguere chiaramente attività pianificate, prototipi e risultati effettivamente verificati.

## Convenzioni

Cartelle progetto in `kebab-case`; documentazione primaria in Markdown; codice, schemi e dati di test separati e accompagnati da istruzioni di riproduzione. Usare licenze appropriate prima di riutilizzare codice o pubblicare contributi di terzi.
