# Contributing

This repository is a focused technical case study, not an actively developed
tool. Two kinds of contribution are welcome.

## Corrections and discussion of the case

If a described mechanism, measurement, or qualification is wrong or unclear,
open an issue with the specific document and section. Please keep reports
grounded in observable evidence. Do not post vendor firmware images, device
configurations, packet captures, credentials, or private network/account
identifiers — such content will be removed.

## Upstream configuration-tooling contribution

The configuration-format observations in this case are intended as input to a
focused discussion in the upstream project
[mkst/zte-config-utility](https://github.com/mkst/zte-config-utility), not as a
competing implementation here.

- [`upstream/ISSUE.md`](upstream/ISSUE.md) is a draft feature/format issue for
  that project: it asks how the examined AIS ZTE G1611 native configuration
  wrapper should be compared with the existing input-container handling.
- [`upstream/TYPE6-DOC-PROPOSAL.md`](upstream/TYPE6-DOC-PROPOSAL.md) is a
  documentation-only proposal describing the existing type-6 decoder path and
  its `--key` / `--iv` seed-string semantics.

Scope boundaries for any upstream submission:

- Generic explicit-key/IV type-6 decoding already exists upstream after
  [PR #204](https://github.com/mkst/zte-config-utility/pull/204); that work
  belongs to its contributors and is not reproposed.
- No G1611 "supported model" claim is made. A real support claim still needs a
  verified native-wrapper/container mapping and a valid synthetic fixture with
  an expected decoded result — neither is included here.
- Keep actual key/IV seed strings and any decrypted output private; sanitize
  logs before attaching them to an upstream issue.
