# Bangladesh Board Examination Emergency Network (BBEEN)

A Cisco Packet Tracer simulation of a nationwide emergency network for the Bangladesh Board of Examinations — connecting the National Education Board HQ, Divisional Board Offices, District Exam Centers, Secure Printing Presses, and an Emergency Distribution Centre.

---

## 📖 Overview

The **BBEEN** project designs and simulates a fault-tolerant WAN topology that supports the secure flow of examination materials and control data across Bangladesh. The network is built around a central **National Education Board (NEB) HQ** with redundant WAN links to regional sites, enabling reliable communication even if a primary path fails.

The topology is implemented in **Cisco Packet Tracer** using Cisco IOS CLI for routers and GUI-based services for servers.

---

## 🗺️ Network Topology

```
                              ┌─────────────────────┐
                              │   NEB HQ (22.99.0.0/21)  │
                              │  DNS/Web + Mail Srv  │
                              └──────────┬──────────┘
                                         │
                     ┌───────────────────┼───────────────────┐
                     │                   │                   │
              Serial0/0/0          Serial0/0/1         Serial0/1/0
              (Primary)            (Backup)            (To EDC)
                     │                   │                   │
                     ▼                   ▼                   ▼
            ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
            │  DIV-Router    │  │  (Backup Path) │  │   EDC-Router   │
            │  22.99.24.138  │  │  22.99.24.142  │  │  22.99.24.146  │
            └───────┬────────┘  └────────────────┘  └────────────────┘
                    │
        ┌───────────┴───────────┐
        │   Backbone Switch     │
        │  22.99.24.128/29      │
        └───┬───────────────┬───┘
            │               │
      ┌─────▼─────┐   ┌─────▼─────┐
      │ DIS-Router│   │PRINT-Router│
      │22.99.24.130│  │22.99.24.131│
      └─────┬─────┘   └─────┬─────┘
            │               │
   ┌────────┴────────┐  ┌───┴──────────────┐
   │ District Admin  │  │ Secure Printing  │
   │ District Exam   │  │ Press + Packaging│
   └─────────────────┘  └──────────────────┘
```

---

## 🌐 Site & IP Addressing Summary

| Site | Network(s) | Gateway | Assignment |
|------|-----------|---------|------------|
| **NEB HQ** | 22.99.0.0/21 | 22.99.0.1 | Static |
| **District Admin** | 22.99.8.0/22 | 22.99.8.1 | Router DHCP |
| **District Exam Cell** | 22.99.12.0/22 | 22.99.12.1 | Router DHCP |
| **Divisional Exam Control** | 22.99.16.0/23 | 22.99.16.1 | Router DHCP |
| **Divisional Monitoring** | 22.99.18.0/23 | 22.99.18.1 | Router DHCP |
| **Printing Zone** | 22.99.20.0/23 | 22.99.20.1 | Server DHCP |
| **Packaging Unit** | 22.99.22.0/23 | 22.99.22.1 | Server DHCP (Relay) |
| **EDC** | 22.99.24.0/25 | 22.99.24.1 | Static |
| **Backbone** | 22.99.24.128/29 | — | Static |
| **WAN Primary** | 22.99.24.136/30 | — | Static |
| **WAN Backup** | 22.99.24.140/30 | — | Static |
| **WAN to EDC** | 22.99.24.144/30 | — | Static |

---

## 🧩 Key Design Decisions

1. **Host Distribution** – Device counts per main unit were split 50/50 between sub-units (e.g., Divisional Board Office: 500 Exam Control + 500 Monitoring).
2. **Server Placement** – `mail.neb.gov.bd` resides on the NEB LAN (22.99.0.3); `mail.print.gov.bd` resides on the Printing Press LAN (22.99.20.3).
3. **Packet Tracer Limitation** – Generic servers do not support Cisco IOS CLI. DNS, HTTP, DHCP, and Email services were configured via the **GUI Services tab**.
4. **Backbone Switch Port Usage** – The 2960-24TT has only 2 GigabitEthernet ports: DIV-Router and DIS-Router use Gig0/1 & Gig0/2; PRINT-Router uses Fa0/1.
5. **WAN Clocking** – NEB-Router acts as the **DCE** on all serial links with `clock rate 64000`.
6. **DHCP Exclusions** – The first 10 usable IPs in each DHCP pool are excluded for reserved/static hosts.

