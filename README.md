# S-SLTE Undergraduate Thesis

**Project:** Evaluating Semantic-Neighborhood Label Correction for LLM-Generated Noise in Text Classification  
**Researchers:** Islam Uddin (231-115-063) and Md Yousuf Bin Nazib (231-115-064)  
**Institution:** Department of Computer Science and Engineering, Metropolitan University, Sylhet  
**Supervisor:** Golam Mostofa Naeem  
**Target:** January 2027

## Research status
**Current stage (23 October 2026):** AG News 500-record exploratory LLM annotation pilot and retrospective reference-label evaluation completed.

- Dataset: 20,000 stratified training examples (seed 42): 2,000 clean anchor, 1,800 development, 16,200 noisy training.
- Annotator: `meta-llama/Llama-3.1-8B-Instruct`, pinned revision `0e9e39f249a16976918f6564b8830bc894c89659`.
- Pilot: 500/500 JSON-valid LLM labels (100% output-format compliance).
- Retrospective evaluation: 391/500 reference-label matches; 109/500 disagreements (21.80%; Wilson 95% CI 18.40%–25.63%).
- Largest class-specific disagreement: Sci/Tech, 67/129 (51.94%). These are exploratory disagreements, not manually adjudicated errors.
- Outstanding: S-SLTE correction experiments, abstention/risk evaluation, baselines, and downstream classifier evaluation. The official AG News test split remains untouched.

Pilot reference labels are restricted to retrospective evaluation, not correction-threshold tuning. No superiority or novelty claim is established. See [research log](docs/research-log.md) for full results and limitations.

## Repository layout
- `notebooks/`: reproducible Google Colab notebooks
- `src/`: reusable Python source code (future stages)
- `configs/`: documented experiment settings and seeds
- `results/`: small aggregate results and figures, once verified
- `docs/`: research logs, protocol and literature notes
- `thesis/`: writing drafts, as appropriate

## Dataset protocol
AG News official training split: select 20,000 stratified examples (seed 42), then partition into 2,000 clean anchor, 1,800 development, and 16,200 noisy training examples. Keep the official 7,600-example test set isolated from model selection. Preserve noisy training reference labels separately and never expose them to the correction pipeline.

## Data storage and privacy
GitHub holds code and documentation, **not** bulk datasets, private reference labels, credentials or large checkpoints. The Google Drive workspace `S_SLTE_Thesis/` is used for restricted research files and backups. Check notebook outputs before committing.

## Start here
Open `notebooks/01_dataset_preparation.ipynb` in Colab. Review the cells and update the notebook only after checking output correctness. Maintain a dated experiment log with code version, dataset split seed and package/model versions.

## Research integrity
The candidate correction method combines clean-anchor confidence, semantic-neighborhood evidence and conservative abstention. A close-prior-work comparison is required; any claimed novelty must be substantiated. Future experiments and outcomes must be documented even when results are negative.
