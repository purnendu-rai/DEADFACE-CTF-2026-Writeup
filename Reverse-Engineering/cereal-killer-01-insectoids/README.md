# Writeup — Cereal Killer 01: Insectoids

| | |
|---|---|
| **Category** | Reverse Engineering |
| **Author** | @TheZeal0t |
| **Points** | 50 |
| **Artifact** | `re01-win-x86.exe` (PE32 console binary, MinGW, with debug info) |
| **Flag** | `deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}` |

> The insectoids demand their favorite terrestrial breakfast cereal. Crack the
> password check and find out what they eat.

---

## 1. Recon

Static analysis with `objdump` on the PE32 binary (debug info present, not stripped).
The flow in `main` is straightforward:

1. Reads a password from stdin via `getline`.
2. SHA1-hashes the password, hex-encodes it (`%02x` per byte), and `strcmp`s the
   result against a stored blob (`blob1`).
3. On match → "Access granted" → a second blob is decrypted → leads to the flag.

## 2. The custom base64

`blob1` and `blob2` are wide-char strings in `.data`, decoded with a **custom
base64 alphabet** — a 64-char lookup table at `0x40b860`. Characters not in the
table (e.g. trailing `2`s) act as terminators. Blob1's last quad decodes to
exactly 40 bytes — a SHA1 hex string.

Target hash: `68158678a3f9d76a3d1ca12d20b2d6af9bd3f290`

## 3. Cracking the password

The password is the insectoids' favorite terrestrial breakfast cereal. I ran a
cereal wordlist against the target SHA1 and got a hit.

## 4. From password to flag

With the correct password, blob2 is **RC4-decrypted using the password as the
key**, yielding a URL:

```
https://cerealkiller.lyttonlabs.org/unwanted-guests/ick.png
```

The image (Dr. Charles K. Gesch-ICK-ter, Lytton Labs) has the flag printed at
the top of the page:

```
deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}
```

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | `objdump` on PE32 | password → SHA1 → hex → strcmp against blob1 |
| 2 | Custom base64 table at `0x40b860` | 40-byte target SHA1 hex |
| 3 | Wordlist vs `68158678a3f9d76a3d1ca12d20b2d6af9bd3f290` | password recovered |
| 4 | RC4(blob2, password) | URL to `ick.png` |
| 5 | Image inspection | **`deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}`** |

## Takeaways

- **Debug info is a gift.** The binary shipped unstripped — function names and
  string layout made the password→hash→compare flow trivial to read.
- **Custom base64 alphabets are just lookup tables.** Once you spot the 64-char
  table, decoding is mechanical.
- **Follow the crypto to the end.** The password only unlocks blob2; the flag
  lives one hop further at the URL.

## Tools Used

- `objdump` (static analysis)
- Cereal wordlist (password cracking)
- Custom base64 / RC4 decoding (Python)
