# [FEATURE] AIS ZTE G1611: native configuration-format evidence

Draft only; not submitted.

## Description

I would like to investigate configuration compatibility for an owned AIS ZTE G1611. This report provides selected native-loader observations and asks how best to compare them with the existing implementation.

The inspected baseline is [`e3a1a7f56c2df26a02f52f72b16173d0702d9ad0`](https://github.com/mkst/zte-config-utility/commit/e3a1a7f56c2df26a02f52f72b16173d0702d9ad0). Generic explicit-key/IV type-6 decoding is already present after [PR #204](https://github.com/mkst/zte-config-utility/pull/204). This report does not claim G1611 support or ask for that generic feature to be added again.

## Native observations

- Platform: ZX279128S, dual Cortex-A9 ARMv7; reported hardware G1611 V9.1.41; photographed main PCB `73013F6600P_220700`.
- Recorded software fields: `G1611_PON_1.1`, extension `07e8P4N10396`, build date 2024-09-18; Linux 4.1.25. These are retained separately from the boot-software string below.
- Primary native CFG: `/userconfig/cfg/db_user_cfg.xml`; fallback: `/userconfig/cfg/db_backup_cfg.xml`.
- Retained analysis recorded a 60-byte header read and classified the examined native wrapper as type 6.
- `dbcCfgFileDecry` dispatched native types 5/6 through `DecryByAESCBC`; a CRC-related branch used `DecryByCRC`.
- Decryption and XML validation were separate parts of the load path.
- Normal loading tried primary first and used backup on primary failure in the examined path.
- The live bootstrap reported `V9.0.10P10N16`; a separate retained offline report named `V9.0.10P4N1`. Their equivalence remains unresolved, so these binary observations should not be generalized across builds.

The historical investigation rehearsed native restoration of an exact opaque saved CFG. It did not establish an external decode/encode codec or demonstrate compatibility of a web-exported `config.bin`.

## Compatibility question and next evidence

Does this native saved-CFG wrapper map directly to the current type-6 input format, or should it be compared as a different container? The native type number alone does not answer that question.

The next useful evidence would be a pinned-baseline reproduction, identification of the actual wrapper and derivation inputs, and a valid synthetic fixture with an expected decoded result. No fresh G1611 decode/encode test or CLI error log is claimed in this draft.

No device configuration, firmware image, key material, or identifying device/network data is attached. A shareable fixture has not yet been produced.

The companion case study and mechanism note can be linked here after they have public URLs.

A separate small README proposal documents the existing type-6 CLI and seed-string derivation. It does not depend on claiming G1611 support and can be discussed as a documentation-only change.
