# Writeup — Cereal Killer 02: The Nordics

| | |
|---|---|
| **Category** | Reverse Engineering |
| **Author** | @TheZeal0t |
| **Points** | 150 |
| **Artifacts** | `re03-ck02.7z` (password `KlippyAI`) → `re03-lin-x86.bin`, `re03-win-x86.exe`, `KlippyAI-trained-on-CK02.exe` |
| **Flag** | `deadface{N0rdics-R3ally-like-Frank3nb3rry-cuz-bro-is-tall-like-they-are!}` |

> The Nordics have a favorite cereal too. The key has been tampered with —
> "TIME is REVERSIBLE, but the KEY is NOT... Go Back! GO BACK!!!"

---

## 1. Unpacking

The challenge ships as an encrypted 7z (SHA256 verified, password `KlippyAI`).
Extracted with `py7zr`:

- `re03-lin-x86.bin` — Linux ELF, the real target
- `re03-win-x86.exe` — Windows twin
- `KlippyAI-trained-on-CK02.exe` — a packed, obfuscated .NET "surprise" program

The .NET binary only yields static strings and smells like a **red herring** —
it was never executed.

## 2. The binary's logic

`re03-lin-x86.bin` is not stripped. Its logic (reconstructed as `re03.c`):

1. XORs two `.rodata` blobs (`0x101` and `0x100` bytes, at offsets `0x2a20` /
   `0x2b24`) into a `0x100` buffer.
2. Runs `memmem` over the result looking for the needle `"badf00d"`.

With the blobs as shipped, the needle is **absent** → the error path fires:

```
The key has been tampered with...
TIME is REVERSIBLE, but the KEY is NOT... Go Back! GO BACK!!!
```

## 3. The fix

The binary's own hint says the key was tampered with and that time is
reversible. The fix: **reverse the KEY blob (blob2) before XORing**.

With blob2 reversed, the needle is found at offset 206, and the success path
prints an ASCII grid. Stripping the space/dot filler, the grid reads:

```
feed / face / deadbeef / dead / face{ / N0rdics / -R3ally-like- /
Frank3nb3rry- / cuz-bro-is- / tall-like- / they-are! / badf00d / da7e4107
```

i.e. **"Nordics really like Frankenberry cuz bro is tall like they are!"** —
Franken Berry being the cereal answer. Per the challenge rules (no
spaces/periods/asterisks), the flag is:

```
deadface{N0rdics-R3ally-like-Frank3nb3rry-cuz-bro-is-tall-like-they-are!}
```

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | `py7zr` with password `KlippyAI` | three binaries, one red herring |
| 2 | Static analysis of the ELF | XOR(blob1, blob2) → `memmem("badf00d")` |
| 3 | Follow the hint: reverse the KEY blob | needle found at offset 206 |
| 4 | Read the ASCII grid | **`deadface{N0rdics-R3ally-like-Frank3nb3rry-cuz-bro-is-tall-like-they-are!}`** |

## Takeaways

- **Read the error messages.** "TIME is REVERSIBLE, but the KEY is NOT" is the
  challenge telling you exactly which transform to apply and where.
- **Red herrings are part of the game.** The packed .NET binary existed only to
  eat your time — a quick strings pass was enough to deprioritize it.
- **Verify the needle, don't assume the XOR.** Running the documented transform
  with the blobs as-shipped fails; only the reversed key produces the needle.

## Tools Used

- `py7zr` (7z extraction — no p7zip on the machine)
- Static binary analysis (reconstructed `re03.c`)
- Python (blob reversal, grid rendering)
