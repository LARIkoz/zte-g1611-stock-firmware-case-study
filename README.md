# ZTE G1611 stock-firmware case study

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

A focused account of an investigation into persistent maintenance access and an isolated Ethernet gateway on an owned AIS ZTE G1611. It covers selected mechanisms, successful experiments, and the physical-link test that invalidated the proposed gateway setup.

Research cutoff: **25 October 2026**. Later device work is outside this account.

## Read the case

- [Case study](CASE-STUDY.md): the investigation, the turning points, and the measured outcome.
- [Key mechanisms](MECHANISMS.md): native configuration loading, saved-command startup, service ordering, and packet isolation.
- [Reproducing selected checks](REPRODUCING.md): conditions, commands, expected observations, rollback criteria and explicit reproduction gaps.
- [Upstream issue draft](upstream/ISSUE.md): the configuration-format question relevant to [mkst/zte-config-utility](https://github.com/mkst/zte-config-utility).
- [Type-6 documentation proposal](upstream/TYPE6-DOC-PROPOSAL.md): a concrete README addition explaining an existing decoder path and its parameter semantics.

## What was established

Persistent userspace maintenance services were demonstrated across reboot. Temporary Ethernet isolation worked in a controlled packet test, but a physical cable interruption caused native firmware to restore LAN forwarding. The proposed isolated gateway therefore failed its acceptance criterion.

G1611 decode/encode compatibility with `zte-config-utility` remains unestablished. The native configuration observations are research input for a compatibility investigation.

## Contribution status

These documents and an issue draft are prepared locally. No upstream issue or PR has been submitted as part of this publication preparation. There is no claimed merged contribution. If a submission is made, add its actual URL and status here.

Two concrete contributions are available for discussion: the G1611 native-format research report, and a README explanation of existing type-6 decoding. A G1611 support implementation still needs a tested format/parameter mapping. Synthetic type-6 regression tests would be another possible contribution, but no new test patch is included or claimed as executed.

Generic type-6 decoding already exists at the [inspected upstream baseline](https://github.com/mkst/zte-config-utility/commit/e3a1a7f56c2df26a02f52f72b16173d0702d9ad0), following [PR #204](https://github.com/mkst/zte-config-utility/pull/204). That work belongs to its upstream contributors.

## Publication scope

This is an edited technical narrative, not a complete transcript or a reproducibility dataset. The underlying working records remain private; readers cannot independently replay every historical experiment from these documents alone.

The case now includes a device/software profile, the examined native startup contract and a practical evidence checklist. Read-only inventory can be repeated with compatible access. The native DB adapter/save route and original topology-test scripts remain unpublished, so full copy-and-run reproduction is explicitly incomplete. No successful G1611 decoder or persistent isolated gateway is supplied.

This document set contains no raw session archive, device configuration, firmware image, packet capture, credential, private account or network identifier. Native vendor paths and function names are retained where they explain the mechanism. The earlier bulk journal packages are outside this publication.

## Keeping the case independent

Publish these documents in a standalone case repository, with its own history and a retained local copy. The narrative, hardware/software profile and essential mechanism descriptions are stored here; they remain readable if the upstream repository is closed or deleted. Upstream links provide attribution and source context rather than being the only copy of the explanation.

Keep a versioned document-only archive and checksum alongside the local repository, and optionally attach that archive to a case release. This preserves the case documents, not the upstream tool. Preserving executable tooling additionally requires a complete upstream checkout/archive and its dependency information; any redistributed upstream source must retain its accompanying license notices. A fork for a future PR serves a separate contribution workflow.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to report corrections to the case and how the configuration-format observations feed a focused upstream discussion in [mkst/zte-config-utility](https://github.com/mkst/zte-config-utility).

## License

The original text of this repository is licensed under [CC BY 4.0](LICENSE). Referenced third-party projects keep their own licenses; no vendor firmware, configuration, capture, credential, or private identifier is included.
