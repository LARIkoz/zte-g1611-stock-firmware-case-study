# Root access was only the beginning: investigating the ZTE G1611

The goal was to turn an owned stock AIS ZTE G1611 into a remotely maintainable gateway for an isolated Ethernet client. The investigation eventually demonstrated persistent maintenance services, but the gateway proposal failed a physical-link test. The useful result is the distinction between access, persistence, and forwarding isolation: each required its own measurement.

This account selects the technical turning points from work completed by 25 October 2026. Private working records support the account but are not attached.

## Device and software under investigation

These values identify the examined platform and software observations. They are product/build identifiers, not the unit's serial number or network identity.

| Item | Recorded value | Evidence and qualification |
| --- | --- | --- |
| Product / operator variant | AIS ZTE G1611 | Exact-device bootstrap observations |
| Reported hardware version | G1611 V9.1.41 | Recorded device inventory |
| Main PCB marking | `73013F6600P_220700` | Read from the supplied component-side photograph |
| Antenna-board marking | F6600P V9.1 | Photograph observation; does not establish interchangeability with every F6600P |
| Main SoC / CPU | ZTE ZX279128S; dual Cortex-A9, ARMv7 | Exact-device bootstrap inventory |
| Floating-point/SIMD capabilities | VFP/NEON not advertised in the retained CPU inventory | The executed userspace binary used a conservative ARM target; do not infer all ARMv7 binaries will run |
| Wi-Fi implementation | `mt7916.ko` driver; MT7916 driver-family analysis | Driver evidence; individual radio-package markings are not independently certified here |
| RAM | Boot parameter 256 MiB; kernel `MemTotal` 223,760 KiB | Installed/boot memory and memory available to Linux are distinct figures |
| Flash | 128 MiB logical whole-device MTD read | A logical view; physical NAND chip vendor/part and OOB-complete geometry are not established here |
| Reported firmware string | `G1611_PON_1.1` | Recorded bootstrap version field |
| Firmware extension / build date | `07e8P4N10396`; 2024-09-18 | Additional recorded version fields, retained separately |
| Reported boot-software string | `V9.0.10P10N16` | Recorded bootstrap field; not substituted for the other firmware strings |
| Retained offline report | `V9.0.10P4N1` | Its equivalence to the running software is unresolved |
| Kernel / userspace build environment | Linux 4.1.25; Buildroot 2017.05; GCC 4.9.4; glibc 2.26 | Retained bootstrap inventory |
| Stock tools | BusyBox 1.17.2; curl 7.59.0 | Tool availability constrained the experiments |
| Tailscale feasibility binary | Official 1.104.1 ARM, GOARM5 | Executed in the initial userspace feasibility test; later custom service revisions have separate provenance |

The PCB marking is not a verified UART pinout. Exact electrical levels and interactive recovery remained unproved. Related F6600P/F6600R boards share some names and platform features, but their component populations and behavior cannot be transferred to this unit automatically.

The bootstrap report supplies the system inventory; the later PCB report supplies the photographed board marking. Firmware-analysis results below retain their separate offline-image qualification. This combines recorded observations, without claiming every field was collected simultaneously or independently remeasured for this publication.

For another unit, begin with the [reproduction and evidence checklist](REPRODUCING.md). This article explains the outcome; that checklist distinguishes repeatable read-only checks from steps whose complete adapters or fixtures have not been published.

## A root shell did not provide a writable firmware image

A factory-service path provided temporary root access. Logical flash and partition backups were retained and transfer-checked. A round trip between two healthy firmware banks also succeeded.

Those were useful starting points. They did not establish recovery from a damaged image, a physical NAND backup including OOB data, or safe operation of a newly modified firmware bank.

The active root filesystem remained read-only. Remount commands sometimes returned success, while mount-state readback and attempted file creation contradicted the intended result. A proposed startup-file approach therefore stopped at a concrete filesystem limitation.

