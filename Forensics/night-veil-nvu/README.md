# Writeup — Night Veil University: Document Forensics

| | |
|---|---|
| **Category** | Forensics (document/image metadata) |
| **Points** | — (not recorded) |
| **Artifact** | Recovered JPEG — "STUDENT CONDUCT NOTICE" for Cornelius Kellerman, NVU |
| **Flag** | `deadface{4659716e-3949-0eae-8231-7c2d09c1c108}` |

> What asset-id was used to create the document? The answer is hidden in the
> image's metadata.

---

## 1. The document

A recovered JPEG shows a "STUDENT CONDUCT NOTICE" for Cornelius Kellerman
(Night Veil University). The question: which asset-id created it?

## 2. The planted 8BIM resource

The JPEG's **APP13 Photoshop IRB** segment holds a planted, non-standard 8BIM
resource — id `0x0425`, 16 bytes:

```
4659716e39490eae82317c2d09c1c108
```

Supporting metadata: IPTC `ObjectName` = "NVU Conduct Notice"; XMP says the
document was produced by **Affinity Photo 2 2.6.5** on 2026-06-08. No
single-byte XOR decoding — the value is the raw id.

First submission:

```
deadface{4659716e39490eae82317c2d09c1c108}
```

**The platform rejected it.**

## 3. The correction

A full re-sweep of the JPEG (strings, all APPn segments, XMP, ICC, IPTC)
found **no other asset-id candidate** — the `0x0425` value is the only ID-like
artifact in the file. The data is right; the joining format was wrong. The
same 16 bytes in UUID dashed format:

```
deadface{4659716e-3949-0eae-8231-7c2d09c1c108}
```

Submission status of the corrected variant: **unknown**.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | JPEG APP13 IRB walk | non-standard 8BIM `0x0425`, 16 bytes |
| 2 | Submit plain hex | ❌ rejected |
| 3 | Re-sweep (APPn, XMP, ICC, IPTC) | no other candidate — format issue, not data |
| 4 | UUID-dashed variant | submitted |

## Takeaways

- **APP13 IRB is a classic hiding spot.** Photoshop resource blocks are
  rarely inspected — walk them byte by byte.
- **When the data is definitive but rejected, suspect the format.** A full
  re-sweep ruling out every other candidate is what justifies reformatting
  the same value instead of hunting for a new one.
- **16 raw bytes → try UUID.** Asset-ids are UUIDs far more often than raw
  hex strings.

## Tools Used

- JPEG/APP13/EXIF/XMP metadata analysis (Python)
- `strings`, segment walkers
