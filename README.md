# ZTE G1611 (AIS, stock firmware): persistent root-level maintenance access with Tailscale + SSH

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

A practical how-to case study of turning an **owned, stock-firmware AIS ZTE G1611** GPON router into a remotely maintainable box over Tailscale and SSH — without flashing custom firmware and without a writable root filesystem. It documents what worked, how, and (just as usefully) what did **not**.

Research cutoff: **25 October 2026**. All work was done on hardware the author owns.

## Device identifiers (for search / same-device owners)

If you have this exact unit, these are the markings and strings to match. They are product/build identifiers — not the unit's serial number, MAC, or network identity.

| Item                     | Value                                                            |
| ------------------------ | ---------------------------------------------------------------- |
| Model / operator variant | **AIS ZTE G1611** (GPON ONT)                                     |
| Hardware version         | **G1611 V9.1.41**                                                |
| Main PCB marking         | **`73013F6600P_220700`**                                         |
| Antenna-board marking    | **F6600P V9.1**                                                  |
| Main SoC / CPU           | **ZTE ZX279128S** — dual Cortex-A9, ARMv7                        |
| Wi-Fi                    | **MT7916** (`mt7916.ko`)                                         |
| RAM / Flash              | 256 MiB / 128 MiB                                                |
| Firmware string          | **`G1611_PON_1.1`** (extension `07e8P4N10396`, build 2024-09-18) |
| Boot-software string     | **`V9.0.10P10N16`**                                              |
| Kernel / toolchain       | **Linux 4.1.25**, Buildroot 2017.05, GCC 4.9.4, glibc 2.26       |
| Stock tools              | BusyBox 1.17.2, curl 7.59.0                                      |
| Tailscale build used     | official **1.104.1** ARM, GOARM5 (userspace, no TUN)             |

## What this achieves — and what it does not

**Achieved (reboot-tested):**

- ✅ Temporary **root shell** through a factory-service path.
- ✅ Logical flash / partition **backups**, transfer-checked; a round trip between two healthy firmware banks.
- ✅ **Persistence without a writable rootfs**: a saved-command native startup consumer (`CMDPARAMINFO`) launches a maintenance binary after boot.
- ✅ **Tailscale** (userspace networking, no TUN) enrolled and authenticated, reachable **after reboot**.
- ✅ Restricted **SSH (Dropbear)** with a **delayed start** to avoid a race with native firewall init.
- ✅ A bounded **service reduction** that survived reboot.

**Not achieved (honest limits — don't expect these from this repo):**

- ❌ **No custom firmware flash / no writable root filesystem.** The active rootfs stayed read-only; a remount that returned "success" was contradicted by readback and file-creation. We did **root**, not reflash.
- ❌ **No isolated Ethernet gateway for a downstream client.** It worked in a controlled packet test, then **failed a physical cable-cycle test** — native firmware restored LAN forwarding after a ~5 s carrier interruption.
- ❌ **No external `config.bin` decoder/encoder.** G1611 compatibility with `zte-config-utility` is unresolved (see Contribution status).

## How we did it (overview)

Each step has its measured detail in the linked documents.

1. **Get temporary root** via a factory-service path → a shell, plus logical flash/partition backups. Detail: [CASE-STUDY.md](CASE-STUDY.md#a-root-shell-did-not-provide-a-writable-firmware-image).
2. **Persist without touching the read-only rootfs** — use the native `CMDPARAMINFO` saved-command startup consumer to launch a background maintenance binary after native init. Detail: [MECHANISMS.md](MECHANISMS.md#saved-command-startup-through-cmdparaminfo).
3. **Bring up Tailscale** in userspace mode (no TUN) as the maintenance path; confirm enrollment and authenticated reachability after a reboot. Detail: [CASE-STUDY.md](CASE-STUDY.md#starting-a-process-and-preserving-a-working-service-were-separate-tasks).
4. **Add restricted SSH (Dropbear)** with a delayed start so it comes up after native firewall init. Same section.
5. **Reduce services** within a bounded experiment that accounts for native process-controller restart behavior. Same section.
6. **Step-by-step conditions, commands, expected observations, and rollback criteria:** [REPRODUCING.md](REPRODUCING.md) — including which steps are repeatable read-only checks and which need adapters/scripts that are **not** published.

## Read the full case

- **[CASE-STUDY.md](CASE-STUDY.md)** — the investigation, the turning points, and the measured outcome.
- **[MECHANISMS.md](MECHANISMS.md)** — native configuration loading, saved-command startup, service ordering, packet isolation.
- **[REPRODUCING.md](REPRODUCING.md)** — conditions, commands, expected observations, rollback, and explicit reproduction gaps.
- **[upstream/ISSUE.md](upstream/ISSUE.md)** / **[upstream/TYPE6-DOC-PROPOSAL.md](upstream/TYPE6-DOC-PROPOSAL.md)** — the configuration-format question and documentation proposal for [mkst/zte-config-utility](https://github.com/mkst/zte-config-utility).

## Scope and safety

This is an edited technical narrative, not a full transcript or a copy-and-run dataset. Read-only inventory checks are repeatable with compatible access; the native DB adapter/save route and the original topology-test scripts remain unpublished, so full reproduction is explicitly incomplete.

This document set contains **no** raw session archive, device configuration, firmware image, packet capture, credential, private account, or network identifier. Native vendor paths and function names are kept only where they explain a mechanism.

## Contribution status

These documents are published, and the following upstream submissions to [mkst/zte-config-utility](https://github.com/mkst/zte-config-utility) are open (under review; no merged contribution is claimed):

- Issue [#208](https://github.com/mkst/zte-config-utility/issues/208) — native configuration-format evidence and the compatibility question for the owned AIS ZTE G1611.
- Pull request [#209](https://github.com/mkst/zte-config-utility/pull/209) — a documentation-only README example for the existing type-6 decode path.

Generic type-6 decoding already exists upstream after [PR #204](https://github.com/mkst/zte-config-utility/pull/204); that work belongs to its contributors. A G1611 "supported model" claim would still need a verified container/derivation mapping and a valid synthetic fixture — neither is included here.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to report corrections and how the configuration-format observations feed the focused upstream discussion.

## License

The original text of this repository is licensed under [CC BY 4.0](LICENSE). Referenced third-party projects keep their own licenses; no vendor firmware, configuration, capture, credential, or private identifier is included.
