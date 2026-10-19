# Writeup — Cereal Killer 04

| | |
|---|---|
| **Category** | Reverse Engineering |
| **Points** | 250 |
| **Author** | @TheZeal0t |
| **Artifact** | `re05.exe` (PE32 .NET, 14 KB, SHA256 `045f72d2…bdef2f96f`) |
| **Flag** | `deadface{Before_Consenting_to_Med_Exam_Ascertain_Destination_of_Finger!}` |

> A .NET loader hides an encrypted inner DLL. Peel the layers.

---

## 1. Recon

A 14 KB .NET PE32 binary. The `.cctor` (static constructor) references a
2-byte XOR key — but that key (`[0x01, 0x01]`) is a **decoy** (a KlippyAI-style
trap for automated unpackers).

## 2. The real outer key

The genuine 2-byte repeating XOR key is derived from the **MZ header** itself:
`[0x41, 0x33]` (`"A3"`). Applying it to the 6656-byte blob at RVA `0x4000`
yields a valid **PE32 DLL** — the real second stage. (The `.cctor` key only
produces garbage, which is exactly what the trap wants you to chase.)

## 3. The inner DLL

The decrypted DLL exposes:

- `correctHash`: MD5 `7730e6666bec3dc346c105bd2012a813`
- `encryptedFlag`: 80 bytes
- `iv`: `"thegreysarecomin"`

The password is the greys' favorite cereal — **Boo Berry**:

```
password = booberry
```

Verified: `MD5("booberry") == 7730e6666bec3dc346c105bd2012a813`.

## 4. Decrypting the flag

Key derivation: **PBKDF2-HMAC-SHA1**, 1000 iterations, salt = `iv`
(`"thegreysarecomin"`), password = UTF-8 `"booberrybooberry"`, output 32 bytes.
Decryption: **AES-256-CBC** with that key and IV = `iv`.

```python
key = PBKDF2HMAC("booberrybooberry", salt=b"thegreysarecomin",
                 iterations=1000, dklen=32)
flag = AES_CBC_decrypt(encryptedFlag, key, iv=b"thegreysarecomin")
```

## 5. Flag

```
deadface{Before_Consenting_to_Med_Exam_Ascertain_Destination_of_Finger!}
```

Verified via full Python reimplementation (XOR → DLL → PBKDF2 → AES-CBC).

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Inspect .NET `.cctor` key | `[0x01,0x01]` — decoy, produces garbage |
| 2 | XOR blob at RVA `0x4000` with MZ-derived `[0x41,0x33]` | Valid inner PE32 DLL |
| 3 | MD5 password check in DLL | `booberry` |
| 4 | PBKDF2-HMAC-SHA1 (1000 iters) → AES-256-CBC | Flag decrypted |

## Takeaways

- **Distrust the obvious key.** The `.cctor` key was a planted trap — the real
  key came from the MZ header, a much less "helpful-looking" source.
- **Layered packers reward patience.** Each layer (XOR → DLL → PBKDF2 → AES)
  is simple in isolation; the challenge is not stopping at the first plausible
  answer.
