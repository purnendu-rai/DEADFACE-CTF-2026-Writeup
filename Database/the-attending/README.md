# Writeup — The Attending

| | |
|---|---|
| **Category** | Database |
| **Points** | 40 |
| **Target** | MySQL at `aurora.dctf.io`, database `aurora_ehr` |
| **Flag** | `deadface{mreyes}` |

> A VIP patient was admitted as an inpatient exactly once during November 2025.
> Find who attended them.

---

## 1. Recon

The challenge exposes a MySQL instance (`aurora.dctf.io`) hosting the `aurora_ehr`
electronic health record database. The task: identify a VIP patient with exactly
one inpatient admission in November 2025, then determine their attending provider.

Relevant tables: `patients`, `encounters` (admissions), `providers`.

## 2. Finding the VIP admission

Filter `encounters` for inpatient admissions with an admit date in November 2025,
then group by patient and keep only those with exactly one such admission:

```sql
SELECT patient_id, COUNT(*) AS n
FROM encounters
WHERE encounter_type = 'inpatient'
  AND admit_date BETWEEN '2025-11-01' AND '2025-11-30'
GROUP BY patient_id
HAVING n = 1;
```

Exactly one patient matches: **patient 1007**.

## 3. Identifying the patient

```sql
SELECT * FROM patients WHERE patient_id = 1007;
```

Patient 1007 is **Gregory Halloran**, admitted under the alias **George Harmon**.
The qualifying encounter is **encounter 5017**, admitted **2025-11-14**.

## 4. The attending provider

The encounter row carries `attending_provider_id = 5`:

```sql
SELECT * FROM providers WHERE provider_id = 5;
```

Provider 5 is **Marcus Reyes**, whose login username is `mreyes`.

## 5. Flag

The challenge asks for the attending provider's username:

```
deadface{mreyes}
```

Submitted — platform returned **Correct**.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Filter inpatient encounters to Nov 2025, `HAVING COUNT(*) = 1` | Patient 1007 |
| 2 | Patient lookup | Gregory Halloran (alias George Harmon), encounter 5017 |
| 3 | Encounter → `attending_provider_id = 5` | Marcus Reyes, username `mreyes` |
| 4 | Submit `deadface{mreyes}` | **Correct** |

## Takeaways

- **Aggregate filters beat eyeballing.** `GROUP BY … HAVING COUNT(*) = 1` isolates
  the "exactly once" condition in a single query instead of manual sifting.
- **Aliases matter.** The patient was admitted under a different name — joining on
  IDs rather than names avoids missing the record.
