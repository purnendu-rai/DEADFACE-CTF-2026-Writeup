# Writeup — Return to Sender

| | |
|---|---|
| **Category** | Forensics (network) |
| **Points** | 80 |
| **Artifact** | `PCAP03.pcap` |
| **Flag** | `deadface{http_exfil_sneaky_business}` |

> Something in this capture is being sent back where it came from. Find the
> exfiltration hidden inside normal web traffic.

---

## 1. The flow

Amid normal web traffic (the cover), one flow stands out: compromised host
**`10.20.30.40`** exfiltrating data to **`45.66.77.88`**
(`telemetry.deadface-cc.net`) in **5 chunked base64 POSTs**.

## 2. The chunks — and the missing tail

Decoding the 5 POST request bodies gives only a *prefix* of the flag:

```
deadface{http_exfil_sneaky_
```

The closing fragment is smuggled in the **`X-Session-Id` header** carried on
every exfil POST — base64 `YnVzaW5lc3N9` decodes to:

```
business}
```

## 3. Assembly

POST-body chunks + session-header tail:

```
deadface{http_exfil_sneaky_business}
```

**Note:** if the platform rejects it, the header segment is the part to
re-check first — it's the only piece that didn't come from the POST bodies.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Isolate the anomalous flow | `10.20.30.40` → `45.66.77.88`, 5 base64 POSTs |
| 2 | Decode POST bodies | flag prefix `deadface{http_exfil_sneaky_` |
| 3 | Decode `X-Session-Id` header | tail `business}` |
| 4 | Assemble | **`deadface{http_exfil_sneaky_business}`** |

## Takeaways

- **Exfil hides in plain HTTP.** Chunked POSTs to a plausible telemetry
  domain blend into background web traffic — filter by behavior, not by
  protocol.
- **Headers are a channel too.** When the body decode ends mid-flag, check
  the request headers — `X-Session-Id` carried the tail.
- **Always verify the join.** A flag assembled from two sources should be
  re-checked against both if the platform rejects it.

## Tools Used

- `scapy` (PCAP parsing; tshark/tcpdump weren't installed)
- Base64 decoding (Python)