The same distinction later appeared with storage: a successful remount command left the card writable. A normal unmount followed by a fresh read-only mount produced the intended flags. Exit status alone was an unreliable acceptance test for both operations.

## Persistence depended on the consumer of saved configuration

Writable configuration offered another route. Early SNTP and Sysdiag canaries failed and were reverted. Analysis of one consumer showed direct process execution rather than shell interpretation, explaining why shell-looking configuration text did not produce the hoped-for behavior.

A later native startup consumer of `CMDPARAMINFO` provided a different mechanism. After native initialization, it read a saved database view and passed an immediate-mode command to the process controller. A short executable path could reach a separately writable launcher without changing the read-only root filesystem.

The experiment first checked a benign canary after a distinct boot, then checked a background launcher after another boot. The launcher needed to return promptly because the synchronous native execution path had timeout and cleanup behavior.

This superseded the broad early assumption that no native startup path was available. It did not turn the failed SNTP/Sysdiag experiments into successes: they exercised different consumers.

## Starting a process and preserving a working service were separate tasks

An official ARM Tailscale binary ran in userspace mode without TUN. Later reboot checks demonstrated retained enrollment and authenticated reachability. That established a particular maintenance path across reboot; it did not establish a working Ethernet forwarding gateway.

Restricted SSH required two separate corrections. The first Dropbear build used a non-multiuser build mode incompatible with the measured kernel. After the corrected build authenticated, persistent startup exposed a race with native firewall initialization. Delayed startup resolved the observed ordering problem.

Service retirement also interacted with native supervision. Ordinary OMCI stopping triggered process-controller reboot behavior. A later bounded experiment accounted for restart registration and the global exception state, then checked that the selected entries remained absent after reboot. This was a measured service-reduction result, without an Internet-load stability claim.

## A native configuration restore was useful without being an external codec

The investigation rehearsed restoring an exact saved native configuration through two boots. Loader analysis distinguished the normal primary load from backup fallback.

The examined native configuration wrapper was classified as type 6, and native types 5/6 reached an AES-CBC decryption path. This is relevant to configuration tooling, but it leaves an important compatibility question: the native wrapper's type number has not been independently mapped to the utility's input-container classification.

Restoring an opaque saved file demonstrated that native restoration path. It did not demonstrate an external decoder, encoder, key derivation, or a compatible web-exported `config.bin`. The [mechanism note](MECHANISMS.md#native-configuration-loading) describes the evidence and remaining gaps.

## Packet tests exposed the decisive failure

Native WAN-role reporting and removal of `eth0` from the Linux LAN bridge did not prevent external marker broadcasts from reaching the maintenance LAN. The observed path included `pon0` and its untagged child `nbif0`. The evidence did not justify reducing the explanation to an ASIC-only bypass.

A controlled runtime change removed `nbif0` from `br0`. No WAN-phase markers crossed in the 1,185-frame test, while positive controls before and after the change each delivered 297 markers. Maintenance SSH remained available. An offline DHCP check then obtained a lease, and a bounded IP check passed three pings out of three.

At that point, the temporary arrangement looked promising. The next test interrupted physical carrier for about five seconds. Native firmware returned the port to LAN, and traffic again crossed into the maintenance network after reconnection. Temporary topology changes and associated fallbacks were restored or cancelled.

The proposed isolated gateway failed its cable-cycle acceptance criterion. There was no qualified production provider/client gateway or completed client migration at this cutoff.

## What is worth carrying forward

The investigation produced a reboot-tested maintenance path and evidence about native configuration/startup consumers. It also identified a forwarding arrangement that worked temporarily and failed during a normal physical event.

The configuration observations can support a focused upstream compatibility discussion. Generic type-6 decoding was already added by other contributors in [PR #204](https://github.com/mkst/zte-config-utility/pull/204); no G1611 support or contribution to that change is claimed here. A future support PR still needs a pinned-baseline reproduction and a valid synthetic fixture.
