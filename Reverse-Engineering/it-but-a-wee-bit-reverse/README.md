# Writeup — It But A Wee Bit Reverse

| | |
|---|---|
| **Category** | Reverse Engineering |
| **Points** | 15 |
| **Artifact** | 16 KB ELF 64-bit crackme (not stripped, `crackme2.c`) |
| **Flag** | `deadface{x0r_1s_not_encryption}` |

> Reverse the password check, then decrypt the flag.

---

## 1. Recon

A 16 KB ELF 64-bit binary, not stripped. Two interesting data blobs:

- `pw_enc` at `0x4010` — the obfuscated expected password
- `flag_enc` at `0x4020` — the encrypted flag

## 2. The password check

`main` reads input and requires **exactly 10 characters**. Each input byte is
XORed with `0x42` and compared against `pw_enc`:

```c
for (i = 0; i < 10; i++)
    if ((input[i] ^ 0x42) != pw_enc[i]) fail();
```

Inverting is trivial: `password[i] = pw_enc[i] ^ 0x42`.

## 3. The flag decryption

Once the password verifies, the flag is decrypted byte-by-byte, reusing the
password cyclically:

```c
for (i = 0; i < 31; i++)
    flag[i] = flag_enc[i] ^ password[i % 10];
```

Running the binary with the recovered password prints `Access granted!`,
confirming the password before the flag is derived — no guessing involved.

## 4. Flag

```
deadface{x0r_1s_not_encryption}
```

Submitted — platform returned **Correct**.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Static analysis of unstripped ELF | `pw_enc`/`flag_enc` located |
| 2 | Invert `input[i] ^ 0x42 == pw_enc[i]` (10 bytes) | Password recovered |
| 3 | `flag[i] = flag_enc[i] ^ password[i % 10]` (31 bytes) | Flag decrypted |
| 4 | Verify via binary (`Access granted!`), submit | **Correct** |

## Takeaways

- **XOR with a constant is not encryption** — as the flag itself says. A single
  known-plaintext byte would break it; here the comparison gives us everything.
- **Verify with the binary.** Running the crackme to confirm `Access granted!`
  validates the password before trusting the derived flag.
