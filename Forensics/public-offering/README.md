# Writeup — Public Offering

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 75 |
| **Artifact** | `demonne.pcap` (same capture as *Name to a Face*) |
| **Flag** | `deadface{167.172.132.224_4815}` |

> Kali sent a ZIP archive outside the corporate network. Which public IPv4 and
> TCP destination port received it?

---

## 1. Recon

Same De Monne Financial capture. The attacker host (`10.77.10.66`, the Kali
box that poisoned name resolution in *Name to a Face*) exfiltrated a ZIP
archive to an external address. The flag format is `deadface{IP_PORT}`.

## 2. Finding the exfiltration flow

Scanning for connections from the internal network to **public** IPv4 space
with sustained data transfer reveals one standout flow:

- **Source:** `10.77.10.66:34174` (Kali)
- **Destination:** `167.172.132.224:4815` (public)
- **Protocol:** raw TCP, **164 packets** — a bulk transfer, not a handshake

## 3. Confirming the ZIP

The flow's payload contains `PK\x03\x04` local file headers — the ZIP magic —
with entries for the exact documents taken from the executive share:

- `annual_report.pdf`
- `board_transition_timeline.pdf`
- `executive_risk_register.xlsx`
- `succession_vote_draft.pdf`

This ties the exfiltration directly to the *Crown Jewels* theft: the stolen
executive-share documents, zipped, sent to `167.172.132.224` on TCP port 4815.

## 4. Flag

```
deadface{167.172.132.224_4815}
```

Submitted — platform returned **Correct** (1/25 attempts).

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Hunt internal → public flows with bulk transfer | `10.77.10.66:34174 → 167.172.132.224:4815`, 164 pkts |
| 2 | Inspect payload for `PK\x03\x04` ZIP headers | Archive of the 4 stolen executive docs |
| 3 | Submit `deadface{167.172.132.224_4815}` | **Correct** |

## Takeaways

- **Correlate across challenges.** The ZIP's file listing matches the *Crown
  Jewels* document set — confirming this flow is the exfiltration, not noise.
- **Raw TCP on an odd port is the tell.** No HTTP/DNS wrapping — just a direct
  socket to a public IP, the simplest exfil there is.
