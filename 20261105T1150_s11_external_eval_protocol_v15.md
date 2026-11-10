# S11 external evaluation protocol v15: FROZEN (worker-EV8, 2026-11-05)

The machine-readable protocol is `docs/reports/S11/20261105T1150_s11_external_eval_protocol_v15.json`; its sha256 is recorded in
channel/95_work_log.md. This text explains it. It extends v14 (sha256 82cd21e0…) and does not change it. It supersedes drafts
r1, r2 and r3. The audit was docs/reports/consult/20261104T1921Z_v15_draft_audit.md (GO WITH FIXES; must-fixes closed in
r2/r3). Decisions are in channel/04, 2026-11-05 ~11:50 and the rulings ~12:05.

**Single look.** Every test step runs once, only through `tools/v15_guard.py`. At the freeze, no test output exists.

## Benchmarks

| Benchmark | n / patients | CheXbert macro | SRR-BERT macro | Status |
|---|---|---|---|---|
| CheXpert test | 500 / 500 | 9 (v14 list) | 7 | second look |
| VinDr-CXR test | 3,000 / 3,000 | 7 (v14 list) | 7 | second look |
| MIMIC-CXR official test | 2,210 / 285 | 13 | 11 | fresh |

- Finding lists use the suite-v2 rule (see the JSON `benchmarks` block).
- The MIMIC reference is the FINDINGS section, extracted with the official MIT-LCP parser. 831 studies have no findings
  section and are excluded.

## Headline arm (selected on dev by the pre-declared rule)

**SWRD_headline** is the following chain:
- the tagged Qwen3.5-9B generator (R-all v8 + status_prob classifier-tag block), seeds 29/43/71, each at checkpoint-2000
  (selected by the frozen rule);
- test tags from `e2e_raddino_merged_tta_site_devices` with site Wilson-90 thresholds
  (configs_v5/e2e_raddino_merged_tta__site_wilson90__t22) and the six device heads;
- classifier-consistency re-ranking, k = 4 (configs/sota/rerank_1.yaml);
- SWRD-constrained repair with template sentences v2.2.1.

**Selection rule (governing).** Rank the candidates by the 3-seed mean, over VinDr dev + ReXGradient val, of the mean of
CheXbert and SRR-BERT positive-only macro. A candidate is eligible only if its dev FCR is within the cap. Ties go to the
generator.

**Cap ruling.** The cap is the comparator's exact dev FCR, LLaVA-Rad 5/26 = 0.19231 on CheXpert val; "<=" includes
equality.

Winner tables (all kept in the JSON):

| Reading | FCR cap on CheXbert only | Per-labeller FCR caps |
|---|---|---|
| VinDr + ReX metric (governing) | generator rr+con Wilson-90, 0.431 | same, 0.431 |
| 3-set metric (amendment 4 transcription error; disclosed) | template Wilson-90, 0.518 | same, 0.518 |
| VinDr + ReX metric, rounded cap 0.1923 (disclosed) | generator rr+con LSO, 0.421 | n/a |

**Gate** (per benchmark; headline dev CheXbert ≥ comparator + 0.05 AND SRR-BERT ≥ comparator; suite v2). All three pass:

| Benchmark (dev set) | Headline CheXbert / SRR-BERT | LLaVA-Rad CheXbert / SRR-BERT |
|---|---|---|
| CheXpert test (CheXpert val) | 0.732 / 0.668 | 0.553 / 0.563 |
| VinDr test (VinDr dev) | 0.468 / 0.463 | 0.359 / 0.398 |
| MIMIC test (ReXGradient val) | 0.422 / 0.460 | 0.310 / 0.374 |

## Claim rule (3 decisions, one per benchmark)

- **Comparator:** LLaVA-Rad on all three benchmarks. Each choice was fixed beforehand, from that benchmark's own dev split.
- **Win on a benchmark:** both conditions must hold.
  - Holm (m = 3, one-sided α = 0.025) rejects p_b = max(p_CheXbert, p_SRR, p_FCR).
  - The point gain in CheXbert macro is at least +0.03.
- **FCR non-inferiority:** the upper 95% bound of the paired ΔFCR is at most +0.03, under both labellers.
- **Statistics:**
  - paired patient-cluster bootstrap of the seed-mean difference (B 2000, seed 20261028);
  - undefined F1 counts as 0, and the cells are counted;
  - a missing prediction counts as an empty report.
- Everything else is descriptive.
- **Scorer:** src/swrd/eval/final_suite.py (`score --final`, then `decide`).

## Reported arms (descriptive, same single look)

- **Generator variants:** SWRD_plain, SWRD_reranked and SWRD_constrained (Wilson-90 tags, 3 seeds), and SWRD_lso (the
  headline pipeline with LSO tags, 3 seeds).
- **Templates:** template_wilson90 and template_lso (classifier → template v2.2.1).
- **noVinDr_control:** the novindr classifier with vindrlso thresholds → template v2.1.
- **v8_untagged:** 3 seeds, greedy.
- **R_all_4B:** the v14 student. Its CheXpert and VinDr test generations are reused and hashed.
- **Baselines:** the 4 baselines. Their CheXpert and VinDr test generations are the v14 E3 ones, reused and hashed. MIMIC
  is generated once with the frozen harness.
