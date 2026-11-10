# Public registration: chest X-ray report generation evaluation protocol (second, disclosed test look)

This repository publicly timestamps a frozen evaluation protocol before the evaluation it governs reads any test label or
test reference. It holds protocol files and hashes only. It contains no data, no images, no reports, no per-study
records and no model weights.

## Files

| File | sha256 | Role |
|---|---|---|
| `20261105T1150_s11_external_eval_protocol_v15.json` | `bc4459635437df1d1ce56d9bd1db4abb489a76ac0fee88eb5a2d0ff00a4799fd` | First (confirmatory) test-look protocol (v15, frozen 2026-11-05) |
| `20261105T1150_s11_external_eval_protocol_v15.md` | `96158a0f1d7da8ad13a510eb3a723269fb0ec87f9ed5d3f36355b4aba1b6fb1a` | Its explanation |
| `20261110T2140_s11_external_eval_protocol_v16.json` | `f5e91838c80b16113a1e25352b32f34239fbaec76ca4f7dd2ca943b833132c83` | Second, disclosed test-look protocol (v16, frozen 2026-11-10 21:34 UTC) |
| `20261110T2140_s11_external_eval_protocol_v16.md` | `bf6392bbcd2516925ce8cc6833e7a980855203c82a5c4a74413d15d269887366` | Its explanation and disclosures |
| `20261110T2155_v16_secondary_outcomes_addendum.md` | `f1f901103b921a492942359a8c24294eac20ce6405b8d0dbd778b471695b81d0` | Pre-look secondary outcomes (descriptive): held-out labellers, grounded baselines, classifier only |
| `fusion_rule_1.json` | `dfa107df5c7b26c03791a58cfe5218baeaef7160419c29da3c145b59675ae4cb` | Frozen fusion decision rule (fitted on development data only) |
| `20261101T1745_s11_external_eval_protocol.json` | `82cd21e086c1824d24f8d1d5069053c1556bf0a99d525de8342798efbb6e5f0e` | Labelling protocol both looks use |

## What will be added before the second look

The following are committed here before any v16 test read:
- the v16 pin addendum (the hashes of the code written after the freeze);
- the frozen generator choice (made mechanically on development data).

The results of both looks are reported in full in the paper, whatever their direction.
