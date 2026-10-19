# Writeup — Ghost of Squares Past

| | |
|---|---|
| **Category** | OSINT |
| **Points** | 150 |
| **Artifacts** | Five challenge photos (historic squares) — no downloads |
| **Flag** | `deadface{1733-1742-1815-1733-gordon}` |

> Five historic Savannah, Georgia squares are arranged as a compass —
> North, East, South, West, and center. Flag format:
> `deadface{YEAR-YEAR-YEAR-YEAR-SURNAME}` — the year each square was laid out,
> in clue order, plus the surname engraved on the center monument.

---

## 1. The compass

Five photos, each a Savannah square, arranged N / E / S / W / center. The job:
identify each square, find the year it was laid out, and read the surname off
the center monument.

## 2. The five squares

**P1 — North.** Riddle: "first of them all"; a Rhode Island Quaker
(Nathanael Greene — born to a Rhode Island Quaker family, buried beneath his
monument there); also the banking square → **Johnson Square → 1733**.

**P2 — East.** Photo match → **Oglethorpe Square** (Abercorn & President,
east of Wright Square) → **1742**.

**P3 — South.** Riddle: a battle of a northern frontier (the Battle of
Chippawa, War of 1812); the Forrest Gump feather/bench; the founder Oglethorpe
statue; "newest of all" among the five → **Chippewa Square → 1815**
(Wikipedia + historical marker; one tour site claimed 1812 — treated as an
outlier).

**P4 — West.** Photo match → **Telfair Square** (Barnard & President, west of
Wright; the white Telfair Academy building is visible in the photo) → **1733**.

**P5 — Center.** Compass geometry places it at **Wright Square**
(Bull & President). Its William Washington Gordon Monument has the lone name
**"GORDON"** carved in block letters on the north-facing panel (per
Savannah's official "Monuments and Fountains" document) → surname **gordon**.

## 3. The flag

In clue order:

```
deadface{1733-1742-1815-1733-gordon}
```

---

## Summary

| Photo | Square | Year | How identified |
|---|---|---|---|
| P1 (North) | Johnson Square | 1733 | "first of them all", RI Quaker, banking square |
| P2 (East) | Oglethorpe Square | 1742 | photo match (Abercorn & President) |
| P3 (South) | Chippewa Square | 1815 | Chippawa battle, Gump bench, Oglethorpe statue |
| P4 (West) | Telfair Square | 1733 | photo match (Telfair Academy) |
| P5 (Center) | Wright Square | — | compass geometry; Gordon Monument → `gordon` |

## Takeaways

- **OSINT riddles are cross-checks, not single clues.** Each square had 2–3
  independent identifiers (photo, riddle, geography) — never trust one.
- **Treat outlier sources as outliers.** One tour site said 1812 for
  Chippewa; Wikipedia + the historical marker said 1815. Majority + primary
  sources win.
- **Read the monument, not the square's name.** The surname came from the
  inscription on the Gordon Monument, not from anything named "Wright".

## Tools Used

- Web research (Wikipedia, historical markers, City of Savannah documents)
- Photo comparison
