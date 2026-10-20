# GutBrainIE 2025 Baselines and Evaluation

> **Research use only. Not a medical device.**
> This software was developed in the HEREDITARY project as a research prototype. It has not been assessed or certified under the EU Medical Device Regulation (EU) 2017/745. It must not be used to diagnose, treat, monitor or make any other decision about individual patients.

## 1. Overview

This repository provides the baseline models and evaluation pipeline for [GutBrainIE 2025](https://hereditary.dei.unipd.it/challenges/gutbrainie/2025/), Task 6 of the [BioASQ Lab](https://www.bioasq.org/) at [CLEF 2025](https://clef2025.clef-initiative.eu/). It supports named entity recognition (NER) and three relation extraction (RE) subtasks: binary tag-based RE, ternary tag-based RE, and ternary mention-based RE.

The full collection, comprising documents and their annotations, can be found at: [https://zenodo.org/records/16845409](https://zenodo.org/records/16845409)

Users can reproduce the complete baseline, generate predictions with the released fine-tuned models, or evaluate the supplied predictions and their own outputs.

If you use this benchmark fully or in part, please cite the following paper:

```bibtex
@inproceedings{BioASQ2025taskGutBrainIE,
   title = {{Overview of GutBrainIE@CLEF 2025: Gut-Brain Interplay Information Extraction}},
   author = {Martinelli, M. and Silvello, G. and Bonato, V. and Di Nunzio, G. M. and Ferro, N. and Irrera, O. and Marchesin, S. and Menotti, L. and Vezzani, F.},
   year = 2025,
   booktitle = {CLEF 2025 Working Notes},
   editor = {Faggioli, Guglielmo and Ferro, Nicola and Rosso, Paolo and Spina, Damiano}
}
```

## 2. Intended use

**Primary intended use.** Reproducible development and evaluation of information extraction systems that identify biomedical entities and relations in scientific literature on the gut–brain axis. The baseline provides a reference for comparing systems on the GutBrainIE 2025 benchmark.

**Intended users.** Biomedical NLP researchers, data scientists, software developers, and clinicians conducting research on literature analysis.

**Out-of-scope uses.** This benchmark must not be used for:

- clinical diagnosis, prognosis, treatment, monitoring, or other decisions about individual patients;
- decisions about individuals, such as insurance, employment, or access to care;
- re-identification of annotators or other individuals, including linkage of pseudonymized annotator IDs to identities;
- processing identifiable patient records or other personal records as part of the documented benchmark workflow.

Performance on other languages, domains, full-text articles, or data types has not been established. Such research requires separate evaluation. Extracted relations must be checked against the source publication and must not be interpreted as validated clinical recommendations or evidence of causation.

## 3. Data provenance

The baseline uses the GutBrainIE 2025 collection of 1,647 PubMed titles and abstracts concerning gut microbiota, Parkinson’s disease, and mental health. No full-text articles or restricted-access documents are included in the collection. Source-text reuse remains subject to applicable licensing terms. Corpus statistics and annotation guidelines are available on the [challenge website](https://hereditary.dei.unipd.it/challenges/gutbrainie/2025/).

The collection is available through the [GutBrainIE 2025 benchmark archive](https://doi.org/10.5281/zenodo.16845409). 

The software relies on publicly available libraries and models, including GLiNER/NuNER Zero and ATLOP. Their respective licenses and model terms continue to apply. The baseline does not require proprietary model APIs.

## 4. Method

- **Algorithm or architecture.** NER uses [GLiNER](https://github.com/urchade/GLiNER), initialized from [NuNER Zero](https://huggingface.co/numind/NuNER_Zero), and fine-tuned on the Platinum, Gold, and Silver training collections. The published baseline uses a confidence threshold of 0.6 and merges adjacent or overlapping predicted entities. Document-level RE uses [ATLOP](https://github.com/wzhouad/ATLOP), with Platinum, Gold, and Silver as manually annotated training sets and Bronze as the distantly supervised set. ATLOP receives the text and predicted entities; its output is filtered to the permitted relation schema.
- **Training setting.** Centralized, local training and inference. Data conversion, prediction, and evaluation operate on local files. No interactive user input or hosted inference service is required.
- **Privacy techniques.** Annotator identities are represented by pseudonymized IDs in the source annotations, and identity mappings are kept separately. These IDs are excluded from model inputs. Differential privacy, secure aggregation, and synthetic-data privacy mechanisms are not part of the baseline.

### Setup

Clone the repository and work from its root directory:

```bash
git clone https://github.com/MMartinelli-hub/GutBrainIE_2025_Baseline.git
cd GutBrainIE_2025_Baseline
```

Use an isolated Python environment, for example with Conda, and install the dependencies listed in `requirements.txt`:

```bash
pip install -r requirements.txt
```

For the CUDA 11.8 PyTorch setup used in the original instructions, install compatible PyTorch packages in that environment:

```bash
pip uninstall torch
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

Replace the empty `Annotations` and `Articles` folders with the corresponding folders from the GutBrainIE 2025 dataset. Configure file paths in the scripts before running them. The RE launchers use Bash; on Windows, use a suitable Bash environment or adapt the commands to PowerShell syntax.

### Quick start: evaluation only

To evaluate the supplied development-set predictions in `Eval/org_*`, check the configured paths in `Eval/evaluate.py`, then run:

```bash
cd Eval
python evaluate.py
```

To evaluate your own predictions, use the same submission format as the `Eval/org_*` files, place the files in `Eval/`, and update the paths in `evaluate.py`. Select the matching reference annotations for the split being evaluated.

### Prepare training data

1. Run [`Utils/annotations_to_gliner_format.ipynb`](Utils/annotations_to_gliner_format.ipynb) to produce NER training data in `Train/NER/data`.
2. Run [`Utils/annotations_to_atlop_format.ipynb`](Utils/annotations_to_atlop_format.ipynb) to produce RE training data in `Train/RE/data`.

### Fine-tune the NER model

In `Train/NER/gliner_interface.py`, configure the pretrained model, training sets, training and prediction flags, confidence threshold, batch size, and training/evaluation steps. Run the script from `Train/NER`:

```bash
python gliner_interface.py
```

Evaluation checkpoints are saved in `Train/NER/logs`, and the final model is saved in `Train/NER/outputs`.

### Train the RE model

1. In `Train/RE`, configure and run `compose_training_sets.py` to choose the manually annotated and distantly supervised collections. It produces `data/train_annotated.json` and `data/train_distant.json`.
2. Check that the relation count in `data/meta/rel2id.json` matches `num_class` in `atlop_finetune.sh`, and that the mapping covers all training relations.
3. Configure the training parameters and run:

```bash
bash atlop_finetune.sh
```

### Use the released fine-tuned models

To skip training:

| Model | Download | Setup |
|---|---|---|
| GLiNER/NuNER Zero fine-tuned for NER | [NER.zip](https://www.dei.unipd.it/~martinell2/gbie2025models/NER.zip) | Extract the contents into `Train/NER/outputs`; update the model path in `gliner_interface.py`; set `finetune_model` to `False` and `generate_predictions` to `True`. |
| ATLOP trained for RE | [RE.zip](https://www.dei.unipd.it/~martinell2/gbie2025models/RE.zip) | Extract the contents into `Train/RE/outputs`; update `load_path` and `load_checkpoint` in `atlop_generate_predictions.sh`. |

### Generate and convert predictions

**NER predictions:**

1. Set `generate_predictions` to `True` in `Train/NER/gliner_interface.py` and run the script. Predictions are saved to `Predictions/NER/predicted_entities.json`.
2. Run [`Utils/NER_predictions_to_evaluation_format.ipynb`](Utils/NER_predictions_to_evaluation_format.ipynb) to produce `Predictions/NER/predicted_entities_eval_format.json`.
3. Run [`Utils/NER_predictions_to_atlop_format.ipynb`](Utils/NER_predictions_to_atlop_format.ipynb) to produce `Train/RE/data/predicted_entities_atlop_format.json` for RE inference.

**RE predictions:**

1. From `Train/RE`, run:

   ```bash
   bash atlop_generate_predictions.sh
   ```

2. The expected output is `Predictions/RE/predicted_relations.json`. If the script leaves the output at `Train/RE/outputs/results.json`, move it to that prediction path.
3. Run [`Utils/merge_predictions_to_evaluation_format.ipynb`](Utils/merge_predictions_to_evaluation_format.ipynb) to combine the NER and RE outputs. It produces:

   - `Predictions/predictions_eval_format.json`
   - `Eval/teamID_T61_runID_systemDesc.json`
   - `Eval/teamID_T621_runID_systemDesc.json`
   - `Eval/teamID_T622_runID_systemDesc.json`
   - `Eval/teamID_T623_runID_systemDesc.json`

Evaluate the resulting files using `Eval/evaluate.py` as described above.

## 5. Performance

- **Evaluation data.** The published final results use the held-out GutBrainIE 2025 test set of 40 documents. The repository also supplies development-set predictions for reproducible evaluation. These are benchmark evaluations, not clinical validation.
- **Metrics and results.** Precision, recall, and F1 are reported with micro- and macro-averaging. The table below gives the organizer baseline results reported in the [2025 overview paper](https://ceur-ws.org/Vol-4038/paper_5.pdf), Tables 7–10. Values are on a 0–1 scale; they are test-set results, rather than scores for the supplied development predictions.

| Subtask | Micro precision | Micro recall | Micro F1 | Macro F1 |
|---|---:|---:|---:|---:|
| 6.1 — Named Entity Recognition | 0.7639 | 0.8238 | 0.7927 | 0.7047 |
| 6.2.1 — Binary Tag-Based RE | 0.7584 | 0.4892 | 0.5947 | 0.3864 |
| 6.2.2 — Ternary Tag-Based RE | 0.7533 | 0.4650 | 0.5751 | 0.3745 |
| 6.2.3 — Ternary Mention-Based RE | 0.4986 | 0.2453 | 0.3288 | 0.2123 |

Detailed scoring definitions and additional baseline results are available on the [challenge website](https://hereditary.dei.unipd.it/challenges/gutbrainie/2025/). Reproduced scores may vary with model configuration, random seed, software versions, and the selected data split.

- **Results by subgroup.** No patient-level evaluation by age, sex, country, or clinical site is reported. Benchmark categories concern biomedical entity and relation types, not demographic groups. Confidence intervals are not reported in the cited baseline tables.

## 6. Known limitations and risks

- **Generalizability.** The baseline was evaluated on English PubMed titles and abstracts selected for the gut–brain domain. Performance on other languages, domains, full-text articles, clinical notes, or individual patient records has not been established.
- **Known biases and confounders.** The collection reflects its retrieval queries, publication coverage, and annotation schema. Annotation quality varies across the Platinum, Gold, Silver, and Bronze tiers; Bronze annotations are automatically generated and were not manually revised. Uneven entity and relation frequencies may affect performance on less frequent categories.
- **Failure modes.** NER can miss entities, assign incorrect types, or predict inaccurate boundaries. These errors propagate to RE, which can omit relations or assign incorrect predicates and arguments. Extracted associations do not establish causation or clinical efficacy, and outputs require verification against the original text.

## 7. Responsible use and reporting

- **Terms of use.** Use this release for research in accordance with the repository license, the terms of the pretrained and fine-tuned models, and the applicable source-data licenses. The research-only notice describes the intended use and clinical limitations; it does not replace the applicable license terms.
- **Contact for misuse or vulnerability reports.** Email [martinell2@dei.unipd.it](mailto:martinell2@dei.unipd.it). General questions, reproducibility issues, and proposed improvements may also be submitted through [GitHub Issues](https://github.com/MMartinelli-hub/GutBrainIE_2025_Baseline/issues). Do not post sensitive data or credentials in public issues.

## 8. Acknowledgement

Funded by the European Union under grant agreement No 101137074 (HEREDITARY). Views and opinions expressed are however those of the author(s) only and do not necessarily reflect those of the European Union or the European Health and Digital Executive Agency (HaDEA). Neither the European Union nor the granting authority can be held responsible for them.
