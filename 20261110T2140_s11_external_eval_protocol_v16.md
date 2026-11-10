# S11 external evaluation protocol v16: FROZEN (worker-V16, 2026-11-10 ~21:40 UTC). SECOND, DISCLOSED TEST LOOK

The machine-readable protocol is `docs/reports/S11/20261110T2140_s11_external_eval_protocol_v16.json`. Its sha256 is
recorded in channel/95_work_log.md and in the commit that adds it. This file explains it and holds the disclosures. v16 is a
copy-then-edit of v15 (sha256 bc4459635437df1d…). It was built by `tools/v16_freeze_protocol.py`, which refuses to run if any
fix-1 dev evaluation output exists.

**This is the second look at the three test sets.** The v15 look (2026-11-06) is and stays the confirmatory record: its claim
was not met on any benchmark (docs/reports/S11/20261106T1834_v15_test_results.md). Abhijit approved a system fix, a v16
protocol frozen on dev, and ONE disclosed second look. Both looks are reported, in full, side by side.

## What is unchanged from v15

- **Benchmarks, finding lists, references.** CheXpert test (500), VinDr-CXR test (3,000), MIMIC-CXR official test (2,210 /
  285 patients, FINDINGS section). The `benchmarks` block is copied byte-for-byte as JSON.
- **Claim rule (the `headline` block, copied; only `swrd_arm` renamed to SWRD_v16).**
  - Comparator: LLaVA-Rad on all three benchmarks.
  - A benchmark is won when Holm (m = 3, one-sided α = 0.025) rejects p_b = max(p_CheXbert, p_SRR, p_FCR-NI), AND the point
    gain in CheXbert macro is at least +0.03.
  - p_FCR-NI is the FCR non-inferiority p, taken under both labellers: the upper 95% bound of the paired ΔFCR must be at most
    +0.03.
  - Statistics: paired patient-cluster bootstrap of the seed-mean difference (B 2000, seed 20261028). An undefined F1 counts
    as 0, and those cells are counted. A missing prediction counts as an empty report.
  - Scorer: `swrd.eval.final_seal` (seal → score → decide), which wraps `swrd.eval.final_suite` unchanged.
- **Gate.** The v15 rule, applied to the selected arm on each benchmark's dev split (dev CheXbert macro ≥ LLaVA-Rad + 0.05
  AND SRR-BERT macro ≥ LLaVA-Rad). A failed gate makes that benchmark descriptive; Holm m stays 3.
- **Tags.** The v15 look's Wilson-90 classifier tags and device heads are reused by sha256. The classifier is not rerun.
- **Rerun rules.** Infrastructure failures only, rerun as a whole arm × benchmark with an identical config, job ids logged.

## What is new

### Headline arm SWRD_v16: the selected generator + the fix-2 fusion rule

The chain, applied per seed (29/43/71, checkpoint-2000):
1. Test inputs carry the v15 Wilson-90 tags, rendered with the candidate's tag block.
2. Draw k = 4 samples (configs/sota/rerank_1.yaml).
3. Run the rule-matcher reads of the 4 samples.
4. Compute fused statuses with the **simple** rule. A finding is stated iff the classifier tags it present AND (at least 2 of
   the 4 samples state it, OR p ≥ h_f). A classifier-present finding the rule rejects becomes uncertain.
5. Run `fusion select`: re-rank the samples against the fused statuses, tie-break on q.
6. Run `constrained_fused repair` (v2.2.1 sentences, device heads unchanged).

So the fusion rule sits after the samples and before the repair, as fix 2 specified
(docs/reports/S9/20261110T0852_fix2_fusion.md; rule outputs/eval/fix2_fusion_20261110T0640Z/frozen/fusion_rule_1.json,
sha256 dfa107df…). The test domain map is CheXpert → chexpert_val, VinDr → vindr_dev, MIMIC → rexgradient_val.

### Candidates (exactly two)

