# Writeup — Upon the Throne

| | |
|---|---|
| **Category** | Database / SQL |
| **Author** | @syyntax |
| **Points** | 120 |
| **Target** | `aurora.dctf.io` — MySQL web shell (SELECT-only) |
| **Flag** | `deadface{REVENANT-7}` |

> Pull the Aurora breach threads together. Find the single DEADFACE handler
> codename that appears in BOTH the `handler_ref` (compromised-account) column
> of the audit-system users table AND the EHR providers table — the one
> operative running an insider on each side. Flag format: `deadface{CODENAME}`.

---

## 1. Where the column lives

`handler_ref` exists in exactly two tables:

- `aurora_audit.users`
- `aurora_ehr.providers`

Each side has **16 distinct non-null** codenames.

## 2. The intersection

```sql
SELECT a.handler_ref
FROM aurora_audit.users a
INNER JOIN aurora_ehr.providers p
  ON p.handler_ref = a.handler_ref;
```

The INNER JOIN returns **exactly 1 row**:

```
REVENANT-7
```

One operative, running an insider on each side.

```
deadface{REVENANT-7}
```

Read-only queries only.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Locate `handler_ref` | `aurora_audit.users` + `aurora_ehr.providers` |
| 2 | INNER JOIN on the codename | exactly 1 intersection: **REVENANT-7** |

## Takeaways

- **INNER JOIN is the question.** "Appears in both" is literally a join —
  don't overthink it.
- **Check both sides' cardinality first.** 16 distinct codenames per side
  with a single-row intersection means the answer is unambiguous.

## Tools Used

- Aurora MySQL web shell (SELECT-only, via browser)
