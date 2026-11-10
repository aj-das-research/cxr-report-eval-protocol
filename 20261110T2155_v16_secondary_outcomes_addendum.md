# v16 secondary-outcomes addendum (declared before the v16 test look; descriptive only)

v16 protocol: `docs/reports/S11/20261110T2140_s11_external_eval_protocol_v16.json` (sha256 f5e91838…). Declared on
2026-11-10, ~22:00 UTC. That is before the v16 pin addendum, before the generator choice, and before the v16 look reads any
test label or reference. The commit that adds this file is its timestamp.

These outcomes answer the two blind reviews of draft v15 (`docs/reviews/20261110_blind_review_{D,E}_v15.md`).
- Their main attack: on test, SWRD is scored only by CheXbert and SRR-BERT, the two labellers its repair consults.
- They also asked for grounded baselines on test and for the classifier's own label accuracy.

None of the outcomes below enters the claim, the gate or the selection. They change nothing in the v16 decision rule. Every one
is reported in full, whatever its value.

## S-H: held-out readers on the sealed test generations

- **Readers.** The two C5 readers, unchanged.
  - (a) The local LLM labeller: `swrd.eval.ctrl_llm_label`, frozen `configs/ctrl/llm_labeller_1.yaml` (sha256 59d8328b…),
    Qwen3.5-122B-A10B-GPTQ-Int4, run locally with vLLM.
  - (b) The rule-based CheXpert labeller: `tools/ctrl_c5_rule_label.py`, chexpert-labeler 44ddeb3 + NegBio 073199e.
- **Inputs.** Every sealed arm dir of every benchmark: SWRD_v16, SWRD_v15_headline, the template, the four baselines and the
  four grounded baselines (S-G). On MIMIC, the reference reports are added too.
- **Reference.**
  - CheXpert test and VinDr test: the benchmark's radiologist labels.
  - MIMIC test: the same held-out reader also reads the FINDINGS reference. Primary reference = positive or uncertain;
    positive-only reference = positive; normal = no primary-positive finding outside the claim excludes.
- **Statistics.** `swrd.eval.secondary_test score`. This is a copy of final_suite's metric path:
  - macro over the benchmark's frozen CheXbert macro list, with undefined F1 counted as 0;
  - macro F1 with uncertain counted as positive, positive-only macro F1, and the normal false-claim rate under both
    readings;
  - seed means;
  - SWRD_v16 minus every other arm, on the shared patient-cluster replicates (B = 2000, seed of swrd.eval.score), with 95%
    intervals and one-sided p.
- **Label integrity.** Each label file's identity must carry the sha256 of the sealed preds.jsonl. Labels are produced only
  after the seal. A missing label row is an empty report.

## S-G: grounded baselines on test

- **What is applied.** SWRD's grounding module: the v15 constrained repair (`swrd.eval.constrained repair`, v2.2.1
  sentences, device heads). It is applied to each baseline's reused, sealed-by-hash v15 test generation, with the v15 look's
  Wilson-90 test tags and device tags. That is the dev C1 procedure, with the headline's own test inputs.
- **Arms and scoring.** The arms are `<baseline>_grounded`. They are labelled and sealed with the other arms, and scored by
  final_seal (CheXbert, SRR-BERT) and by S-H.
- **Fusion.** The fusion rule needs k = 4 samples, and the baselines have one generation each, so S-G uses repair without
  fusion. This matches C1.

## S-C: classifier only (no text)

- **Definition.** The classifier's thresholded statuses (`swrd.sota.tags.statuses`, tag_block_1, on the v15 Wilson-90 test
  tag inputs), scored as a label vector against the benchmark's CheXbert-tier reference. No text and no labeller are involved.
  Primary = present or uncertain; positive-only = present.
- **Purpose.** It is the classifier's accuracy ceiling, and the row against which the generated reports are read.
- **Dev check.** Run on 2026-11-10 against the dry-run protocol: positive-only macro F1 is 0.745 on CheXpert val, 0.481 on
  VinDr dev and 0.371 on ReXGradient val. These are dev numbers and are reported as such.

## Runbook additions (`tools/v16_test_look.sh`, all through `tools/v16_guard.py`)

Each stage below runs at a fixed point in the v16 runbook:

| Stage | Runs |
|---|---|
| `grounded-launch` | after `launch`, before `seal` |
| `heldout-launch`, `heldout-score`, `classifier-score` | after the record is committed, with RECORD_SHA |

The files they run are pinned in the v16 pin addendum:
- role `fusion`: `jobs/sota_constrained.sh`;
- role `rq`: `src/swrd/eval/secondary_test.py`, `src/swrd/eval/ctrl_llm_label.py`, `configs/ctrl/llm_labeller_1.yaml`,
  `jobs/ctrl_llm_label.sh`, `tools/ctrl_c5_rule_label.py` and `jobs/v16_rule_label.sh`.

The dev dry run exercises every stage.

## Disclosures

1. These outcomes were added after the v15 look, after the dev controls (C1–C5), and after the blind reviews. They were added
   before any v16 test read. The direction of the S-H answer is unknown. On dev, SWRD's lead over CheXagent vanished under both
   readers (S13 C5).
2. The held-out readers are imperfect. Against CheXpert val radiologist labels, macro F1 is 0.56 for the LLM reader and 0.54
   for the rule reader, and kappa is 0.41 and 0.36. The rule reader produced CheXbert's training labels, so it is the less
   independent of the two.
3. S-G and S-C reuse the v15 look's test tags and generations by hash. They add no new generation by a baseline.
