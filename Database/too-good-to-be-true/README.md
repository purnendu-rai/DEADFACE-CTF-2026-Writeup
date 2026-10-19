# Writeup — Too Good to Be True

| | |
|---|---|
| **Category** | Database |
| **Points** | 310 |
| **Target** | MySQL at `aurora.dctf.io`, database `aurora_trials` |
| **Flag** | `deadface{VELT-0148}` |

> A clinical trial's efficacy numbers look too good to be true. Find the subject
> whose results don't belong.

---

## 1. Recon

The `aurora_trials` database contains trial data for study **AUR-VLT-3001**
(Veltrexa Phase III): 60 subjects, with the primary endpoint
`primary_efficacy_delta` measured at Week 24. The flag is the `subject_code`
of the anomalous subject.

## 2. Establishing the baseline

Aggregate the Week 24 primary endpoint across all 60 subjects:

- Mean ≈ **2.965**
- Standard deviation ≈ **2.83**
- Mean + 3σ threshold ≈ **11.4**

Any legitimate reading should fall well under ~11.4.

## 3. The outlier

Exactly one subject blows past the threshold:

| subject_code | primary_efficacy_delta (Week 24) | z-score |
|---|---|---|
| **VELT-0148** | **22.00** | ≈ 6.8 |
| runner-up | 6.00 | ≈ 1.1 |

VELT-0148 sits nearly **7 standard deviations** above the mean — roughly double
the 3σ cutoff — while the next-highest subject is comfortably inside one sigma.
There is no ambiguity about which subject is anomalous.

## 4. Flag

```
deadface{VELT-0148}
```

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Scope study AUR-VLT-3001, 60 subjects, Week 24 endpoint | Baseline population |
| 2 | Compute mean (2.965) and σ (2.83) → 3σ cutoff ≈ 11.4 | Anomaly threshold |
| 3 | Find subjects above cutoff | Exactly one: VELT-0148 at 22.00 (z ≈ 6.8) |
| 4 | Submit `deadface{VELT-0148}` | **Solved** |

## Takeaways

- **Let statistics do the accusing.** A 6.8σ outlier in a 60-subject trial is
  not a judgment call — the z-score makes the anomaly self-evident.
- **Check the runner-up.** The gap between the outlier (22.00) and the next
  value (6.00) confirms this is a single corrupted/fabricated record, not a
  heavy-tailed distribution.
