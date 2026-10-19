# Writeup — After the Lock

| | |
|---|---|
| **Category** | Database / SQL |
| **Author** | @syyntax |
| **Points** | 150 |
| **Target** | `aurora.dctf.io` — MySQL web shell (SELECT-only) |
| **Flag** | `deadface{VELT-0137}` |

> The Veltrexa trial data was locked — but someone modified a row afterward.
> Find the tampered record.

---

## 1. The lock date

The Veltrexa study (study_id 1, `aurora_trials.studies`) has:

```
data_lock_date = 2025-12-01
```

Nothing in the trial results should have a modification timestamp after that.

## 2. The tampered row

Exactly **one** `primary_efficacy_delta` row was modified after the lock:

```sql
SELECT result_id, subject_code, modified_at, modified_by
FROM primary_efficacy_delta
WHERE modified_at > '2025-12-01'
  AND study_id = 1;
```

| result_id | subject_code | modified_at | modified_by |
|---|---|---|---|
| 200058 | **VELT-0137** | 2025-12-06 23:14:00 | mreyes |

Week 24, value 6.00, originally recorded 2025-10-15 by `achang` — then
quietly edited five days after the data lock.

```
deadface{VELT-0137}
```

Read-only queries only.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Read `aurora_trials.studies` | `data_lock_date` = 2025-12-01 |
| 2 | `WHERE modified_at > data_lock_date` | exactly 1 row: **VELT-0137** |

## Takeaways

- **Audit columns are the challenge.** `modified_at`/`modified_by` exist for
  exactly this — compare them against the business rule (the lock date).
- **One row is the answer.** Data-tampering challenges are designed so the
  anomaly is unique; if your query returns dozens of rows, your filter is
  wrong.

## Tools Used

- Aurora MySQL web shell (SELECT-only, via browser)
