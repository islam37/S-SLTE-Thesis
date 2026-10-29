# Research Progress Log

**Project:** Evaluating Semantic-Neighborhood Label Correction for LLM-Generated Noise in Text Classification  
**Candidate method:** Semantic Label Trustworthiness Estimation (S-SLTE)  
**Institution:** Metropolitan University, Sylhet — Department of Computer Science and Engineering  
**Researchers:** Islam Uddin (231-115-063); Md Yousuf Bin Nazib (231-115-064)  
**Target completion:** January 2027 (overall research); **supervisor-ready draft milestone:** 16 November 2026

> **Record-keeping policy:** Enter only work actually performed, observed results, decisions, blockers, and the next planned step. Do not report proposed methods as implemented or make performance claims before experiments. Record who performed or reviewed each task, and keep original reference labels outside the model-correction pipeline.

---

## 2026-10-29 — Stages 9–10: Frozen Dataset Partitions and Main LLM Annotation

**Research team:** Islam Uddin (231-115-063) and Md Yousuf Bin Nazib (231-115-064), Department of CSE, Metropolitan University, Sylhet.  
**Session participation and research contributions:** Both Islam Uddin and Md Yousuf Bin Nazib were present and actively contributed to the 29 October 2026 research session. Their collaborative work encompassed the dataset partition finalization and the initiation of main LLM annotation. Experimental procedures and outcomes are documented through the team's Colab notebooks, saved checkpoints, and GitHub commits; the recorded outputs were reviewed for this log without an additional independent rerun.  
**Research objective:** Establish reproducible, leakage-conscious text partitions and generate LLM-derived noisy labels for subsequent semantic-neighborhood correction experiments.

### 1. Stage 9 — Dataset partition freeze

The team followed the documented conservative policy for resolving possible cross-partition near-duplicates in the AG News sample. The procedure used word-level TF-IDF unigram/bigram cosine similarity at a screening threshold of **0.80**, comparing original texts and an **experimental candidate-cleaned view**. Candidate flags were treated as screening evidence—not automatically as confirmed duplicate articles.

- Resolved 17 previously unreviewed candidate relationships through conservative partition exclusions and replacement sampling; a full subsequent re-screen identified five additional candidate-cleaned flags, which were also conservatively resolved. No additional human adjudication is claimed.
- Reconstructed a final active corpus of **19,921** records in five disjoint partitions: **2,000 clean anchor**, **1,800 development**, **500 pilot**, **13,621 main training**, and **2,000 correction holdout**.
- The final screening reported **zero unresolved candidate pairs** at the selected threshold, **zero normalized exact-text duplicate records**, and **zero surviving previously adjudicated duplicate edges**. Of the remaining similarity flags, **17 raw-text** and **3 candidate-cleaned** pairs were all previously classified as **F (different)** or **R (related but distinct)**; none remained classified **D (duplicate)**.
- Versioned partition identifiers and original-text fingerprints were saved privately as `configs/holdout_partition_manifest_v1.json`. The notebook reported `NEW_MANIFEST_SAVED`, followed by read-back verification. The original source CSVs were unchanged, and reference labels were not accessed in this audit.
- The manifest recorded **26 holdout replacements** and **49 training exclusions** relative to the prior candidate populations. Frozen ID SHA-256 digests:

| Frozen partition | Records | SHA-256 of record IDs |
| --- | ---: | --- |
| Correction holdout | 2,000 | `0c529eaddf7abfd977dcad8a1627bab1369460d5846370fe64304600506609c4` |
| Main training | 13,621 | `ce13626940296ed9158f4dc116580987f404114f66d42fdc6dcc87acf7c62625` |

