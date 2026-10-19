# DEADFACE CTF 2026 — Challenges & Writeups

[![CTF](https://img.shields.io/badge/CTF-DEADFACE%202026-red.svg)](https://deadface.io)
[![Challenges](https://img.shields.io/badge/Challenges-30%20Solved-brightgreen.svg)](#challenge-index)
[![Event](https://img.shields.io/badge/Event-17--19%20Oct%202026-blue.svg)](https://deadface.io)

This repository contains detailed writeups for the **DEADFACE CTF 2026** challenges
I solved during the event (17 Oct 19:30 IST → 19 Oct 05:30 IST, ~34h online jeopardy
on [deadface.io](https://deadface.io)). Played **solo**; every flag was submitted
by hand on the platform.

All challenges are organized by category, with clean writeups covering the approach,
methodology, tools, and step-by-step process for each solve.

---

## Repository Structure

```text
DEADFACE-CTF-2026-Writeup/
├── Database/                  # Aurora Health MySQL audit logs, pharmacy & clinical trial data
├── Exploitation/              # Linux capability abuse → privilege escalation
├── Forensics/                 # DNS tunnels, JPEG metadata, PCAP exfil & memory forensics
├── Mobile-Security/           # Aurora Patient Portal iOS app forensics
├── OSINT/                     # GhostTown forum, DoomScroll API, historic squares
├── Reverse-Engineering/       # Cereal Killer series (PE32, ELF)
└── Web-Exploitation/          # Health Spark portal SQLi, exposed backend source
```

---

## Challenge Index

| Category | Challenge | Points | Writeup | Solved Flag |
|---|---|---|---|---|
| **Reverse-Engineering** | [Cereal Killer 01 — Insectoids](./Reverse-Engineering/cereal-killer-01-insectoids/README.md) | 50 | [writeup](./Reverse-Engineering/cereal-killer-01-insectoids/README.md) | `deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}` |
| **Reverse-Engineering** | [Cereal Killer 02 — The Nordics](./Reverse-Engineering/cereal-killer-02-nordics/README.md) | 150 | [writeup](./Reverse-Engineering/cereal-killer-02-nordics/README.md) | `deadface{N0rdics-R3ally-like-Frank3nb3rry-cuz-bro-is-tall-like-they-are!}` |
| **Web-Exploitation** | [Doctor Appointment](./Web-Exploitation/doctor-appointment/README.md) | 26 | [writeup](./Web-Exploitation/doctor-appointment/README.md) | `deadface{typical_and_classic}` |
| **Web-Exploitation** | [Spook-A-Thon](./Web-Exploitation/spook-a-thon/README.md) | 80 | [writeup](./Web-Exploitation/spook-a-thon/README.md) | `deadface{serve_me_up_a_flag}` |
| **Forensics** | [Night Veil University — Document Forensics](./Forensics/night-veil-nvu/README.md) | — | [writeup](./Forensics/night-veil-nvu/README.md) | `deadface{4659716e-3949-0eae-8231-7c2d09c1c108}` |
| **Forensics** | [SILENT EXFIL — The DNS Tunnel Beacon](./Forensics/silent-exfil/README.md) | 150 | [writeup](./Forensics/silent-exfil/README.md) | `CODEX{9E4FB7DE04_ALPHA_OVERRIDE_8821}` |
| **Forensics** | [Return to Sender](./Forensics/return-to-sender/README.md) | 80 | [writeup](./Forensics/return-to-sender/README.md) | `deadface{http_exfil_sneaky_business}` |
| **Forensics** | [SHADOW PROCESS — The In-Memory Intruder](./Forensics/shadow-process/README.md) | 350 | [analysis](./Forensics/shadow-process/README.md) | — |
| **OSINT** | [Ghost of Squares Past](./OSINT/ghost-of-squares-past/README.md) | 150 | [writeup](./OSINT/ghost-of-squares-past/README.md) | `deadface{1733-1742-1815-1733-gordon}` |
| **OSINT** | [The Mark](./OSINT/the-mark/README.md) | 350 | [writeup](./OSINT/the-mark/README.md) | `deadface{Santos_Staudenmaier}` |
| **Database** | [Prying Eyes](./Database/prying-eyes/README.md) | 55 | [writeup](./Database/prying-eyes/README.md) | `deadface{rborden}` |
| **Database** | [Ghost in the Registry](./Database/ghost-in-the-registry/README.md) | 240 | [writeup](./Database/ghost-in-the-registry/README.md) | `deadface{AH-0090001}` |
| **Database** | [After the Lock](./Database/after-the-lock/README.md) | 150 | [writeup](./Database/after-the-lock/README.md) | `deadface{VELT-0137}` |
| **Database** | [Upon the Throne](./Database/upon-the-throne/README.md) | 120 | [writeup](./Database/upon-the-throne/README.md) | `deadface{REVENANT-7}` |
| **Mobile-Security** | [Aurora Patient Portal series (4 challenges)](./Mobile-Security/aurora-patient-portal/README.md) | 100 | [writeup](./Mobile-Security/aurora-patient-portal/README.md) | `deadface{847261}` · `deadface{Candesartan}` · `deadface{gw-westlake-int-03}` · `deadface{mara-voss-1988-04-17}` |
| **Exploitation** | [Dangerous Capability](./Exploitation/dangerous-capability/README.md) | 300 | [writeup](./Exploitation/dangerous-capability/README.md) | `deadface{c4p_s3tu1d_cuts_d33p}` |
| **Database** | [The Attending](./Database/the-attending/README.md) | 40 | [writeup](./Database/the-attending/README.md) | `deadface{mreyes}` |
| **Database** | [Two Places at Once](./Database/two-places-at-once/README.md) | 450 | [writeup](./Database/two-places-at-once/README.md) | `deadface{kbhatt}` |
| **Database** | [Too Good to Be True](./Database/too-good-to-be-true/README.md) | 310 | [writeup](./Database/too-good-to-be-true/README.md) | `deadface{VELT-0148}` |
| **Forensics** | [Conduct Unbecoming](./Forensics/conduct-unbecoming/README.md) | 75 | [writeup](./Forensics/conduct-unbecoming/README.md) | `deadface{NVU-PRINT-17}` |
| **Forensics** | [Name to a Face](./Forensics/name-to-a-face/README.md) | 10 | [writeup](./Forensics/name-to-a-face/README.md) | `deadface{nkessler}` |
| **Forensics** | [Crown Jewels](./Forensics/crown-jewels/README.md) | 75 | [writeup](./Forensics/crown-jewels/README.md) | `deadface{annual_report.pdf}` |
| **Forensics** | [Public Offering](./Forensics/public-offering/README.md) | 75 | [writeup](./Forensics/public-offering/README.md) | `deadface{167.172.132.224_4815}` |
| **Reverse-Engineering** | [It But A Wee Bit Reverse](./Reverse-Engineering/it-but-a-wee-bit-reverse/README.md) | 15 | [writeup](./Reverse-Engineering/it-but-a-wee-bit-reverse/README.md) | `deadface{x0r_1s_not_encryption}` |
| **Reverse-Engineering** | [Cereal Killer 03](./Reverse-Engineering/cereal-killer-03/README.md) | 500 | [writeup](./Reverse-Engineering/cereal-killer-03/README.md) | `deadface{Liz4rds_L0ve_Krampuffs_for_some_odd_reason...}` |
| **Reverse-Engineering** | [Cereal Killer 04](./Reverse-Engineering/cereal-killer-04/README.md) | 250 | [writeup](./Reverse-Engineering/cereal-killer-04/README.md) | `deadface{Before_Consenting_to_Med_Exam_Ascertain_Destination_of_Finger!}` |
| **Exploitation** | [Foothold on Charon](./Exploitation/foothold-on-charon/README.md) | 10 | [writeup](./Exploitation/foothold-on-charon/README.md) | `deadface{f00th0ld_p4st_th3_v31l}` |
| **Web-Exploitation** | [Health Spark — Real Pain in the Prostate](./Web-Exploitation/health-spark/README.md) | — | [writeup](./Web-Exploitation/health-spark/README.md) | `deadface{hipaa_exposure_is_bad}` |

---

Happy Learning :)
