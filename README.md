# 🛡️ Network Traffic Analysis & Intrusion Investigation

This repository documents our hands-on work in **packet analysis** and **intrusion investigation**.  
Each project is a self-contained case study with clear objectives, concise tooling, and repeatable steps.

Each project includes:
- Objectives
- Tools & techniques used
- Step-by-step walkthroughs
- Sample outputs
- Key takeaways

---

## 🔬 Project Index

| #  | Project Title                                  | Domain / Techniques Covered                                                                  | Link |
|----|------------------------------------------------|-----------------------------------------------------------------------------------------------|------|
| 01 | Using Network Sniffers (Wireshark)             | PCAP capture, display filters, Follow TCP/HTTP Stream, DNS lookups, TTL/ARP reasoning        | [View Project](./01-using-network-sniffers/README.md) |
| 02 | Network Incident Investigation & Remediation   | PCAP ↔ Windows logs correlation, Event 4720 (user created), process/account containment      | [View Project](./02-network-incident-investigation-remediation/README.md) |

---

## 🧭 What we do in each project

- **Investigate:** capture/ingest traffic, apply Wireshark **display filters** and **Follow Stream** to extract evidence fast.  
- **Correlate:** pivot between **PCAP** (e.g., Security Onion) and **host logs** (Windows Event Viewer) to explain *what/where/who/how*.  
- **Respond:** stop malicious processes, disable rogue accounts, and validate with targeted recaptures/queries (no more flows to the egress IP).  
- **Document outcomes:** short success criteria after key steps (e.g., “no `WinT0Ols` process”, “no traffic to 75.30.5.55”).

---

## 🧰 Tools & Techniques (short list)

- **Wireshark** — display filters; Follow TCP/HTTP Stream for rapid content validation when TLS is absent.  
- **Security Onion** — PCAP access and pivots for packet-level validation.  
- **Windows Event Viewer** — focus on **User Account Management** events (e.g., **4720** for user creation).

---

## 🌍 Connect With Me
- [LinkedIn](https://www.linkedin.com/in/zakaria-a-432624154/)

---

## 📢 Disclaimer
This repository is for educational purposes only.  
All activities were performed in a controlled, legal lab environment.  
Do **not** attempt these techniques on unauthorized systems.