**Reproducibility evidence:** [`notebooks/08_boilerplate_contamination_audit.ipynb`](../notebooks/08_boilerplate_contamination_audit.ipynb), committed as [`a525df43474361aafba2b7aae50a2dc4ad59f31e`](https://github.com/islam37/S-SLTE-Thesis/commit/a525df43474361aafba2b7aae50a2dc4ad59f31e). Partition IDs are considered **v1 frozen**; source-text cleaning was not adopted as a universal transformation. The private manifest, including record IDs, should not be pushed to the public repository.

### 2. Stage 10 — Revision-pinned main LLM annotation

A new Colab notebook validated **15,621 text-only annotation inputs** against the frozen ID and raw-text fingerprints: **13,621 main-training** and **2,000 holdout** examples. The annotation input excluded reference labels. The same model and prompt specification as the previously validated 500-item pilot were retained.

| Parameter | Verified configuration |
| --- | --- |
| Annotator | `meta-llama/Llama-3.1-8B-Instruct` |
| Model revision | `0e9e39f249a16976918f6564b8830bc894c89659` |
| Prompt SHA-256 | `77a8d266f6f9d3e2dcdbeb906182e0d64f0836190fe84617a638d3ab3c5ed794` |
| Decoding | `do_sample=False`, `max_new_tokens=32` |
| Quantization and compute | 4-bit NF4; NVIDIA Tesla T4 (5.32 GB allocated VRAM upon loading) |
| Reported library versions | `transformers==5.20.1`, `bitsandbytes==0.50.2` |
| Checkpoint policy | Individual JSON records, text SHA-256, model/prompt verification, and atomic writes; original text preserved |

**Observed execution results:**

1. An eight-record smoke test (four training; four holdout) generated **8/8 JSON-valid responses**, persisted privately as `outputs/llm_main_v1/smoke_test_v1.json`.
2. The first 100-record production batch yielded **100/100 JSON-valid responses** (92 new inferences and eight verified smoke-test reuses), with **zero invalid outputs** and a reported execution duration of **87.91 seconds**.
3. The next checkpointed run generated **2,000 new responses**, yielding **2,100 saved record checkpoints in total** out of **15,621 expected** (**13.44%**). The final output reported **13,521 remaining**; **zero invalid outputs encountered in that run**; **50 previously saved checkpoints reverified within the run**; and **1,483.19 seconds** of execution time. Existing checkpoints were retained and only missing record predictions were generated.
4. The Colab notebook was committed to [`notebooks/09_main_llm_annotation.ipynb`](../notebooks/09_main_llm_annotation.ipynb) in commit [`d11a962f054b59b0e11cf69b4e2ad9a1ce4cbbda`](https://github.com/islam37/S-SLTE-Thesis/commit/d11a962f054b59b0e11cf69b4e2ad9a1ce4cbbda). Raw per-record prediction JSON files remain in restricted Google Drive under `outputs/llm_main_v1/records/` and were **not** committed.

### 3. Scientific interpretation and methodological safeguards

- **JSON-format validity is not label accuracy.** No S-SLTE detection, label correction, baseline comparison, downstream classifier improvement, or statistical significance result is claimed at this stage.
- TF-IDF screening does **not** guarantee the absence of all semantic duplicates or all possible leakage. Conservative removal of unadjudicated similar pairs may exclude genuinely distinct examples. The raw-versus-candidate-cleaned distinction must be preserved in reporting.
- The partition manifest freezes the **record IDs and raw-text fingerprints** for v1; later substantive changes require explicit versioning and documentation rather than silent replacement.
- Reference labels were not supplied to LLM annotation. Protected holdout reference labels, the retrospective pilot reference labels, and the official AG News test split must remain outside any annotation or correction-parameter selection procedure.
- The project supervisor may review the methodology separately; this is not recorded as having been approved. The immediate experimental workflow need not be suspended solely because supervisor consultation has not yet occurred.

### 4. Session handoff and next experimental milestone

**Current validated state:** Stage 9 partition v1 frozen. Stage 10 annotation is **in progress**, with **2,100/15,621** prediction checkpoints saved according to the committed Colab output. Preserve existing checkpoint directories and the original manifest.

**Next session:** Reconnect to Colab/T4, verify the frozen manifests and pinned model, and rerun the same resume-safe annotation cell. It must check existing record-level JSON files and generate only missing predictions. Once all **15,621** checkpoints exist, independently audit completeness, ID/text correspondence, model revision, prompt hash, decoding parameters, parser validity, and output distribution before consolidating the noisy-label table and proceeding to S-SLTE training, baselines, and ablation studies.

**Draft milestone:** Prepare a reproducible, supervisor-ready thesis draft by **16 November 2026**; the earlier **January 2027** target remains the broader research-completion horizon unless formally revised.

---

## 2026-10-27 — Stage 8: Boilerplate Contamination and Draft Leakage Resolution

**Researcher/session execution:** Islam Uddin.  
**Verified notebook:** [`08_boilerplate_contamination_audit.ipynb`](../notebooks/08_boilerplate_contamination_audit.ipynb), committed as [`1a19e3b1d6327d723918dae88929164a0109051b`](https://github.com/islam37/S-SLTE-Thesis/commit/1a19e3b1d6327d723918dae88929164a0109051b). The committed code and saved outputs were inspected; the Colab experiments were **not independently rerun**.

### Completed analysis and observed results
- Audited five text-only AG News partitions: **2,000 anchor, 1,800 development, 500 pilot, 2,000 proposed holdout, and 13,700 pre-deduplication training candidates**. Identified **18** advertisement-keyword matches, including **12** records with structured advertisement tails. An in-memory, experimental prefix-preserving tail-removal rule was evaluated without modifying the source texts; six non-structured uses of advertising terminology were left unchanged.
- Compared raw and candidate-cleaned TF-IDF near-duplicate screens (word unigrams/bigrams, five neighbors per holdout, cosine threshold **0.85**). Flagged pairs decreased from **63 external + 8 internal (71 total)** to **54 external + 2 internal (56 total)**; **15** pairs disappeared and **no new pairs** crossed the threshold. All **15 disappearing pairs** matched previously recorded working human decisions of **F (different)**. One pair disappeared following corpus-level vectorizer refitting although neither article text changed; this is not evidence of direct boilerplate removal for that pair.
- Reconstructed and text-aligned **71/71 original review pair IDs**. Among the **56** remaining candidate pairs, the prior decisions were **53 duplicate (D), 2 different (F), and 1 related-distinct (R)**. The 53 D edges formed **51 connected components**: 35 holdout–training components, 14 holdout–protected-split components, and 2 internal-holdout components.
- Proposed a deterministic, **non-executing** resolution of these D relationships: **37 unique training exclusions** and **16 unique holdout replacements** (14 protected-split conflicts plus one replacement from each of two internal components). Proposed seed **20261027** selected 16 distinct replacement candidates. The in-memory proposal held **2,000 holdout** and **13,643 training** records. Draft holdout-ID SHA-256: `66ab8197d1396a3dca7c308cad0e2f2b13eae7a8dbda9812b2805f4cd51b4963`; draft training-ID SHA-256: `bcb8e6000c915e6e2b163072b5d308397f318a065725a6d6e914759301bd2d9a`. These fingerprints are **not final dataset manifests**.
- Screened all 16 proposed replacement records against the **19,943-record active in-memory corpus** using both raw and candidate-cleaned TF-IDF texts. Found **zero pairs at cosine ≥0.80** and **zero exact-text collisions** in that active corpus.
- Subsequently screened the full **2,000-record proposed holdout** against the same **19,943-record active corpus**, using candidate-cleaned TF-IDF and exhaustive sparse pair enumeration at **cosine ≥0.80**. Found **21 flagged pairs**: 12 holdout–training, 3 holdout–anchor, 2 holdout–development, 2 holdout–pilot, and 2 within-holdout. No normalized cleaned-text exact duplicates or surviving prior D edges were reported.
- Reconciled those 21 pairs with the earlier review register: **4 previously reviewed** (3 F, 1 R) and **17 not previously reviewed**. The team elected to **propose**, rather than execute, a conservative exclusion policy for the 17 unreviewed candidates: **9 additional unique holdout-replacement candidates** (seven protected-partition edges plus two internal-holdout pairs) and **8 unique training-exclusion candidates**. These similarity flags remain **unadjudicated** and must **not** be described as confirmed duplicates.

### Methodological limitations and status
- The advertisement-tail transformation is a **screening candidate**, not an approved universal preprocessing rule; aggressive truncation could remove relevant text. TF-IDF similarity thresholds are candidate-detection heuristics and do not prove the absence of semantic duplicates or all contamination.
- The earlier 71-pair review decisions were preserved; reviewer independence was **not verified**. The 17 additional pairs were **not manually adjudicated**. The conservative policy is therefore a risk-reduction proposal with possible over-exclusion, not a human-validated duplicate classification.
- All partition adjustments, exclusions, and replacements above are **in-memory proposals only**. **No source dataset was modified, no replacement/exclusion manifest was finalized, no private reference labels were accessed for these audits, and the holdout remains NOT FROZEN.** No main S-SLTE or baseline experiment was performed.
- The notebook is available in the public repository. Its saved diagnostics include limited advertisement/article-text previews; do not publish private reviewer workbooks, source-reference labels, or identifiable confidential material.

**Next:** implement and validate the additional conservative exclusions and deterministic replacement sampling; re-screen the complete proposed holdout and evaluate whether remaining risk is acceptable; document the final leakage policy and immutable partition manifests **before** freezing the holdout or conducting main experiments.

---

## 2026-10-26 — Stage 7: Human Review Consensus and Draft Holdout Policy

**Researchers:** Islam Uddin and Md Yousuf Bin Nazib.  
**Notebook:** [`07_holdout_review_policy.ipynb`](../notebooks/07_holdout_review_policy.ipynb); verified in GitHub commit `a4d3bd4e2e664c6eedb46244cff358c7e158d043`. Saved notebook outputs and restricted Drive review files were inspected, not independently rerun.

### Completed work
- Reproduced the proposed **2,000-record holdout** and **13,696-record clean-training candidate view**, matching previous ID fingerprints. Source datasets were not modified.
- Screened **63 external** and **8 internal** near-duplicate candidate pairs, totaling **71 edges**, **56 connected components**, and **61 flagged holdout IDs**.
- Compared 71 reviewed pairs from two selected English-language reviewer spreadsheets stored on restricted Google Drive. **68/71 matched (95.77%); descriptive Cohen's kappa 0.8913**. The three disagreements were `PAIR-032`, `PAIR-043`, and `PAIR-057`.
- The researchers recorded discussion consensus for those three cases: `PAIR-032=R`, `PAIR-043=F`, `PAIR-057=D`. Final **working pair-level decisions: 53 D (duplicate), 17 F (different), 1 R (related-distinct)**.
- A non-executing candidate policy flagged **37 training-exclusion pairs (37 unique training IDs)**, **14 holdout-replacement pairs (14 unique holdout IDs)**, **2 internal-holdout pairs (4 unique holdout IDs)**, **17 no-action pairs**, and **1 related-distinct pair**. **No record was removed or replaced.**

### Limitations and safeguards
- Reviewer independence was **not verified**; prior reviewer-file history and AI-assisted exploratory assessment prohibit presenting observed agreement/kappa as validated independent inter-rater reliability. Consensus was reached through two-researcher discussion, not independent third-party adjudication.
- TF-IDF top-five screening at threshold 0.85 is not exhaustive. Repeated HTML/advertisement text created misleading similarity candidates; further cleaning and audit are pending.
- Restricted reference labels and the official test labels were not used in the review-policy notebook. The holdout remains **unfrozen**, with no real S-SLTE or baseline evaluation.
- The committed notebook has saved successful outputs, but its packet-generation cells are **not fully rerun-safe** (they may overwrite draft CSVs or stop if adjudication worksheets already exist). Fix in a later revision.
- The repository was **public** when inspected. Private reviewer workbooks and labels stay in restricted Drive; do not push them to GitHub. No common token signature was found in the examined newly committed notebook.

**Next:** audit advertisement/HTML boilerplate; develop component-aware leakage policy; make notebook rerun-safe; refresh outdated README research status; freeze evaluation boundaries only after review.

---

## 2026-10-25 — Stage 6: Proposed Holdout and Duplicate Screening

**Researcher:** Islam Uddin (231-115-063)  
**Verified notebook:** [`notebooks/06_matched_rate_protocol.ipynb`](../notebooks/06_matched_rate_protocol.ipynb) (commit `6c8516e2dd2c914bcece8292d8a1714ce0f0be3c`). Saved notebook outputs were checked; experiments were not independently rerun.

### Completed and observed
- Revalidated AG News partitions: 2,000 anchor, 1,800 development, 16,200 noisy training, and 500 exploratory pilot. Record-ID disjointness, pilot subset, and reference-label omission from noisy inputs passed.
- According to notebook output, created a **draft** `configs/matched_rate_noise_protocol_draft_v1.json` on Google Drive. Proposed rates: 10%, 20%, 30% plus exploratory 21.8%; no real matched-rate experiment executed.
- Deterministically selected a **proposed, unsaved and unfrozen** 2,000-record correction holdout from the noisy pool excluding pilot (seed `20261025`), leaving 13,700 training records. Holdout selection ID SHA256: `37bd15b5fb600e7cfc392c936fcb187b0a39313c3502098c0e6a403153057051`.
- Normalized exact-text audit found two cross-split collisions (anchor–training, development–training) and two internal training duplicates. A **candidate, unsaved** training view excluded IDs `9266`, `43231`, `68708`, `79605`, yielding **13,696** records. Candidate ID SHA256: `52176ba0a8d8d43415ff34c083f75de26f9db05b73458893a23413998aa28b88`. Original split CSVs were not changed.
- Text-only TF-IDF screening compared 2,000 candidate holdout records to 17,996 non-holdout records. Initial top-1 audit found **18** scores at least 0.95 and **57** at least 0.85; no normalized exact-text holdout overlaps. Of the 57 pairs, **15** satisfied both additional sequence and Jaccard screening criteria.
- Expanded to five nearest neighbors per holdout: **63 candidate pairs at similarity ≥0.85 involving 57 unique holdout IDs**, with 6 holdouts having multiple flagged matches. Pair counts by comparison split: training 45, anchor 7, development 9, pilot 2. Five protected-split article pairs appeared near-identical upon preliminary textual comparison, but independent human review is pending.

### Scientific limits and continuation
- Similarity flags are **not confirmed duplicates**, and top-five screening cannot guarantee independence or catch all near-duplicates; within-holdout near-duplicate review remains undone.
- No private reference labels or official test records were used. No S-SLTE correction effectiveness was measured. The proposed holdout and training view are **NOT frozen**, and no candidate CSV files were saved.
- The committed notebook contains five illustrative full article texts in its diagnostic outputs; review and clear unnecessary article-text outputs in a future notebook revision. No common credential signatures were detected in the notebook review.
- **Next:** apply a consistent human-review rubric to flagged pairs; develop a deterministic, leakage-aware holdout replacement policy; verify further duplicate risks; obtain methodological review before freezing partitions or training models.

---

## 2026-10-24 — Stage 5: Class-Dependent Error Analysis and S-SLTE Evaluation Protocol Draft

**Reported by:** Islam Uddin (231-115-063).  
**Project:** Evaluating Semantic-Neighborhood Label Correction for LLM-Generated Noise in Text Classification  
**Evidence:** Connected GitHub notebook `notebooks/04_pilot_error_patterns.ipynb` (commit `669dcc617e2e256658b4445b66400348096d72be`), including saved cell outputs. The private Drive files were not independently re-executed or fetched in this GitHub review.

### Completed work and observed results

1. Revisited the frozen 500-record AG News pilot labels from the revision-pinned Llama annotation run; performed retrospective error-pattern analysis separately from LLM inference and S-SLTE correction. Verified 500 records and 109 disagreements against AG News reference labels, consistent with the prior 21.80% preliminary disagreement estimate.
2. Computed the **class-conditional label-transition matrix**, class-wise counts, disagreements, and ordered directional transitions. The notebook reported `CLASS-DEPENDENT ANALYSIS COMPLETED`, and the aggregate tables were saved to the restricted Google Drive evaluation directory.
3. Saved `configs/s_slte_evaluation_protocol_draft_v1.json` on Google Drive, with **12 planned metrics** and proposed `retain`, `correct`, `abstain` decisions. Its status is explicitly **`DRAFT_REQUIRES_REVIEW`**.
4. Saved `configs/s_slte_metrics_draft_v1.json` on Google Drive, defining **seven mutually exclusive candidate outcome categories** and **nine formula/count entries**. The metrics document also remains **`DRAFT_REQUIRES_REVIEW`**.
5. Executed an initial seven-example **synthetic metric consistency unit test**, one example per outcome category. The notebook reported `S-SLTE METRIC CONSISTENCY TEST PASSED`, including explicit handling of zero denominators. This verifies the worked synthetic examples, **not** the correction method's real-world effectiveness.

### Class-dependent pilot observations

| Reference class | Reference records | Disagreements | Within-class rate |
| --- | ---: | ---: | ---: |
| World | 120 | 11 | 9.17% |
| Sports | 108 | 6 | 5.56% |
| Business | 143 | 25 | 17.48% |
| Sci/Tech | 129 | 67 | 51.94% |
| **Total** | **500** | **109** | **21.80%** |

Most frequent directional disagreements:

| Reference → LLM | Count | Percent of reference class |
| --- | ---: | ---: |
| Sci/Tech → Business | 38 | 29.46% |
| Sci/Tech → World | 23 | 17.83% |
| Business → World | 19 | 13.29% |
| World → Sports | 7 | 5.83% |
| Sci/Tech → Sports | 6 | 4.65% |

The Sci/Tech class accounts for **67/109 (61.47%)** of observed disagreements. This is a descriptive pattern in the exploratory pilot, not confirmation of systematic bias, manually adjudicated errors, or any improvement attributable to S-SLTE.

### Proposed outcome taxonomy (not yet finalized)

- **B**: wrong original label corrected to the reference label (beneficial).
- **H**: correct original label changed to an incorrect label (harmful).
- **U**: incorrect original label changed to another incorrect label (unsuccessful).
- **RC / RW**: originally correct / wrong label retained without change.
- **AC / AW**: originally correct / wrong label abstained from automatic acceptance.

Proposed evaluation: correction precision `B/(B+H+U)`, harmful-correction share `H/(B+H+U)`, abstention rate `(AC+AW)/N`, coverage `(N-AC-AW)/N`, and error rate on accepted records `(RW+H+U)/(N-AC-AW)`. All such rates must state their denominator and use an undefined/null result for an empty denominator. Original error rate in this taxonomy is `(B+U+RW+AW)/N`. These are draft definitions rather than final supervisor-approved metrics.

**Synthetic test results (7 artificial records):** correction precision 0.3333, harmful-correction share 0.3333, abstention rate 0.2857, coverage 0.7143, accepted error rate 0.6000, original error rate 0.5714; zero-denominator handling passed. The test uses **synthetic labels only**.

### Research safeguards and open points

- Keep retrospective reference labels out of LLM inference, S-SLTE score computation, and correction-threshold selection; do not tune on the exploratory 500-record pilot's reference labels.
- Use the independent development partition for appropriate method selection, and reserve strictly held-out evaluation data; the official AG News test split is for the final downstream task.
- Report coverage alongside accepted error rate, and distinguish beneficial, harmful, unsuccessful and abstained cases.
- The current synthetic test confirms only simple arithmetic/outcome classification paths. It does not yet test larger randomized cases, invalid decisions, metric sensitivity, or downstream performance.
- Draft metrics still need precise operational definitions for noise-detection precision/recall/F1, risk at coverage, class-wise outcomes, and downstream macro-F1; the proposed abstention/training policy also requires review.
- No S-SLTE correction or abstention predictions, baseline comparisons, effectiveness gains or confirmatory claims have been made.
- Supervisor feedback and independent teammate review are not yet documented.

### Artifacts and next steps

- GitHub: [`notebooks/04_pilot_error_patterns.ipynb`](../notebooks/04_pilot_error_patterns.ipynb) (commit `669dcc617e2e256658b4445b66400348096d72be`).
- Restricted Google Drive drafts: `configs/s_slte_evaluation_protocol_draft_v1.json`; `configs/s_slte_metrics_draft_v1.json`.
- Restricted Google Drive aggregate analysis: `private_reference/evaluation/pilot_v1/classwise_disagreement.csv` and `label_transitions.csv`.
- Next: review and freeze the evaluation protocol with supervisor input; define matched-rate symmetric-noise experiments and safeguard against data leakage; implement a robust correction outcome evaluator and stronger synthetic tests; proceed to S-SLTE implementation only after experimental design is adequately specified.

### Extra session — fixed-count symmetric noise generator (synthetic validation)

**Notebook:** [`notebooks/05_symmetric_noise_baseline.ipynb`](../notebooks/05_symmetric_noise_baseline.ipynb)  
**Verified notebook commit:** `0d8d571cec896945394edd8dfaa8cda032744fb1`  
**Scope:** Synthetic validation only; no AG News rows or restricted reference labels were loaded in this notebook.

1. Implemented `inject_symmetric_noise(clean_labels, noise_rate, seed, num_classes)`. It selects a fixed number `round(N * noise_rate)` of examples without replacement and replaces each selected label uniformly with one of the other `num_classes - 1` labels using a nonzero modular offset. Fixed seed enables deterministic repeat runs. For `N=500`, four classes, and `noise_rate=0.218`, exactly **109 labels** were changed (**21.80%**). The original labels are not retained for selected examples.
2. Tested 500 balanced synthetic records (125 per class), exact flip count, absence of self-label transitions, valid class values, identical-seed reproducibility, and rejection of an invalid rate.
3. Executed **200 seed-specific synthetic trials**, each with 500 records and 109 flips; **21,800 flips** in aggregate. Incorrect-label modular offsets 1, 2, 3 occurred **7,170**, **7,418**, **7,212** times respectively, corresponding to **0.328899**, **0.340275**, **0.330826** of total flips; each theoretically has probability 1/3 under this implementation. The maximum absolute empirical deviation from 1/3 was **0.0069**, below the diagnostic tolerance of **0.035**.
4. The committed notebook reports `SYMMETRIC NOISE GENERATOR TEST PASSED` and `SYMMETRIC NOISE FAIRNESS TEST PASSED`. Saved outputs contain no per-record reference labels, and no common credential patterns were detected in the reviewed notebook.

**Interpretation and limitations:** This is a **fixed-count** corruption generator with uniform replacement among the incorrect classes. Unlike independent Bernoulli corruption per record, the total count is fixed, so individual corruption selections are not independent within a run. Equal probabilities do **not** imply exactly equal observed class-wise corruption rates. The 200-run check is a diagnostic, not a formal statistical significance test, independent ground-truth evaluation, or demonstration that S-SLTE is effective. No real matched-rate baseline has been run.

**Protocol safeguard:** The exploratory 500-record pilot's **21.80% reference-label disagreement** is used here solely as a motivating nominal synthetic test rate. It is not a proven noise rate for all 16,200 records. Before any comparable real-data noise experiment, explicitly specify the reference-label access boundary, record matching, clean source of labels, seeds, target-rate selection without peeking at protected evaluation labels, control conditions, and sensitivity analysis across several rates.

**Next:** Review the draft evaluation protocol, formalize legitimate clean-label sources for synthetic corruption, define matched-rate comparator experiments and held-out evaluation boundaries, then implement robust baselines and the proposed S-SLTE correction pipeline.

---

## 2026-10-23 — Stage 4: Pinned LLM Pilot and Preliminary Noise Evaluation

**Reported by:** Islam Uddin (231-115-063).  
**Model:** `meta-llama/Llama-3.1-8B-Instruct`  
**Pinned revision:** `0e9e39f249a16976918f6564b8830bc894c89659`  
**Dataset:** AG News; 500 random records (seed 42) from the 16,200-record noisy-training partition. Reference labels excluded from model input.

### Completed work

1. Saved model revision, pilot-data SHA256, prompt fingerprint, generation settings and package metadata. Retained the initial manifest and wrote `llm_pilot_manifest_v2.json` to correct the previously missing `bitsandbytes==0.50.2` version.
2. Loaded the revision-pinned Llama tokenizer and 4-bit NF4 model on a Colab Tesla T4 GPU.
3. Generated five pinned-model predictions as a preliminary test. Completed the 500-record annotation pilot with **5 checkpointed predictions reused and 495 newly generated**.
4. Verified all 500 prediction records for unique IDs, text hashes, pinned model revision, prompt fingerprint and decoding settings. **500/500 responses used valid JSON labels; 0 invalid responses.**
5. Evaluated labels retrospectively in a separate notebook against the restricted AG News reference labels: **391 matches, 109 disagreements**, corresponding to **21.80% disagreement** (Wilson 95% CI: **18.40%–25.63%**).

### Predicted class distribution

| Predicted class | Count | Share |
| --- | ---: | ---: |
| World | 156 | 31.2% |
| Sports | 117 | 23.4% |
| Business | 161 | 32.2% |
| Sci/Tech | 66 | 13.2% |

### Confusion matrix (reference labels as rows; LLM predictions as columns)

| Reference label | World | Sports | Business | Sci/Tech |
| --- | ---: | ---: | ---: | ---: |
| World | 109 | 7 | 4 | 0 |
| Sports | 5 | 102 | 1 | 0 |
| Business | 19 | 2 | 118 | 4 |
| Sci/Tech | 23 | 6 | 38 | 62 |

### Class-conditional disagreement

| Reference class | Examples | Disagreements | Rate |
| --- | ---: | ---: | ---: |
| World | 120 | 11 | 9.17% |
| Sports | 108 | 6 | 5.56% |
| Business | 143 | 25 | 17.48% |
| Sci/Tech | 129 | 67 | 51.94% |

The largest observed disagreement concerns Sci/Tech articles (38 predicted as Business, 23 as World and 6 as Sports). This is an **exploratory class-dependent disagreement pattern**, not proof of systematic model bias or of any S-SLTE correction benefit.

### Academic limitations and safeguards

- JSON validity is not classification accuracy. Disagreement with a reference label is not necessarily a manually verified model error.
- The 500-item pilot was sampled randomly rather than stratified separately; its observed disagreement cannot be generalized without further study.
- Reference labels were consulted in the separate evaluation notebook only *after* the annotation outputs were generated. Do not feed them to the correction pipeline or tune S-SLTE hyperparameters on pilot reference labels.
- The official AG News test set remains untouched. S-SLTE correction, comparator baselines, correction/abstention risk and downstream model results have not yet been produced.
- Notebook code and outputs are verified in GitHub, but this entry is not a second independent runtime reproduction or direct audit of private Drive files.

### Artifacts

- `notebooks/02_llm_annotation.ipynb` — pinned-model pilot annotation, checkpoints and audit
- `notebooks/03_pilot_noise_analysis.ipynb` — separate retrospective reference-label evaluation
- Google Drive (restricted): `configs/llm_pilot_manifest_v2.json`, `outputs/llm_pilot/pinned_0e9e39f249a1/`, `private_reference/evaluation/pilot_v1/noise_summary.json`
- Pilot CSV SHA256: `56c3d987cf681b590e8371d76a848b39ee8e3bd99c6385e598acdcfcf66b27de`

### Next session

1. Review representative disagreements and label ambiguity without retroactively altering the annotation protocol.
2. Define an S-SLTE correction/abstention evaluation plan with separate development and held-out datasets.
3. Specify a matched-rate symmetric-noise comparator and class-wise metrics.
4. Record supervisor and collaborator review before making method-effectiveness claims.

---

## 2026-10-22 — Stage 3: LLM Access, Environment Setup, and 10-Record Smoke Test

**Reported by:** Islam Uddin (231-115-063)  
**Model:** `meta-llama/Llama-3.1-8B-Instruct`  
**Compute:** Google Colab, NVIDIA Tesla T4 (14.56 GB total VRAM)  
**Objective:** Verify authorized model access, run zero-shot AG News classification, and confirm robust output parsing and persistence.

### Reported completed work

1. Hugging Face model-access request was approved; `HF_TOKEN` was stored in Colab Secrets rather than embedded in committed code.
2. Diagnosed and resolved Colab package version conflicts; the researcher reported `transformers==5.20.1`, `huggingface_hub==1.33.0`, and a successful dependency check after environment repair. These are **observed runtime versions**, not yet a frozen environment manifest.
3. Loaded Meta Llama 3.1 8B Instruct on the Tesla T4 using 4-bit NF4 quantization. The researcher reported **5.32 GB allocated GPU memory**, with approximately **9.0 GB free** after loading.
4. Defined a four-class zero-shot news categorization prompt with strict JSON response format and validated a JSON parser that rejects malformed or out-of-range responses.
5. Saved and verified the annotation protocol in Google Drive; a previously existing protocol is preserved rather than silently overwritten.
6. Ran the first inference on AG News training record `43`. The model returned `{"label": 0}` (World); the parser accepted it, and the first-prediction JSON file was reportedly saved and verified.
7. Ran a **10-record smoke test** from the fixed pilot sample, saving individual JSON prediction files in Google Drive. All 10 responses parsed as valid JSON category labels (**10/10 valid; 0 invalid**).

### Reported smoke-test predictions

| Record ID | Predicted label | Predicted class | JSON valid |
| ---: | ---: | --- | --- |
| 43 | 0 | World | Yes |
| 188 | 3 | Sci/Tech | Yes |
| 765 | 1 | Sports | Yes |
| 913 | 1 | Sports | Yes |
| 926 | 0 | World | Yes |
| 932 | 0 | World | Yes |
| 1107 | 3 | Sci/Tech | Yes |
| 1117 | 3 | Sci/Tech | Yes |
| 1260 | 0 | World | Yes |
| 1262 | 1 | Sports | Yes |

### Interpretation and important boundaries

- **10/10 JSON-valid responses measure formatting compliance, not classification accuracy.** The true/reference labels were not used for these predictions.
- The GPU load, inference results, and Google Drive output files were reported through the Colab session; a separate end-to-end reproduction and direct Drive file inspection have **not** yet been performed.
- The annotation model repository's **exact model revision SHA** and complete package environment should be recorded before treating the 500-record pilot as a reproducible experiment.
- There are no S-SLTE label corrections, noise-detection scores, or downstream classification results yet.
- The GitHub notebook `notebooks/02_llm_annotation.ipynb` has not yet been verified to contain today's actual inference and 10-record smoke-test cells at the time of this log update.

### Next actions

1. Verify the 10 persisted prediction files, their IDs, JSON validity, and matching prompt hashes.
2. Record the exact model revision, library versions, and the fixed pilot input list; avoid any changes to the frozen prompt during the pilot.
3. Save the updated Colab notebook to `notebooks/02_llm_annotation.ipynb` and verify the GitHub copy.
4. Run a checkpointed 500-record pilot, treating invalid answers as invalid rather than relabeling them by hand.
5. Inspect results and evaluate annotation disagreements only **after** producing the predictions, without feeding reference labels into the inference pipeline.

---

## 2026-10-21 — Stage 2: Dataset Verification and Annotation Pilot Preparation

**Reported by:** Islam Uddin (231-115-063)  
**Team:** Islam Uddin and Md Yousuf Bin Nazib (231-115-064); Yousuf's independent review is not yet recorded  
**Objective:** Verify persisted AG News partitions and create a reproducible 500-record pilot for LLM labeling.

### Completed according to the researcher's session report

- Reloaded the saved Google Drive data-partition CSV files.
- Ran the partition verification checks and received `DATASET VERIFICATION PASSED`.
- Selected a reproducible 500-record pilot sample from the noisy-training partition using `PILOT_SEED = 42`.
- Saved `pilot_500.csv` in Google Drive and received `PILOT SAVE VERIFIED`.
- Reported completing a Colab-to-GitHub notebook commit.

**Files expected in Google Drive:**
- `MyDrive/S_SLTE_Thesis/data/splits/anchor.csv`
- `MyDrive/S_SLTE_Thesis/data/splits/development.csv`
- `MyDrive/S_SLTE_Thesis/data/splits/noisy_training.csv`
- `MyDrive/S_SLTE_Thesis/private_reference/noisy_reference.csv` (restricted, never provided to correction code)
- `MyDrive/S_SLTE_Thesis/data/pilot/pilot_500.csv` (record ID and text only)

### Independent repository review / outstanding issue

During a GitHub check on 2026-10-21, the `main` branch notebook at `notebooks/01_dataset_preparation.ipynb` still contained the earlier dataset-splitting code **without** the new verification and pilot-sampling cells. Thus the latest notebook commit is **not independently verified at that canonical repository path**. It may have been committed under another path or not synchronized.

**Required follow-up:** In Colab, verify the current notebook includes the Stage 2 cells; save it to `main` at `notebooks/01_dataset_preparation.ipynb`; open GitHub to confirm it contains the pilot code. Avoid adding credentials or private-reference labels to notebook outputs.

### Scientific significance

- Independent reload and assertions reduce the risk of silently missing or overlapping data partitions.
- Fixed-seed pilot sampling makes the preliminary LLM annotation study reproducible.
- Keeping the original reference labels out of the pilot protects against label leakage.
- All 500 pilot items are **unlabeled by an LLM at this stage**; these actions do not measure LLM accuracy or demonstrate S-SLTE performance.

### Next actions

1. Reconcile Colab with the GitHub notebook path and confirm the Stage 2 code is committed.
2. Prepare `notebooks/02_llm_annotation.ipynb` and check Colab GPU/VRAM.
3. Establish authorized access to the intended LLM annotator or document a supervisor-approved fallback.
4. Freeze a zero-shot prompt, class-label mapping, generation settings and parsing policy.
5. Run a 10-example inference **smoke test**, inspect invalid outputs and persist the annotation batches with record IDs.
6. Update this log after actual model inference; do not infer experiment results in advance.

**Status:** Dataset and pilot completion user-reported; GitHub synchronization requires confirmation; LLM inference not yet started.

---

## 2026-10-17 — Day 01: Environment Setup and AG News Data Preparation

**Primary work recorded for:** Islam Uddin (231-115-063)  
**Teammate:** Md Yousuf Bin Nazib (231-115-064); independent review/verification not yet recorded  
**Objective:** Establish a reproducible working environment, obtain the AG News dataset, prepare the research partitions, and preserve the code in version control.

### Work completed

1. Created a private GitHub repository: [islam37/S-SLTE-Thesis](https://github.com/islam37/S-SLTE-Thesis).
2. Created a Google Colab notebook for dataset preparation and installed the required Python packages (`datasets`, `pandas`, and `scikit-learn`).
3. Loaded the publicly available AG News dataset from Hugging Face (`fancyzhx/ag_news`).
4. Verified that the official dataset contains **120,000 training records** and **7,600 test records**.
5. Connected Colab to Google Drive and saved copies of the original training and test datasets as CSV files; a subsequent notebook run showed that these could be loaded with the expected record counts.
6. Implemented stratified sampling using **random seed 42** to select **20,000 records** from the official training split.
7. Implemented non-overlapping partitions: **2,000 clean-anchor records**, **1,800 development records**, and **16,200 noisy-training records**.
8. Verified the following balanced per-class counts in the notebook output:

| Partition | World | Sports | Business | Sci/Tech | Total |
| --- | ---: | ---: | ---: | ---: | ---: |
| Clean anchor | 500 | 500 | 500 | 500 | 2,000 |
| Development | 450 | 450 | 450 | 450 | 1,800 |
| Noisy training/reference | 4,050 | 4,050 | 4,050 | 4,050 | 16,200 |
| **Total** | **5,000** | **5,000** | **5,000** | **5,000** | **20,000** |

9. Added code to save the derived partitions in Google Drive and to keep the original noisy-training reference labels in a separate `private_reference` directory. **Persistence of all derived files still needs an independent reload check.**
10. Committed the notebook to GitHub. Organized it under `notebooks/`, added research-oriented repository folders and README guidance, and extended `.gitignore` rules to exclude research datasets, private reference labels, large model checkpoints and secrets.

### Why these steps matter

- **Stratified sampling:** preserves the dataset's four-class balance so class imbalance does not become an unintended explanation for performance differences.
- **Fixed seed (42):** enables the same record partitions to be reproduced in future experiments.
- **Disjoint partitions:** helps prevent overlap between trusted training examples, development data and noisy training data.
- **Separate reference labels:** keeps the original labels available for *retrospective evaluation* without exposing them to the label-correction algorithm during development.
- **Official test split isolation:** reserves the 7,600 test examples for final evaluation rather than model selection.
- **Version control and notes:** provide an auditable code history for collaboration, supervisor review and thesis writing.

### Files / artifacts

- **GitHub notebook:** [`notebooks/01_dataset_preparation.ipynb`](../notebooks/01_dataset_preparation.ipynb)
- **Google Drive workspace:** `MyDrive/S_SLTE_Thesis/`
- **Original data (recorded):** `data/ag_news_train.csv`, `data/ag_news_test.csv`
- **Derived split files (saving code added; independently verify next session):**
  - `data/splits/anchor.csv`
  - `data/splits/development.csv`
  - `data/splits/noisy_training.csv`
  - `private_reference/noisy_reference.csv` (restricted use)

### Verification status and limitations

- **Observed in Colab output:** official AG News dataset sizes and the four-class split distributions.
- **Confirmed in GitHub:** dataset-preparation notebook and repository organization.
- **To verify next session:** CSV paths, actual persisted row counts, unique/disjoint `record_id` values, and the absence of original reference labels from the noisy-training input file.
- **Not yet performed:** 500-record annotation pilot, LLM inference, label-error measurement, DistilBERT training, SBERT/KNN correction, Confident Learning, comparative evaluation, and human audit.
- **Research interpretation:** no annotation accuracy, correction improvement or novel-method performance can yet be claimed.

### Next session — planned tasks

1. Reload all derived split CSV files from Google Drive; validate columns, row counts, unique record IDs, disjointness and reference-label separation.
2. Select and save a reproducible **500-record pilot sample** from the noisy-training partition without including the reference labels.
3. Create `notebooks/02_llm_annotation.ipynb`; check free GPU availability and model-access/licensing requirements.
4. Test a documented zero-shot annotation prompt on **10 examples** before any larger labeling run.
5. Record model identifier/version, prompt, generation parameters, parsing failures, compute constraints, and exact code commit.

**Supervisor confirmation needed:** final data-visibility rules, the selected LLM annotator and contingency model, and any required approval for later human review.

---

*Append future daily entries above or below this one consistently; keep dates in ISO format (YYYY-MM-DD) and update only after completing the work.*
