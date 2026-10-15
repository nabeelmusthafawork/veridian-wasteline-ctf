# Veridian WasteCo — Two-Layer CTF Lab

A self-contained, two-layer capture-the-flag lab shipped as a single VirtualBox
appliance. Layer 2 (a second VM) runs **nested inside** Layer 1 as a QEMU guest,
so both flag chains are reachable from one import.

- **Layer 1** — *Veridian Operations Portal* (`http://192.168.56.102`)
- **Layer 2** — *WasteCo Owner Panel* (`http://192.168.56.50`), nested QEMU VM
- **Flags** — 4 total (user + root, one pair per layer)

## Contents

| File | What it is |
|---|---|
| `veridian-wasteline-ctf.ova.part-*` ([release assets](https://github.com/nabeelmusthafawork/veridian-wasteline-ctf/releases)) | The full appliance, split into 14 parts (13 × 300 MB + 1 × ~106 MB; GitHub caps release assets at 2 GiB each) |
| `SHA256SUMS` (release asset) | Checksums for every part and for the reassembled OVA |
| `veridian-wasteline-walkthrough.pdf` | Plain-text, point-by-point intended solution for both layers |

## Download and reassemble

Download all 15 assets (`SHA256SUMS` + 14 parts) from the
[latest release](https://github.com/nabeelmusthafawork/veridian-wasteline-ctf/releases),
then:

```sh
sha256sum -c SHA256SUMS                        # verifies every part
cat veridian-wasteline-ctf.ova.part-* > veridian-wasteline-ctf.ova
echo "b86226087fbd8a2052dd56d5b6eb4f653dd1cb8770cdb26ed0262a2998c02faa  veridian-wasteline-ctf.ova" | sha256sum -c
```

The reassembled `veridian-wasteline-ctf.ova` is 4200549376 bytes — the exact
original file, byte for byte.

## Import and start

1. VirtualBox → `File → Import Appliance` → select the OVA → boot `CTF-Debian13`.
2. Log in on the console as **`ctf` / `ctf`**.
3. Layer 1 is already up: `http://192.168.56.102`.
4. Start the nested Layer 2 VM: `sudo /root/launch`
   (TCG emulation — allow 40–120 s before `http://192.168.56.50` answers).
   `sudo /root/launch stop` stops only the inner VM; Layer 1 stays up.
5. The host must be on the VirtualBox host-only network (`192.168.56.0/24`) —
   the inner VM has **no port forwarding** and is reachable only from there.

## Solving

`veridian-wasteline-walkthrough.pdf` in this repo is the complete intended
workflow, step by step: OTP source-disclosure → SSRF → git-history credential
recovery → DB hashes → rockyou crack → SSH → SUID injection (Layer 1), and
SQLi + trusted-device bypass → automation-hook webshell → key extraction →
rockyou crack → SSH → writable-config sudo privesc (Layer 2).

Two different privesc bug classes per layer, by design. Every dead end in the
app (reject-all 2FA, `403` on `/storage/*`) is intentional — see the
"Design notes" section of the walkthrough.

## Checksums

The authoritative checksums are in `SHA256SUMS` (release asset). The two that
matter:

```
b86226087fbd8a2052dd56d5b6eb4f653dd1cb8770cdb26ed0262a2998c02faa  veridian-wasteline-ctf.ova  (reassembled, 4200549376 bytes)
```