---

## ⚙️ Configuration Highlights

### Routing Protocols
- **Static routing** – Used at NEB-Router, EDC-Router, and for floating backup routes.
- **RIPv2** – Used between DIV-Router, DIS-Router, and PRINT-Router. `no auto-summary` enabled. WAN and LAN interfaces set to `passive-interface` where appropriate.
- **Floating Static Routes** – Administrative Distance (AD) manipulated to create primary/backup paths (e.g., AD 10 and AD 16 on PRINT-Router).

### DHCP Strategy
| Location | Method |
|----------|--------|
| District, Divisional | Router-based DHCP pools |
| Printing Zone | Server-based DHCP (PRINT-DHCP-Server) |
| Packaging Unit | DHCP Relay (`ip helper-address 22.99.20.2`) |
| EDC & NEB HQ | Static IPs only |

### DNS Records (on NEB-DNS-Web-Server)
| Hostname | Type | IP |
|----------|------|----|
| www.educationboard.gov.bd | A | 22.99.0.2 |
| mail.neb.gov.bd | A | 22.99.0.3 |
| mail.print.gov.bd | A | 22.99.0.3 |
| print.gov.bd | A | 22.99.0.3 |
| neb.gov.bd | A | 22.99.0.3 |

### Email Servers
- **NEB-Mail-Server** (`neb.gov.bd`) – SMTP & POP3 enabled.
- **PRINT-Mail-Server** (`print.gov.bd`) – SMTP & POP3 enabled.

---

## ✅ Validation & Testing

Connectivity was verified using ICMP ping tests from multiple sources. All tests returned **100% success rate (5/5)**.

| Source | Destination | Result |
|--------|------------|--------|
| NEB-Router | EDC (22.99.24.146) | ✅ 100% |
| NEB-Router | DIV Primary (22.99.24.138) | ✅ 100% |
| NEB-Router | DIV Backup (22.99.24.142) | ✅ 100% |
| NEB-Router | NEB DNS Server (22.99.0.2) | ✅ 100% |
| NEB-Router | Divisional LAN (22.99.16.1) | ✅ 100% |
| NEB-Router | District LAN (22.99.8.1) | ✅ 100% |
| NEB-Router | Print Zone (22.99.20.1) | ✅ 100% |
| NEB-Router | EDC LAN (22.99.24.1) | ✅ 100% |
| EDC-Router | NEB DNS Server (22.99.0.2) | ✅ 100% |
| DIS-Router | NEB DNS Server (22.99.0.2) | ✅ 100% |
| PC | NEB DNS Server (22.99.0.2) | ✅ 0% loss |

**Sample ping output (NEB → EDC):**
```
NEB-Router>ping 22.99.24.146
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/41/54 ms
```

**Routing tables** confirm correct static, RIP, and connected routes on DIV-Router and PRINT-Router, including floating static backups.

---

> **Note:** Upload the `.pkt` file to open the full simulation in Cisco Packet Tracer.

---

## 🚀 How to Run

1. Install **Cisco Packet Tracer** (v8.x or later recommended).
2. Open `BBEEN.pkt`.
3. Allow the network to converge (RIP updates every 30s).
4. Test connectivity:
   - From NEB-Router: `ping 22.99.24.146`
   - From any PC: `ping 22.99.0.2`
   - Open the web browser on a PC and navigate to `www.educationboard.gov.bd`.
   - Configure an email client on a PC using `mail.neb.gov.bd` / `mail.print.gov.bd`.

---

## 🛠️ Troubleshooting Tips

| Issue | Check |
|-------|-------|
| No RIP updates | `network 22.0.0.0` on all RIP routers; `no auto-summary` |
| DHCP not working at Packaging | `ip helper-address 22.99.20.2` on PRINT-Router Gi0/1 |
| Floating static not taking over | Verify AD values (lower AD = preferred) |
| Serial link down | Ensure DCE side has `clock rate 64000` |
| DNS not resolving | Verify A records and that DNS service is ON |

---

## 📜 License

This project was created for academic purposes as part of **CSE421 – Computer Networks**. Feel free to reference it for learning, but please credit the original authors.
