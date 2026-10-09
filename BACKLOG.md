# Project Roadmap

This roadmap lists proposed projects in Cyber-Physical Security and systems engineering. **All projects are currently in the planning stage.** The linked playbooks describe intended study and implementation work; completed features and measured outcomes will be documented only after they have been built and verified.

## Automotive CAN-Bus Monitor & IDS

- **Goal:** Build a passive monitor for CAN traffic in a controlled lab and detect deviations using interpretable rules and statistical baselines.
- **Planned stack:** C/C++, Linux SocketCAN with `vcan`, with ESP32/TWAI as a later hardware extension.
- **Planned deliverables:** A read-only monitor, synthetic traffic scenarios, documented detection rules, reproducible tests, and an evaluation report.
- **Key challenges:** Distinguishing legitimate operating modes from anomalies, measuring detection latency and resource use, and describing limits when signal semantics are unavailable.
- **Status:** Planning.
- **Technical plan:** [Automotive CAN-Bus Monitor & IDS playbook](projects/automotive-can-bus-monitor-ids/PROJECT_PLAYBOOK.md).

## IoT Hardware Security & Firmware Auditing Station

- **Goal:** Set up a repeatable lab workflow for identifying components, acquiring firmware, and performing offline static analysis on owned or explicitly authorized devices.
- **Planned stack:** UART, SPI flash programmer, verified firmware images and hashes, Binwalk, and reverse-engineering tools.
- **Planned deliverables:** A non-destructive acquisition checklist, a traceable evidence manifest, an offline analysis workflow, and a findings report.
- **Key challenges:** Preserving acquisition integrity, handling false positives in automated analysis, and documenting encrypted or compressed firmware without attempting unauthorized access.
- **Status:** Planning.
- **Technical plan:** [IoT Hardware Security & Firmware Auditing Station playbook](projects/iot-hardware-firmware-auditing/PROJECT_PLAYBOOK.md).

## Linux Packet Sniffer & Host-IDS

- **Goal:** Build a passive Linux sensor that captures and classifies traffic, then emits local alerts with measured resource overhead.
- **Planned stack:** C++20, Linux `AF_PACKET`, with eBPF/libbpf as an optional advanced capture path.
- **Planned deliverables:** A passive sensor, bounds-checked parsers, structured JSONL events, configurable rules, synthetic traffic tests, and a performance report.
- **Key challenges:** Balancing copies, latency, packet loss, and complexity; identifying useful host signals while keeping privileges limited.
- **Status:** Planning.
- **Technical plan:** [Linux Packet Sniffer & Host-IDS playbook](projects/linux-packet-sniffer-host-ids/PROJECT_PLAYBOOK.md).

## Scope and validation principles

Work will begin in simulators and isolated lab environments. Each project will document its setup, test method, measurements, limitations, and actual completion stage. Testing will be limited to systems and devices owned by me or explicitly authorized for assessment.