| | v15 generator | fix-1 generator |
|---|---|---|
| Runs | `v2_SOTA_9b_R_all_v8_tagse2e_status_prob_2000u_s{29,43,71}/checkpoint-2000` (tree hashes pinned) | `v2_SOTA_9b_R_all_v8_tagfaith_status_prob_2000u_s{29,43,71}/checkpoint-2000` (tree hashes go into the choice; training was running at freeze) |
| Tag block | tag-block-1 status_prob | tag-block-2 status_prob (present iff p ≥ threshold, else absent) |
| Test samples | the v15 look's k = 4 samples, reused by sha256 (generation reads no label or reference) | generated in the v16 look (generate_sample + merge, rerank_1) |
| Fusion rule (test) | the frozen per-seed fits `simple:swrd_s<seed>` in fusion_rule_1.json | the same fusion-rule-1 **procedure** refitted on this generator's dev samples: same module sha256, settings, dev tags, budgets, matcher and 2-fold split; its `rule_full.json` (the all-dev fit) is hashed into the choice |
| Dev selection rows | fix 2's out-of-fold simple rows (`swrd_s<seed>__simple_con`) | the refit's out-of-fold statuses → fusion select → fused repair → labels (as fix 2's run_arm.sh) |

### Pre-declared, mechanical dev selection

- **Metric.** The v15 governing selection metric: the mean over VinDr dev and ReXGradient val of mean(CheXbert, SRR-BERT
  positive-only macro F1), averaged over the 3 seeds per patient-bootstrap replicate. The point value is replicate 0
  (`tools/ctrl_paired_suite.py`, suite v2).
- **Eligibility (v15 cap convention).** The 3-seed-mean CheXbert normal false-claim rate must be at most the exact v15 cap on
  every dev set. The caps are CheXpert val 5/26 = 0.19231, VinDr dev 0.038552 (MAIRA-2) and ReXGradient val 0.135857
  (CheXagent). "≤" includes equality, with a tolerance of 1e-12. The per-labeller reading (SRR-BERT caps 0.19231 / 0.0404 /
  0.1675) is reported and does not govern.
- **Rule.** Select the fix-1 generator iff its metric is strictly greater than the v15 generator's AND the fix-1 generator is
  eligible. Otherwise keep the v15 generator.
- **Tie-break.** Equal float64 points keep the v15 generator.
- **Edge cases.**
  - If the v15 generator were ineligible, fix 1 is selected iff it is eligible.
  - If neither is eligible, the look does not start.
  - A missing fix-1 row means no choice is made and no fallback is used.
- **Output.** The choice is written to disk once, with its sha256, and the process STOPS. The choice file holds the chosen
  checkpoints' tree hashes, the fusion-rule files and their hashes, and the gate values. Starting the test look is a
  separate, explicit decision.

### Reported arms (descriptive)

- SWRD_v16 (headline).
- SWRD_v15_headline: the v15 headline generations, reused by hash. These are the same numbers as in the v15 look; they are
  reported only as paired context.
- template_wilson90 (reused).
- LLaVA-Rad (comparator), MAIRA-2, CheXagent and MedGemma, all with the v15 look's generations reused by hash.

### Secondary outcomes: report quality (non-decision)

- **Metrics.** BLEU-4, ROUGE-L, BERTScore, RadGraph F1 (partial / simple / complete), RaTEScore and GREEN. They run on every
  reported arm, on every seed separately and as the seed mean.
- **Code.** The exact metric code of the S13 C2 control: `tools/ctrl_rq_prep.py` → `jobs/ctrl_rq_score.sh` (swrd.eval.score,
  RadEval 2.2.2, findings track) and the `tools/ctrl_c2_aggregate.py` statistics.
- **GREEN scorer.** GREEN goes through `swrd.eval.score_green_vocab` for every arm. This is the same scorer; the only change
  is that GREEN's tokenizer is cut to its model's 32,000-row embedding. The S13 addendum explains why; token ids are checked
  to be identical on every prompt without the 64 added-token strings.
- **Benchmarks.** Report quality is defined only on **MIMIC test**: the reference is the FINDINGS section, the same text the
  reference labels come from. **CheXpert test has no reference reports** (CheXpert Plus covers train and valid only), so
  report quality is not defined there; this is declared, not filled in. VinDr has no reports.
- **Use.** These metrics never enter the claim, the gate or the selection. A metric that fails after the identical retry is
  reported as undefined.

## Runbook (only through `tools/v16_guard.py`, which checks the protocol, the pin addendum and the frozen choice)

1. **Choice.** Apply the selection rule to the dev rows (`tools/v16_select_generator.py`), write the frozen choice, and STOP.
2. **Inputs and samples.** Fix-1 only: build tag-block-2 inputs from the v15 tag sources, then generate the k = 4 samples
   (3 seeds × 3 benchmarks). For the v15 generator, its v15 samples are verified by hash.
3. **Fusion.** Per seed and benchmark: reads → apply the chosen rule (domain map) → select → fused repair.
4. **Label.** Run `jobs/sota_label_cpu.sh` (CheXbert v14 pin + raw4 + SRR-BERT).
5. **Seal.** Run `final_seal seal` once over every arm dir; the reused dirs must match the v16 pins. Commit the record's
   sha256 before any score.
6. **Score.** Run `final_seal score --final` per benchmark, then `final_seal decide --final` (Holm m = 3).
7. **Secondary.** On MIMIC, build the reference manifest, run the RQ prep + score for every arm dir, then aggregate.

The whole chain is `tools/v16_test_look.sh`. Its `dryrun-dev` mode runs the identical commands on the dev sets against a
dev-only copy of this protocol (benchmarks replaced by the suite-v2 dev sets). It never reads a test label or a test
reference.

## Disclosures

1. **Second look.** CheXpert test and VinDr test have now been scored three times (v14 CheXbert-only, v15, v16), and MIMIC
   test twice. v15 stays the confirmatory record. Holm (m = 3, 0.025) controls the error rate within one look, not across
   looks. As a descriptive sensitivity reading only, we also report whether each v16 win would hold at one-sided 0.0125
   (Bonferroni over the two looks).
2. **The fixes are test-informed in direction.** The v15 look failed on FCR non-inferiority on CheXpert and MIMIC. That
   failure prompted fix 2 (a rule that can only lower false claims) and the decision to evaluate fix 1. Every quantity of
   both fixes was fitted and selected on dev only. No v15 test number was used to fit h_f, to pick the rule family, to pick
   rho, or to choose between the candidates.
3. **The fusion rule was selected on dev among two families** (logistic vs simple). The simple rule's selection-metric gain
   over the v15 headline was not significant (+0.005 [−0.005, 0.015]). Its FCR reductions were significant on VinDr and
   ReX, and borderline on CheXpert val (n = 200, 26 normals).
4. **Fix-1 fusion refit.** For the fix-1 generator the fusion rule is the same procedure refitted on that generator's dev
   samples, because h_f depends on the generator's sample evidence g. Its dev selection value is out-of-fold, exactly as the
   v15 generator's 0.437.
5. **Repair verifiers.** The repair still uses both scoring labellers as sentence verifiers (unchanged from v15; disclosed
   there).
6. **MIMIC FCR is high for every model** (about 0.4-0.6 in v15) under the MIMIC Findings-section normal definition. That
   definition is unchanged here.
7. **GREEN tokenizer.** GREEN's stock tokenizer crashes on texts containing CheXagent's grounding markup. The fix is
   arm-independent, and its token-level identity is verified. The original crash and its diagnosis are in the S13 addendum.
8. **Reused test artifacts.** The v15 look's tags, tagged inputs, k = 4 samples and baseline generations were produced by the
   v15 look. They are reused by sha256 and not regenerated. Re-scoring reused arms reproduces v15 numbers and adds no new
   test information.
9. **All v15 disclosures carry over** (RAD-DINO exposure of CheXpert, VinDr-train classifier supervision, image-only MIMIC
   baselines, cap readings).