- **Lever E is not an arm.** It is reported as a development ablation.

## Runbook

Every command is prefixed with `python3 tools/v15_guard.py --v15 <json> --v15-sha256 <sha> --role <role> --`.

1. **cls_tags.** Run `jobs/v15_test_tags.sh P H RUN CFG DEVICE_RUN NAME OUT MIMIC_SFT` once for each configuration:
   - Wilson-90 (headline): finetune_raddino_merged, configs_v5/…site_wilson90__t22, device_heads_e2e;
   - LSO: configs_v5/…lso__t22, same run and device run;
   - no-VinDr: finetune_raddino_merged_novindr, configs_v4/…novindr_tta__site_vindrlso, device run "none".
2. **tag_inputs.**
   - `python -m swrd.sota.test_template tag-source --tags <predict-folds parquet> --out <src.parquet> --device-out <devices.parquet> --v15 … --v15-sha256 …`
   - then `python -m swrd.sota.tags dev-input --sft <benchmark sft> --tags <src.parquet> --variant status_prob --out <input dir>`
3. **template.** `python -m swrd.sota.test_template render --tags <predict-folds parquet> --sentences <v2.2.1 or v2.1> --out <dir> --v15 … --v15-sha256 …`
4. **generate.**
   - Tagged arms: jobs/generate.sh + generate_merge.sh on the tagged input.
   - Untagged v8 and R-all 4B (MIMIC only): on the benchmark sft.
   - Adapter: the selected checkpoint.
5. **sample_rerank.** jobs/generate_sample.sh + generate_sample_merge.sh (rerank_1.yaml), then `jobs/sota_rerank.sh select --samples … --tags <input dir> --out … --rerank-config configs/sota/rerank_1.yaml`.
6. **constrained.** `jobs/sota_constrained.sh repair --gen <plain or rr> --tags <input dir> --sentences configs/template_sentences_v2_2_1.yaml --device-run outputs/sota_cls/20261104T0830Z/device_heads_e2e --device-tags <devices.parquet> --out …`
7. **e3.** On MIMIC test only, run `jobs/e3_baseline_generate.sh <model> <MIMIC sft> <out>` for each of the 4 models.
8. **label.** jobs/sota_label_cpu.sh, which runs CheXbert under the v14 pin, plus raw4 and SRR-BERT, on every generation dir.
9. **Generation record.** After every generation has finished and before any scoring, write the sealed record once:
   {v15_sha256, generations{benchmark{arm{seed: preds sha256}}}}. Then log its sha256.
10. **score.** Run `final_suite score --final --v15 … --generation-record … --benchmark b --arm …` for each benchmark,
    then `final_suite decide --final …`.

**Reruns.** A rerun is allowed only after an infrastructure failure. It must cover the whole arm × benchmark, with an
identical config, and the job ids are logged.

**Decision rule for tags (all arms).** A tag is on when the stored float32 probability, promoted to float64, is at or
above the float64 threshold. This rule is identical to the dev selection inputs. In the dev dry run on CheXpert val, it
disagreed with predict-folds' internal tag columns in 4 of 200 × 13 cells, all at the threshold boundary; these
disagreements are counted.

## Disclosures

1. CheXpert test and VinDr test were scored once before, for the v14 arms (2026-11-02, CheXbert only). Channels through
   which v14 could have informed v15:
   - the choice of levers;
   - the comparator, which is conservative (LLaVA-Rad was v14's best baseline on both);
   - the SRR list, which cannot be test-informed (no SRR-BERT was run on test);
   - the CheXpert-domain thresholds, guarded by the reported LSO arms;
   - the margins.
2. The selection metric line in amendment 4 ("mean over 3 dev sets") is a carried-over transcription error. The governing
   rule is main's 19:45 declaration. Under the 3-set reading the template would be selected (0.518 vs 0.512). The paper
   reports both readings.
3. The winner depends on reading the cap exactly (5/26): with the rounded value 0.1923, the LSO variant wins (0.421). The
   headline's CheXpert-val FCR equals the cap; one normal study decides eligibility.
4. Two FCR-cap readings exist: CheXbert only, or each labeller against its own comparator FCR. Both select the same
   winner.
5. **Device-type heads and template sentences v2.2.1:**
   - they were motivated by the suite-v2 dev finding (Support Devices F1 0.000 under SRR-BERT);
   - the sentences took two frozen, invented-only rounds;
   - Wilson-90 was reached after the point cap failed the CheXpert-val FCR budget on dev.
6. **RAD-DINO exposure:**
   - RAD-DINO's self-supervised pretraining saw CheXpert train and valid. That makes our CheXpert-val dev set
     image-exposed, and SWRD in-domain on CheXpert test.
   - Its MIMIC images are exactly the official train split, with no test images.
   - MAIRA-2's encoder has the same public composition.
7. **VinDr-train labels** give the classifier in-domain supervision for VinDr test. The no-VinDr control is reported
   alongside.
8. **The headline's margin over the classifier → template arm is small** (+0.007 on the dev metric, within one seed SD).
   Most of the gain over the baselines comes from the classifier.
9. **MIMIC baselines** run image-only on a single frontal view. Published numbers are shown alongside, with the
   differences stated.
