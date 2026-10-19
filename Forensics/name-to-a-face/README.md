# Writeup — Name to a Face

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 10 |
| **Artifact** | `demonne.pcap` (7.5 MB, SHA256 `a7176232…72d41a8ddb3`) |
| **Flag** | `deadface{nkessler}` |

> Someone on the corporate network authenticated to a file server using a
> legitimate employee account. Put a name to that account.

---

## 1. Recon

The packet capture comes from De Monne Financial's corporate network. Background:
a workstation on the local network asked to resolve a file-server name
(`dmf-field-01`), and a suspicious host (`10.77.10.66`) on the corporate IPv4
range answered with poisoned LLMNR/mDNS/NBNS responses — prompting the victim
to attempt SMB authentication against the attacker.

## 2. Extracting the NTLM authentication

The poisoned name resolution triggers an SMB session carrying an **NTLM Type 3
(AUTHENTICATE)** message. Parsing the NTLMSSP structures in the capture, the
username field (UTF-16LE) contains exactly one distinct value across all
authentication attempts:

- **20 occurrences** of `nkessler`
- No other username appears in any Type 3 message

## 3. Flag

The compromised employee account is `nkessler`:

```
deadface{nkessler}
```

Submitted — platform returned **Correct** (1/100 attempts).

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Identify poisoned LLMNR/mDNS/NBNS answers for `dmf-field-01` from `10.77.10.66` | Attacker-forced SMB auth |
| 2 | Parse NTLM Type 3 AUTH messages, extract UTF-16LE usernames | Only `nkessler` (20 hits) |
| 3 | Submit `deadface{nkessler}` | **Correct** |

## Takeaways

- **Follow the auth, not the noise.** A 7.5 MB pcap reduces to one question:
  whose credentials were offered to the rogue server?
- **Count distinct values, not packets.** Twenty hits on a single username with
  zero alternatives means the answer is certain, not merely likely.
