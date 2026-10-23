# Research Progress Log

**Project:** Evaluating Semantic-Neighborhood Label Correction for LLM-Generated Noise in Text Classification  
**Candidate method:** Semantic Label Trustworthiness Estimation (S-SLTE)  
**Institution:** Metropolitan University, Sylhet — Department of Computer Science and Engineering  
**Researchers:** Islam Uddin (231-115-063); Md Yousuf Bin Nazib (231-115-064)  
**Target completion:** January 2027

> **Record-keeping policy:** Enter only work actually performed, observed results, decisions, blockers, and the next planned step. Do not report proposed methods as implemented or make performance claims before experiments. Record who performed or reviewed each task, and keep original reference labels outside the model-correction pipeline.

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
