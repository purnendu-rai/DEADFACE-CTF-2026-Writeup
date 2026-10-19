# Writeup — Two Places at Once

| | |
|---|---|
| **Category** | Database |
| **Points** | 450 |
| **Target** | MySQL at `aurora.dctf.io`, database `aurora_audit` |
| **Flag** | `deadface{kbhatt}` |

> Someone's credentials were used from two workstations at the exact same time.
> Find the compromised account.

---

## 1. Recon

The `aurora_audit` database holds an `access_log` table recording every
authentication event: `user_id`, `accessed_at` timestamp, and `workstation`.
A single human cannot physically log in from two different workstations at the
identical timestamp — that pattern indicates credential sharing or compromise.

## 2. Detecting impossible logins

Group the log by `(user_id, accessed_at)` and keep groups that touched two or
more distinct workstations:

```sql
SELECT user_id, accessed_at, COUNT(DISTINCT workstation) AS stations
FROM access_log
GROUP BY user_id, accessed_at
HAVING COUNT(DISTINCT workstation) >= 2;
```

## 3. Exactly one offender

The query returns rows for **exactly one user**: `kbhatt`, across **8 events**
spanning 2025-11-04 through 2025-11-27. Every event is an identical-timestamp
pair on two distinct workstations. Every other user in the log has zero such
events — the signal is unambiguous, not a threshold judgment call.

## 4. Flag

```
deadface{kbhatt}
```

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Audit `access_log` schema (`user_id`, `accessed_at`, `workstation`) | Login event model |
| 2 | `GROUP BY (user_id, accessed_at) HAVING COUNT(DISTINCT workstation) >= 2` | Impossible-travel pattern |
| 3 | Corroborate: only one user, 8 events, all other users clean | `kbhatt` |
| 4 | Submit `deadface{kbhatt}` | **Solved** |

## Takeaways

- **Impossible-travel is a one-query detection.** Grouping by the exact timestamp
  (not a time window) makes the condition strict: no legitimate user can be in
  two places in the same second.
- **Uniqueness of the result is the confirmation.** One user with 8 hits and
  everyone else at zero leaves no room for false positives.
