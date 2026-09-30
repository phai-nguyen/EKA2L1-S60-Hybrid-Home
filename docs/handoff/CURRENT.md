# CURRENT — N-GAGE QD MACHINE1 PREP

**Ngày:** 2026-09-30
**Handoff chuẩn:** `docs/handoff/NEWCHAT-NGAGE-QD-MACHINE1-2026-09-30.md`

## Trạng thái hiện tại

### Product branch
- repo: `phai-nguyen/-EKA2L1-iOS-fixed`
- branch: `hybridhome-upstream1`
- HYBRIDHOME3 commit: `acb2775a652f44104c841a6ed08557801a3aadbe`
- build: `#36663115510` — **GREEN**
- VPL multi-file import: **DEVICE PASS**
- VPL no-crash fix: **DEVICE PASS**
- Vietnamese UI: working
- JIT/Dynarmic: OFF for ESign
- Bundle ID: `app.lavender1865.valley8348`

### New research direction
- full-machine target: **Nokia N-Gage QD RH-29**
- reason: simpler S60v1/EKA1 target than RM-356
- architecture inspiration: CloudpilotEmu hardware-oriented machine model
- keep Hybrid Home as product branch; machine work is separate research branch

### RH-29 ROM
- source upload: `Nokia_N-Gage_QD_RH-29.zip`
- generated: `SYM.ROM`
- firmware: **V 04.10 — 09-09-2004**
- ROM base: `0x50000000`
- ROM size: `18,284,544`
- MD5: `ba9ae3d584c42b24d44bf0f25ba5896f`
- SHA-256: `4824ee55086cade3415c7478ec54b9654658bfdd737e779995dfe0ef716ab70b`
- **NOT DEVICE-TESTED YET**

## Next

1. Test generated RH-29 `SYM.ROM` in current EKA2L1 iOS.
2. If loader/device/AppList works, create research branch `ngage-machine1`.
3. MACHINE1 first probe:
   - ARM920T-class execution;
   - ROM map;
   - RAM;
   - reset/restart vector;
   - unknown MMIO read/write trace.
4. Do not attempt LCD/Home until early boot execution is understood.

See the full handoff before doing any work:
`docs/handoff/NEWCHAT-NGAGE-QD-MACHINE1-2026-09-30.md`
