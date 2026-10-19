# Writeup — SILENT EXFIL: The DNS Tunnel Beacon

| | |
|---|---|
| **Category** | Digital Forensics |
| **Points** | 150 (Easy track, 0 solves at the time) |
| **Artifact** | `dns_exfil_traffic.pcapng` |
| **Flag** | `CODEX{9E4FB7DE04_ALPHA_OVERRIDE_8821}` |

> A beacon is tunneling data out over DNS. Reassemble the tunnel and read the
> exfiltrated message.

---

## 1. The tunnel

The capture holds **16 sequential DNS queries** to `ns1.shadow-sector.org`
(session `BTR9042`, sequence `001`–`016`). Each query's leftmost label is a
Base32 chunk of the exfiltrated payload.

## 2. Reassembly

Concatenate the Base32 subdomains **in timestamp order** → a 434-character
stream. Whole-stream `b32decode` (with 6 `=` pad characters appended) yields a
271-byte fake critical-infrastructure incident log containing the flag:

```python
import base64
chunks = [...]  # 16 subdomains, timestamp order
stream = "".join(chunks)
stream += "=" * (-len(stream) % 8)
data = base64.b32decode(stream)   # 271 bytes
# ... contains CODEX{9E4FB7DE04_ALPHA_OVERRIDE_8821}
```

```
CODEX{9E4FB7DE04_ALPHA_OVERRIDE_8821}
```

## 3. Why I'm sure (the dispute)

The platform said "incorrect". I re-verified twice:

1. **Round-trip re-encode** of the decoded log matches the chunk stream
   byte-for-byte; the flag bytes are clean ASCII.
2. **Exhaustive sweep** — the string `shadow` appears in exactly 32 packets
   (16 queries + 16 responses); queries carry no EDNS/additional data;
   responses are plain A records to `10.254.1.1`; the Modbus (port 502) and
   syslog traffic is background noise. **No other data channel exists in the
   capture.**

The flag is forensically definitive. The rejection is most likely
submission-side (typo, wrong challenge/track, login/session).

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Filter DNS to `ns1.shadow-sector.org` | 16 sequential queries, session BTR9042 |
| 2 | Concat Base32 labels in timestamp order | 434-char stream |
| 3 | `b32decode` (+ 6 pad) | 271-byte incident log with the flag |
| 4 | Round-trip + full-packet sweep | no other channel — data is definitive |

## Takeaways

- **Order matters in tunnel reassembly.** Chunks must be concatenated in
  timestamp order, not packet order — and the whole stream decoded at once,
  not chunk by chunk.
- **Pad your Base32.** A stream that isn't a multiple of 8 chars needs `=`
  padding or the decode fails.
- **When forensics and the platform disagree, verify exhaustively, then
  suspect the submission.** A byte-exact round-trip plus a full-capture
  sweep is the standard of proof — after that, it's a typo or a wrong form.

## Tools Used

- `scapy` (PCAP parsing; tshark/tcpdump weren't available)
- Python `base64` (b32decode)
