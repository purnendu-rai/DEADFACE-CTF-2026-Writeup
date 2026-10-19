# Writeup — Ghost in the Registry

| | |
|---|---|
| **Category** | Database / SQL |
| **Author** | @syyntax |
| **Points** | 240 |
| **Target** | `aurora.dctf.io` — MySQL web shell (SELECT-only) |
| **Flag** | `deadface{AH-0090001}` |

> Somewhere in the registry there's a fabricated patient — issued
> controlled-substance prescriptions but never seen for a single clinical
> encounter. Find at least one Schedule II prescription and zero encounters;
> report the MRN as `deadface{MRN}`.

---

## 1. Schema

Discovered via `DESCRIBE`/`SHOW`:

- `aurora_ehr.patients` — `patient_id` PK, `mrn` unique, `is_vip`
- `aurora_ehr.encounters` — `encounter_id` PK, `patient_id`
- `aurora_pharmacy.prescriptions` — `rx_id` PK, `patient_id`, `prescriber_id`, `drug_id`, …
- `aurora_pharmacy.drugs` — `drug_id` PK, `dea_schedule`

Schedule II is marked by `dea_schedule='II'` (distinct values in the column:
`II`, `III`, `IV`, `V`, `N`).

## 2. The query

Join the enterprise `patient_id` across `aurora_ehr` and `aurora_pharmacy`,
keep patients with ≥1 Schedule II prescription and 0 encounters:

```sql
SELECT p.mrn, COUNT(rx.rx_id) AS sched2
FROM aurora_ehr.patients p
JOIN aurora_pharmacy.prescriptions rx ON rx.patient_id = p.patient_id
JOIN aurora_pharmacy.drugs d ON d.drug_id = rx.drug_id
LEFT JOIN aurora_ehr.encounters e ON e.patient_id = p.patient_id
WHERE d.dea_schedule = 'II'
GROUP BY p.patient_id
HAVING COUNT(e.encounter_id) = 0;
```

Exactly **one** patient matches — MRN **AH-0090001** (patient_id 1060), with
3 Schedule II prescriptions:

| Rx | Drug | Written |
|---|---|---|
| 40155 | Hydrocodone/APAP | 2025-11-20 |
| 40157 | Hydrocodone/APAP | 2025-10-19 |
| 40156 | Amphetamine Salts | 2025-12-15 |

Zero encounters.

```
deadface{AH-0090001}
```

All queries read-only (`SELECT`/`DESCRIBE`/`SHOW`).

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Schema discovery | patients ↔ encounters ↔ prescriptions ↔ drugs |
| 2 | `dea_schedule='II'` filter + LEFT JOIN encounters | exactly 1 match: **AH-0090001** |
| 3 | Verify | 3 Schedule II scripts, 0 encounters |

## Takeaways

- **Cross-database joins are the whole challenge.** The EHR and pharmacy
  live in different schemas joined on the enterprise `patient_id` — map the
  keys before querying.
- **LEFT JOIN + HAVING COUNT = 0** is the canonical "present in A, absent in
  B" pattern.
- **Uniqueness is the confirmation.** Exactly one patient matching both
  conditions means no ambiguity in the answer.

## Tools Used

- Aurora MySQL web shell (SELECT-only, via browser)
