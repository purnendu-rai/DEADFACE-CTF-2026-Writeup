# Writeup — Aurora Patient Portal series

| | |
|---|---|
| **Category** | Mobile Security (iOS app forensics) |
| **Points** | 10 + 50 + 15 + 25 = 100 |
| **Artifact** | `AuroraPatientPortal.ipa.zip` → IPA → `Payload/aurora-patient-portal.app` (Mach-O arm64 "Aurora Health Mobile", `Info.plist`, `aurora_meds.sqlite`, `aurora_chest_xray.jpg`, `Assets.car`) |

Four challenges, one app.

---

## Challenge 1 — Down the Rabbit Hole (10 pts)

**Flag:** `deadface{847261}`

The Patient ID is hidden in the chest-xray JPEG's **XMP metadata**:

```xml
<dc:identifier>PID-847261</dc:identifier>
```

Corroboration: in `aurora_meds.sqlite` the patient IDs form a +7319 lattice
starting at 101001 — and `847261` is the only off-pattern row. The odd one
out is the answer.

```
deadface{847261}
```

## Challenge 2 — SQL Skillz (50 pts)

**Flag:** `deadface{Candesartan}`

Query `aurora_meds.sqlite` for the PID from Challenge 1:

```sql
SELECT medication_1, medication_2 FROM meds WHERE pid = 847261;
-- medication_1 = 'Ibuprofen 400 mg', medication_2 = 'Candesartan 16 mg'
```

The medication *other than* Ibuprofen:

```
deadface{Candesartan}
```

## Challenge 3 — Back to Basics (15 pts)

**Flag:** `deadface{gw-westlake-int-03}`

Binary `strings` on the Mach-O contain the hidden diagnostic page's gateway
server name:

```
gw-westlake-int-03
```

It matches the patient's Westlake clinic — the intended server for the
diagnostic page:

```
deadface{gw-westlake-int-03}
```

## Challenge 4 — Info Leaks (25 pts)

**Flag:** `deadface{mara-voss-1988-04-17}`

`Info.plist` ships a baked-in patient bootstrap — the last-used patient for
this app version, base64-encoded JSON under `AuroraPatientBootstrap`:

```json
{"patientName":"Mara Voss","dateOfBirth":"1988-04-17","clinic":"Westlake"}
```

Decoded → name + DOB →

```
deadface{mara-voss-1988-04-17}
```

---

## Summary

| Challenge | Technique | Flag |
|---|---|---|
| Down the Rabbit Hole | XMP `<dc:identifier>` in xray JPEG | `deadface{847261}` |
| SQL Skillz | SQLite query on PID 847261 | `deadface{Candesartan}` |
| Back to Basics | `strings` on Mach-O binary | `deadface{gw-westlake-int-03}` |
| Info Leaks | base64 bootstrap in `Info.plist` | `deadface{mara-voss-1988-04-17}` |

## Takeaways

- **An IPA is a zip.** Unpack it and every asset — plists, SQLite DBs,
  images — is forensic surface.
- **Metadata is data.** The PID wasn't in the image; it was in the image's
  XMP. Always check EXIF/XMP/IPTC before pixel-steg.
- **Plists leak.** Baked-in bootstrap configs with PII are a classic mobile
  finding — and a classic challenge plant.
- **Series chain.** Each challenge's answer feeds the next (PID → meds), so
  solve them in order.

## Tools Used

- IPA extraction (`unzip`)
- Plist parsing, base64 decoding
- SQLite queries
- JPEG XMP metadata inspection
- `strings` (Mach-O)
