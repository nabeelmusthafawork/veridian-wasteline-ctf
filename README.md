# Veridian WasteCo — Two-Layer CTF Lab

A self-contained, two-layer capture-the-flag lab shipped as a single VirtualBox
appliance. Layer 2 (a second VM) runs **nested inside** Layer 1 as a QEMU guest,
so both flags chains are reachable from one import.

- **Layer 1** — *Veridian Operations Portal* (`http://192.168.56.102`)
- **Layer 2** — *WasteCo Owner Panel* (`http://192.168.56.50`), nested QEMU VM
- **Flags** — 4 total (user + root, one pair per layer)

## Contents

| File | What it is |
|---|---|
| `veridian-wasteline-ctf.ova.0*.part` (release assets) | The full appliance, split into 4 parts (GitHub release assets are capped at 2 GiB each) |
| `SHA256SUMS` (release asset) | Checksums for every part and for the reassembled OVA |
| `veridian-wasteline-walkthrough.pdf` | Plain-text, point-by-point intended solution for both layers |

## Reassemble and import the OVA

```sh
# download all 4 parts + SHA256SUMS from the release, then:
sha256sum -c SHA256SUMS          # verifies the parts
cat veridian-wasteline-ctf.ova.0*.part > veridian-wasteline-ctf.ova
echo "b86226087fbd8a2052dd56d5b6eb4f653dd1cb8770cdb26ed0262a2998c02faa  veridian-wasteline-ctf.ova" | sha256sum -c
```

Import with `File → Import Appliance` in VirtualBox, then boot the VM
(`CTF-Debian13`).

## Starting the lab

1. Log in on the console as **`ctf` / `ctf`**.
2. Layer 1 is already up: `http://192.168.56.102`.
3. Start the nested Layer 2 VM: `sudo /root/launch`
   (TCG emulation — allow 40–120 s before `http://192.168.56.50` answers).
   `sudo /root/launch stop` stops only the inner VM; Layer 1 stays up.
4. The host must be on the VirtualBox host-only network (`192.168.56.0/24`) —
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

```
b86226087fbd8a2052dd56d5b6eb4f653dd1cb8770cdb26ed0262a2998c02faa  veridian-wasteline-ctf.ova  (reassembled, 4200549376 bytes)
c3ebfe74774d0225ff2b9cbdea32d48a810bc3e4f59ef95fda646f272a03ccdb  veridian-wasteline-ctf.ova.00.part
94af3d011c3e9b5573a0448c8fdf772dc41d6d8a7da59978df7332f21c5ff11b  veridian-wasteline-ctf.ova.01.part
506b9840224ffff2a22ad119ec95c2e5576d98ac57e8fa29790eca17c152189c  veridian-wasteline-ctf.ova.02.part
3e7306b6d11cb1706aa72964d5dabc8d2d083d9578478d7b9029bef3469e93e0  veridian-wasteline-ctf.ova.03.part
```
