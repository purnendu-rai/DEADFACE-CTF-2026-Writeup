# Writeup — The Mark

| | |
|---|---|
| **Category** | OSINT + Crypto + Steg (hybrid) |
| **Author** | @syyntax |
| **Points** | 350 |
| **Artifacts** | `bigdawg-1.jpg`, `tgri_records.csv.gpg` (via shortlink) |
| **Flag** | `deadface{Santos_Staudenmaier}` |

> DEADFACE bragged about accessing sensitive information belonging to TGRI and
> is targeting a person. Identify the mark. Flag format: `deadface{first_last}`.

---

## 1. The description steg (a decoy)

The challenge description carries 240 zero-width characters after a decoy
`flag{08a7cdddc7dcbba8}`. Decoded (U+200B=0, U+200C=1):

```
flag{s0lv3_m3_h4rd3r_AI_d4ddy}
```

An author taunt aimed at AI solvers — **not the flag**.

## 2. OSINT: GhostTown and DoomScroll

The old GhostTown domain (`ghosttown.xyz`) is dead/parked; the 2026 forum
lives at `ghosttown.deadface.io`. The thread **"Who knew doomscrolling could
be so useful?"** reveals DEADFACE hides messages in **DoomScroll** post
metadata via the API (`doomscroll.dctf.io`, `/api/posts/<id>/meta`) — fields
the UI never renders. `daem0n` references "our mark's sensitive file" held by
`d34th`.

## 3. The DoomScroll covert channel

`GET /api/posts/<id>/meta` returns hidden `meta` entries. The values use
whitespace steganography — three invisible space codepoints as a ternary code:

- U+2002 (EN SPACE) → `0`
- U+2003 (EM SPACE) → `1`
- U+2009 (THIN SPACE) → `2`

Six base-3 digits → 1 byte (`chr(int(chunk, 3))`). The `/api/posts` endpoint
paginates with limit+offset (251 posts total).

Decoded covert DEADFACE conversation:

| Post | Meta key | Decoded |
|---|---|---|
| 223 | `pt1` | "TGRI's CFO, **Santos Staudenmaier**, has my attention. …I want to know what he is hiding." |
| 224 | `pt2` | "…I've seen his name all over TGRI's financial reports…" |
| 226 | `pt3` | "…His financial records could expose TGRI's confidential research programs…" |
| 227 | `pt4` | "…Santos seems pretty protective of his reputation…" |
| 228 | `pt5` | "…Staudenmaier is our priority. He may be the key to bringing TGRI down." |
| 235 | `passcode` | `https://pastebin.com/GBcnwhvZ` |
| 200 | `what whaaaaat` | "Check it out, we can use DoomScroll for covert comms" |
| 274 | `Look here` | `AI Check: flag{sUbm1T_m3_1f_Im_Ch34TiNG}` — another AI-taunt decoy |

## 4. The crypto (unsolved — and unneeded)

The Pastebin ("Cookie Recipe") holds base64 `KKq5/Gq6qHW4isNitU8wR02wW/+6y+rH7bZODH0Mnc0=`
(32 bytes). Despite 100+ decryption attempts (XOR, AES, Vigenère-36 with
numeric keys, Beaufort, columnar transposition) and the GhostTown hint
("Vigenère-36, numbers in key"), it was never cracked. The mark's identity
does **not** depend on it — it feeds the follow-up challenge (Get That Bag).

## 5. The mark

The covert comms explicitly name the target — **TGRI's CFO, Santos
Staudenmaier**, "our priority". The decrypted `tgri_records.csv` corroborates
it: `SYN-EMP-0001`, Santos Staudenmaier, Chief Financial Officer, with flagged
records (missing receipts, missing contract codes, a paused vendor payment, a
deferred bonus, an open audit review) while the rest of the roster is clean.

```
deadface{Santos_Staudenmaier}
```

Submitted and **accepted by the platform** — the only flag in this repo with a
platform confirmation.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Description zero-width steg | author taunt decoy, not the flag |
| 2 | GhostTown thread OSINT | DoomScroll `/api/posts/<id>/meta` channel |
| 3 | Whitespace ternary decode (U+2002/3/9) | covert convo naming the mark |
| 4 | Pastebin crypto | unsolved — not needed for identity |
| 5 | Cross-check vs `tgri_records.csv` || **TGRI CFO Santos Staudenmaier** | |

## Takeaways

- **APIs render what the UI hides.** The `/meta` endpoint exposed fields
  invisible in the DoomScroll frontend — always probe the API behind a UI.
- **Three codepoints = ternary.** When you see exactly three distinct
  invisible characters, think base-3, not binary.
- **Decoys target the solver, not the solution.** Two separate AI-taunt
  flags (`s0lv3_m3_h4rd3r_AI_d4ddy`, `sUbm1T_m3_1f_Im_Ch34TiNG`) existed to
  waste time — the real signal was the covert conversation.
- **Confirm identity against the leaked record.** The CSV is a decoy-rich
  roster; only the CFO's records match the chatter.

## Tools Used

- Live browser (GhostTown / DoomScroll / API reads)
- Python (zero-width + whitespace-ternary steg decoders)
- GnuPG (the `.gpg` archive, for the follow-up)
