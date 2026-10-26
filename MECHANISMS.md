# Key mechanisms and the measurements that distinguished them

These notes describe selected observations from the G1611 investigation through 25 October 2026. They are an explanation of the mechanisms, not a deployment script. Native paths and function names below identify vendor components.

## Native configuration loading

The examined native loader used the following files:

| Role | Native path |
| --- | --- |
| Primary saved configuration | `/userconfig/cfg/db_user_cfg.xml` |
| Backup fallback | `/userconfig/cfg/db_backup_cfg.xml` |

The `.xml` suffix did not establish plaintext content. Retained analysis recorded a 60-byte native header read and classified the examined wrapper as type 6.

The trace associated `dbcCfgFileIsEncry` with encryption checking, `dbcCfgFileDecry` with dispatch, and `dbLoadFileDecry` / `dbFileLoadCfg` with loading. Types 5/6 reached `DecryByAESCBC`; a CRC-related branch used `DecryByCRC`. Decryption, wrapper checks, and XML validation were distinct parts of the load path.

The normal path tried the primary configuration and used backup on primary-load failure. The examined path did not establish a general preference for the backup with the newest epoch.

This supports three limited conclusions:

- Native saved configuration can be encrypted despite its filename.
- An exact opaque saved CFG can be useful for native restoration without an external codec.
- A native type number alone does not establish compatibility with a third-party tool's container format.

The live bootstrap reported `V9.0.10P10N16`; a separate retained offline report named `V9.0.10P4N1`. Their equivalence remains unresolved. Binary-analysis observations must retain their image provenance rather than being assigned to every G1611 build.

At upstream commit [`e3a1a7f56c2df26a02f52f72b16173d0702d9ad0`](https://github.com/mkst/zte-config-utility/commit/e3a1a7f56c2df26a02f52f72b16173d0702d9ad0), generic explicit-key/IV type-6 decoding was already present after [PR #204](https://github.com/mkst/zte-config-utility/pull/204). The remaining G1611 work is to compare the actual container, establish the required derivation inputs, and test a valid shareable fixture. None of those gaps is resolved merely by observing AES-CBC in the native binary.

## Saved-command startup through CMDPARAMINFO

The useful chain was:

```mermaid
flowchart LR
    A[Saved database row] --> B[Native startup consumer]
    B --> C[Process controller]
    C --> D[Short-lived launcher]
    D --> E[Background maintenance service]
```

The examined consumer read its saved view after native initialization. A short executable path in the immediate-mode command reached a launcher on a writable location. This avoided the need to add startup files to the read-only root filesystem.

The controller's synchronous execution had timeout/cleanup behavior. A long-running service therefore needed a suitable background launcher rather than occupying the synchronous startup call indefinitely. A distinct-boot canary established execution before the persistent service was tested.

The retained trace gives a more concrete contract:

| Field / behavior | Examined contract |
| --- | --- |
| DB view | `CMDPARAMINFO`, listed in the `DEV` context |
| `CmdName` | Short unique absolute executable path |
| `CmdInfo` | Complete command line; for the canary, the same path as `CmdName` |
| `TimeType` | `0` selects the immediate path; type 1 is a different timer path |
| Other fields | `ViewName`, `Timer`, `TimeCount`, `CmdFlag`; the immediate branch did not inspect `CmdFlag` |
| Execution | Mode 0 reached `/bin/sh -c CmdInfo` through the native process controller |
| Timing | Approximately eight seconds of normal synchronous polling in the examined implementation |
| Timeout cleanup | Could execute `killall` on the command's first token |
| Repeat behavior | One attempt per consumer invocation; no once-ever or automatic respawn guarantee |

The experiment used a conservatively bounded command name of at most 31 ASCII bytes plus its terminating NUL. This bound came from the originating view layout, rather than the larger limits of a downstream API. A unique executable path avoids turning a generic shell name into the timeout cleanup target. The launcher must return promptly and tolerate repeated invocation.

The consumer used `dbAPILstView` / `dbAPIGetView`, then `PcStartProgram`. Those function names describe the observed native contract; they are not a substitute for a working command-line DB adapter. The [reproduction checklist](REPRODUCING.md#native-startup-canary) records the missing adapter/save details explicitly.

The earlier SNTP/Sysdiag attempts help explain why this mattered. Writable text reaches different consumers, with different execution semantics. One consumer's use of direct process execution did not provide shell expansion; another consumer exposed a usable startup path. The database field and the code reading it had to be considered together.

## Native supervision and startup order

Three different failures had different causes:

| Observation | Mechanism identified in the investigation | Measured follow-up |
| --- | --- | --- |
| Initial Dropbear build failed | Non-multiuser build mode did not match the measured kernel | Corrected build authenticated |
| Persistent SSH failed during startup | Race with native firewall initialization | Delayed startup resolved the observed race |
| Ordinary OMCI stopping caused reboot behavior | Native process-controller supervision | Bounded retirement accounted for restart registration and exception state |

These distinctions mattered because a binary fix could not resolve firewall ordering, and stopping a supervised process did not behave like stopping an unmanaged process. Each correction required its own reboot check. Idle resource observations did not qualify behavior under Internet load.

Userspace Tailscale provided another independent boundary: running without TUN could establish a maintenance path, while DHCP, NAT, Ethernet forwarding, and isolation still needed separate tests.

## Forwarding isolation was a state-transition problem

The control-plane state and the packet path disagreed. A WAN-role flag and Linux bridge membership changes did not stop the observed marker leakage. Temporary `nbif0` detachment blocked the measured traffic, but a subsequent physical carrier event caused native firmware to restore LAN forwarding.

The experiment needed both a packet test and a transition test:

| Test | What it established |
| --- | --- |
| Role and bridge readback | The visible configuration state |
| Marker traffic with positive controls | Whether the chosen traffic crossed the tested boundary |
| Offline DHCP and bounded IP check | Temporary lease/IP behavior |
| Physical carrier loss and reconnection | Whether native lifecycle handling preserved the boundary |

A stable runtime snapshot was insufficient for the intended gateway. The physical-link test exposed firmware behavior that the earlier packet snapshot had not exercised.

## Recovery evidence had separate limits

A logical backup, switching between healthy firmware banks, and receiving early UART output were useful observations. They established different capabilities. The investigation did not demonstrate damaged-image restoration or an interactive serial recovery console. Neighboring-model pin diagrams did not substitute for electrical measurements on the exact board.

These recovery limits constrained the conclusions that could be drawn from the maintenance and configuration experiments.

## Upstream type-6 parameters are seed strings

At the pinned upstream baseline, `examples/decode.py` requires both `--key` and `--iv` for type 6. `CBCXcryptor.set_key` treats them as text inputs, derives the AES key using SHA-256 of the key string, and takes the first 16 bytes of SHA-256 of the IV string for the CBC IV. These are the inspected utility's semantics; they do not establish the G1611's seed strings or native derivation.

After decryption, the type-6 decoder checks an inner payload header before the main path decompresses the content. A failure can therefore concern the supplied parameters, the container, its inner header, or compression. It cannot be diagnosed solely from the native label "type 6".

This behavior is visible in the pinned [decoder](https://github.com/mkst/zte-config-utility/blob/e3a1a7f56c2df26a02f52f72b16173d0702d9ad0/examples/decode.py) and [CBC implementation](https://github.com/mkst/zte-config-utility/blob/e3a1a7f56c2df26a02f52f72b16173d0702d9ad0/zcu/xcryptors.py). The essential semantics are stated here so this explanation remains useful if those links become unavailable.
