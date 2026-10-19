# Writeup — Crown Jewels

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 75 |
| **Artifact** | `demonne.pcap` (same capture as *Name to a Face*) |
| **Flag** | `deadface{annual_report.pdf}` |

> An attacker copied documents from an executive share. Which PDF did they read
> **first**?

---

## 1. Recon

Same De Monne Financial capture. This time the question is about file access
order on the executive share: `\\10.77.10.30\executive`. The flag is the full
filename of the first PDF the attacker actually *read*.

Key distinction: an SMB2 **QUERY_DIRECTORY** (listing) is not a read — only a
**CREATE** with read access followed by data transfer counts.

## 2. Mapping the share access

Walking the SMB2 session:

| Packet | Time | Operation | File |
|---|---|---|---|
| 6733 | 1790539645 | TREE_CONNECT | `\\10.77.10.30\executive` |
| 6899 | — | QUERY_DIRECTORY | Lists 4 files (listing only, **not** a read) |
| **7404** | **1790539651** | **CREATE (read/open)** | **`annual_report.pdf`** ← first |
| 8321 | 1790539659 | CREATE (read/open) | `executive_risk_register.xlsx` (not a PDF) |
| 9086 | 1790539667 | CREATE (read/open) | `board_transition_timeline.pdf` |
| 9738 | 1790539673 | CREATE (read/open) | `succession_vote_draft.pdf` |

The directory listing at packet 6899 shows all four files together, but a
listing is not a read. The first actual PDF open is `annual_report.pdf` at
packet 7404.

## 3. Flag

```
deadface{annual_report.pdf}
```

Submitted — platform returned **Correct**.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Locate TREE_CONNECT to `\\10.77.10.30\executive` | Share session identified |
| 2 | Order SMB2 CREATE (read/open) operations by timestamp | `annual_report.pdf` first (pkt 7404) |
| 3 | Exclude QUERY_DIRECTORY listings and non-PDF opens | Listing ≠ read; `.xlsx` ≠ PDF |
| 4 | Submit `deadface{annual_report.pdf}` | **Correct** |

## Takeaways

- **Listing is not reading.** The challenge hinges on distinguishing
  `QUERY_DIRECTORY` from `CREATE` — the first file *listed* is not the first
  file *read*.
- **Order by timestamp, filter by type.** Sorting opens chronologically and
  restricting to PDFs gives the answer mechanically.
