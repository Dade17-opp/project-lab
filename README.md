# Project Lab

An engineering portfolio focused on **Cyber-Physical Security**, embedded systems, and Linux security tooling. This repository collects project plans and, as they develop, implementations, experiments, and measured results.

> **Current status:** the projects listed below are planned. Their playbooks define technical study and implementation paths; they are not claims of completed work or validated results.

## Projects

| Project | Focus | Current stage |
|---|---|---|
| [Automotive CAN-Bus Monitor & IDS](projects/automotive-can-bus-monitor-ids/PROJECT_PLAYBOOK.md) | C/C++, Linux SocketCAN/`vcan`, ESP32 TWAI, anomaly detection | Planning |
| [IoT Hardware Security & Firmware Auditing Station](projects/iot-hardware-firmware-auditing/PROJECT_PLAYBOOK.md) | UART, SPI flash acquisition, Binwalk, firmware analysis | Planning |
| [Linux Packet Sniffer & Host-IDS](projects/linux-packet-sniffer-host-ids/PROJECT_PLAYBOOK.md) | C++20, Linux `AF_PACKET`, eBPF, packet analysis | Planning |

## Repository structure

```text
project-lab/
├── BACKLOG.md
└── projects/
    ├── automotive-can-bus-monitor-ids/
    ├── iot-hardware-firmware-auditing/
    └── linux-packet-sniffer-host-ids/
```

Each project directory contains a `PROJECT_PLAYBOOK.md` organized around system fundamentals, lab setup, architecture, implementation sequence, and validation. As work progresses, it can also include source code, test fixtures, diagrams, and experiment notes.

## Engineering approach

- Start with isolated simulations and lab equipment; use only systems and devices owned by me or explicitly authorized for testing.
- Prefer small, reproducible experiments and explainable detection logic.
- Record the environment, method, measurements, and limitations alongside results.
- Keep credentials, personal data, and sensitive firmware or packet captures out of the repository.
- Mark project stages honestly so plans, prototypes, and verified results are easy to distinguish.

## Languages

This overview and the roadmap are in English. The detailed technical playbooks are currently in Italian. Implementation languages and tools are selected per project and documented in each playbook.
