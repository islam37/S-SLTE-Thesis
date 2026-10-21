# Research Progress Log

**Project:** Evaluating Semantic-Neighborhood Label Correction for LLM-Generated Noise in Text Classification  
**Candidate method:** Semantic Label Trustworthiness Estimation (S-SLTE)  
**Institution:** Metropolitan University, Sylhet — Department of Computer Science and Engineering  
**Researchers:** Islam Uddin (231-115-063); Md Yousuf Bin Nazib (231-115-064)  
**Target completion:** January 2027

> **Record-keeping policy:** Enter only work actually performed, observed results, decisions, blockers, and the next planned step. Do not report proposed methods as implemented or make performance claims before experiments. Record who performed or reviewed each task, and keep original reference labels outside the model-correction pipeline.

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
