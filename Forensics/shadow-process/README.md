# Analysis — SHADOW PROCESS: The In-Memory Intruder

| | |
|---|---|
| **Category** | Memory Forensics |
| **Points** | 350 |
| **Artifact** | `linux_memory_triage.raw.xz` (LiME RAM dump) |
| **Status** | Analysis — flag format unresolved |

> An air-gapped Linux terminal was hit by in-memory process hollowing: a
> legitimate `scada_logger` daemon hollowed, an anonymous RWX region hosting an
> injected 64-bit ELF. Carve the implant, reverse its XOR decryption loop, and
> build the `CODEX{...}` flag.

---

## 1. The hollowed process

Volatility-style triage of the dump:

- Hollowed process: **PID 1337 `scada_logger`** with an anonymous RWX VMA at
  **`0x400000`** (dump region labeled `[anon:injected_hollowed_payload]`).
- Full-dump sweep: non-zero regions only at `0x0` (EMiL header), `0x10000`
  (boot cmdline), `0x100000` (kernel version), `0x200000` (dmesg),
  `0x300000` (process list + PID 1337 maps), `0x400000` (implant ELF),
  `0x600000` (`/proc/net/tcp`: `192.168.10.45:49812 → 10.240.12.88:8443`
  ESTABLISHED — corroborates the C2 below).

## 2. The carved implant

Carved from `0x400000`: a 64-bit x86-64 ELF (576 bytes, zeros after), entry
`0x400080`, 32-byte `.text`, 119-byte `.rodata`.

`.text` runs a simple loop: **XOR over `0x77` bytes of `.rodata` with key
`0x5A`**, then `sys_exit`.

## 3. The decrypted config (definitive)

XOR-decrypting `.rodata` with `0x5A` yields the implant's config:

| Field | Value |
|---|---|
| Squad auth tag | **`949831a1`** |
| Payload classification | **`h0ll0w3d_3lf`** |
| C2 gateway | **`10.240.12.88:8443`** |
| Beacon interval | 60 seconds |

Verified three times; a full-dump string sweep found no other flag/auth/squad
strings anywhere. The data is definitive.

## 4. Flag attempts

1. `CODEX{sh4d0w_pr0c__h0ll0w3d_3lf}` — wrong (double-underscore template
   copied from the directive; missing the auth tag).
2. `CODEX{sh4d0w_pr0c_949831a1_h0ll0w3d_3lf}` — directive format
   `CODEX{sh4d0w_pr0c_<squad_auth_tag>_h0ll0w3d_3lf}` — **rejected** by the
   platform.
3. Untested candidates (submission results never obtained):
   `CODEX{949831a1_h0ll0w3d_3lf}`, then `CODEX{949831a1}`.

**Open question:** the flag-*joining* format, not the data. Everything the
dossier asks to recover was recovered; how the challenge wants it joined is
unknown.

---

## Summary

| Step | Technique | Result |
|---|---|---|
| 1 | Memory triage | PID 1337 `scada_logger`, RWX VMA at `0x400000` |
| 2 | Physical carving | 64-bit ELF implant, entry `0x400080` |
| 3 | Reverse the `.text` XOR loop (key `0x5A`) | auth tag `949831a1`, `h0ll0w3d_3lf`, C2 `10.240.12.88:8443` |
| 4 | Dump-wide corroboration | `/proc/net/tcp` confirms the C2 session |

## Takeaways

- **Anonymous RWX VMAs are the first thing to check** in a process-hollowing
  triage — legitimate daemons don't have them.
- **Tiny `.text` + XOR loop = config decryptor.** 32 bytes of code doing one
  loop over `.rodata` is the classic packed-config pattern; the key is
  usually a single byte.
- **Corroborate across the dump.** The `/proc/net/tcp` ESTABLISHED session to
  the decrypted C2 is independent confirmation the config is real.
- **Data ≠ format.** You can recover every byte correctly and still miss the
  flag if the joining format is unknown — keep candidates minimal
  (`{tag}`, `{tag_classification}`) and test them.

## Tools Used

- Volatility 3-style triage / physical carving
- ELF analysis, XOR decryption (Python)
- Dump-wide string sweeps
