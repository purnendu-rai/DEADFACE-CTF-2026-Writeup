# Writeup — Prying Eyes

| | |
|---|---|
| **Category** | Database / SQL |
| **Author** | @syyntax |
| **Points** | 55 |
| **Target** | `https://aurora.dctf.io/` — MySQL web shell (SELECT-only) |
| **Flag** | `deadface{rborden}` |

> Someone's been prying into a VIP patient's chart. Find out who opened it the
> most.

---

## 1. Finding the VIP

`aurora_ehr.patients` has an `is_vip` flag and an alias column. Exactly one
row has `is_vip=1`:

- MRN **AH-0044719** — Gregory Halloran, admitted under the alias
  **'George Harmon'** (the "Admitted Under Another Name" story is where the
  VIP's MRN comes from).

## 2. Who opened the chart?

`aurora_audit.access_log` (`user_id`, `patient_mrn`, `action`, `accessed_at`)
joined against `aurora_audit.users`:

```sql
SELECT u.username, COUNT(*)
FROM access_log a
JOIN users u ON a.user_id = u.user_id
WHERE a.patient_mrn = 'AH-0044719'
GROUP BY u.username
ORDER BY COUNT(*) DESC
LIMIT 5;
```

| Username | Chart opens |
|---|---|
| **rborden** | **47** |
| mreyes_ehr | 5 |
| ebaker7 | 2 |
| manderson12 | 2 |
| cyoung18 | 1 |

The top chart-opener — by a wide margin — is `rborden`:

```
deadface{rborden}
```

**Tool note:** `aurora.dctf.io:3306` answers externally but isn't a real
MySQL handshake (client packet-sequence errors), so the provided web terminal
was used via the browser. All queries were read-only.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | `WHERE is_vip=1` in `aurora_ehr.patients` | VIP = AH-0044719 (Gregory Halloran / 'George Harmon') |
| 2 | JOIN `access_log` → `users`, GROUP BY, ORDER BY COUNT DESC | **rborden**, 47 opens (next: 5) |

## Takeaways

- **Start from the victim, then follow the access log.** The challenge is a
  two-hop query: identify the VIP first, then aggregate who touched that MRN.
- **A 47-to-5 margin is the answer.** When the distribution has a clear
  outlier, that's the signal — no tie-breaking needed.
- **Read-only shells are enough.** `SELECT`/`DESCRIBE`/`SHOW` covered the
  whole solve; nothing needed to be written.

## Tools Used

- Aurora MySQL web shell (SELECT-only, via browser)
