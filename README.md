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
| `20261111T0504_v16_pin_addendum.json` | `21da4bf5e891e3e14ec23dd3fd67288345b82a0d0cd54a2b8a6a38f30ccbc12e` | v16 pin addendum: hashes of the code written after the v16 freeze (2026-11-11 05:04 UTC) |
| `20261111T0505_v16_generator_choice.json` | `db66b2b86917c9fdae19cc26f9062f0721aa0b47a01552ca6edb17282ba87495` | Frozen generator choice, made mechanically on development data by the v16 selection rule: the v15 generator is kept (the retrained generator's selection metric, 0.4331, is not greater than 0.4367) |
| `20261111T0640_v16_generation_record.json` | `60d34f0ddb62af0b87ca02f13928ab3edfef467ed11419075b865e11b6c29aae` | Seal of the 45 v16 test generation directories (hashes of every generated-report file), made before any v16 score was computed |
| `20261111T0640_v16_decision.json` | `934bb17ebf9e93aff6fec97efaa88c1ea80d01b5ae54a1be2d4d6c1c5c27ca13` | Outcome of the v16 pre-declared decision: met on none of the three benchmarks |

## Status before the second look

The v16 pin addendum and the frozen generator choice above were committed here before any v16 test read.

The results of both looks are reported in full in the paper, whatever their direction.
