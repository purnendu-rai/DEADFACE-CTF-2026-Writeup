# Writeup — Conduct Unbecoming

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 75 |
| **Artifact** | `Doc_20261017_0915.jpg` (SHA256 `6e7809f4…44f1ba`) |
| **Flag** | `deadface{NVU-PRINT-17}` |

> A scanned document hides printer tracking dots. Recover the printer's
> serial identifier.

---

## 1. Recon

The challenge provides a scanned document image. EXIF/XMP metadata carries no
asset identifier, and LSB steganalysis of the image data yields no plaintext.
The document itself is clean — the secret is in how it was *printed*.

## 2. Printer yellow dots

Color laser printers embed nearly-invisible **yellow tracking dots** (Machine
Identification Code) encoding the printer's serial number and timestamp. They
hide in white margins — exactly where a casual glance sees "empty" paper.

Scanning the bottom-right white margin reveals a dot grid:

- Position: x ≈ 2331–2416, y ≈ 3026–3170
- Grid: **8 columns × 13 rows**, 12 px spacing, each dot 2×2 px
- Detection: yellowness metric `(R+G)/2 − B > 7`

## 3. Decoding

Reading the grid row-major, MSB-first, with 1 = dot present:

- **Row 0:** `11111111` = `0xFF` — sync/header row
- **Rows 1–12:** decode to ASCII `NVU-PRINT-17`

```
N = 0x4E, V = 0x56, U = 0x55, - = 0x2D,
P = 0x50, R = 0x52, I = 0x49, N = 0x4E,
T = 0x54, - = 0x2D, 1 = 0x31, 7 = 0x37
```

## 4. Flag

```
deadface{NVU-PRINT-17}
```

Submitted — platform returned **Correct**.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | EXIF/XMP and LSB checks | No embedded data |
| 2 | Yellow-dot scan of white margins (`(R+G)/2 − B > 7`) | 8×13 dot grid located |
| 3 | Row-major MSB-first decode | Sync `0xFF` + `NVU-PRINT-17` |
| 4 | Submit `deadface{NVU-PRINT-17}` | **Correct** |

## Takeaways

- **Printer dots are a classic.** Yellow tracking dots (MIC) survive scanning
  and photocopying — always check blank margins of a "clean" document image.
- **The sync row orients the decode.** The all-ones first row confirms bit order
  and grid alignment before trusting the payload rows.
