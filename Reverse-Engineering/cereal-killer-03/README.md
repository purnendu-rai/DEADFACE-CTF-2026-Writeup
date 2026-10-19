# Writeup — Cereal Killer 03

| | |
|---|---|
| **Category** | Reverse Engineering |
| **Points** | 500 |
| **Author** | @TheZeal0t |
| **Artifact** | `re04-win-x64.exe` (PE32+ x64, 146 KB) |
| **Flag** | `deadface{Liz4rds_L0ve_Krampuffs_for_some_odd_reason...}` |

> The lizards have a two-stage check. Get past both.

---

## 1. Recon

A 146 KB PE32+ x64 binary. The first stage is a password check against an MD5
digest:

```
MD5(password) == 0d107d09f5bbe40cade3de5c71e9e9b7
```

## 2. Cracking the password

The target hash is a plain MD5 of a weak password — a quick lookup/wordlist
attack yields:

```
password = letmein
```

Verified: `MD5("letmein") == 0d107d09f5bbe40cade3de5c71e9e9b7`.

## 3. The gotcha: a second stage

Entering the correct password does **not** print the flag directly. Instead it
triggers a second stage (at `0x140001ad0`): a 55-byte blob at `0x1400185a0`
is decrypted with **xorshift64**:

```
x ^= x << 13;  x ^= x >> 7;  x ^= x << 17;
```

The PRNG is seeded with:

```
seed = bswap64(first_8_bytes(MD5(password))) ^ 0xd1cefa11badc0de5
     = 0xdd2a41e4b3a11de8
```

Each step's low byte is XORed into the blob. The decrypted output is validated
by `MD5(decrypted) == 5d02e679a7bfae73f997e7d4ddfa8c12` — an integrity check
that confirms the decryption before the flag is trusted.

## 4. Flag

Reimplementing both stages in Python (MD5 checks verified programmatically):

```
deadface{Liz4rds_L0ve_Krampuffs_for_some_odd_reason...}
```

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Extract MD5 target from binary | `0d107d09…e9e9b7` |
| 2 | Wordlist/lookup vs MD5 | `letmein` |
| 3 | Reverse xorshift64 second stage, seed `0xdd2a41e4b3a11de8` | 55-byte blob decrypted |
| 4 | Verify `MD5(decrypted) == 5d02e679…0c12` | Flag confirmed |

## Takeaways

- **The obvious password is a trapdoor, not the prize.** Correct input unlocks
  a *second* crypto stage — always trace what happens *after* "Access granted".
- **Seeded PRNGs are deterministic.** With the seed derived from public values
  (MD5 of the password XOR a constant), the entire keystream is reproducible
  offline.
- **Integrity checks are oracles.** The MD5-of-plaintext check lets you verify
  a decryption without any server round-trip.
