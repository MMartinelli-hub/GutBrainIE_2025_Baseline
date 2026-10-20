<i>Research use only. Not a medical device.
This software was developed in the HEREDITARY project as a research prototype. It has not been assessed or certified under the EU Medical Device Regulation (EU) 2017/745. It must not be used to diagnose, treat, monitor or make any other decision about individual patients.</i>
***

# GutBrainIE Baselines & Evaluation

This repository provides the complete pipeline for the baseline implementation and evaluation for the [GutBrainIE](https://hereditary.dei.unipd.it/challenges/gutbrainie/2025/) challenge – the sixth task of the [BioASQ](https://www.bioasq.org/) Lab @ [CLEF 25](https://clef2025.clef-initiative.eu/). We employ [GLiNER (NuNerZero)](https://huggingface.co/numind/NuNER_Zero) for Named Entity Recognition (NER) and [ATLOP](https://github.com/wzhouad/ATLOP) for Relation Extraction (RE).

Users have three main options:
- **Reproduce the baseline from scratch:** Run the entire pipeline — including data processing, training, prediction generation, and evaluation.
- **Test with pre-trained models:** Directly import our fine-tuned models for generating and evaluating predictions.
- **Evaluate provided outputs:** Use the supplied model outputs and directly run the evaluation as detailed in the TL;DR section.

Evaluation results on the dev set and on all of the training sets, along with detailed explanations of the employed evaluation metrics, are available at the [CHALLENGE WEBSITE](https://hereditary.dei.unipd.it/challenges/gutbrainie/2025/). 

These results are obtained by training the NER model on the platinum, gold, and silver collections, while for the RE model the platinum, gold, and silver collections were used as manually annotated sets, and the bronze collection as the distantly supervised annotated set.

---

## Getting Started

1. **Clone the Repository**

2. **Dataset Setup**: Replace the empty `Annotations` and `Articles` folders with the corresponding GutBrainIE dataset folders.

---

## Quick Start (TL;DR)
1. **Evaluate Baseline Predictions:**
   - Execute the script `Eval/evaluate.py` to generate evaluation results.
   - This step will evaluate the baseline predictions on the dev set contained in `Eval/org_*`.
2. **Evaluate Your Predictions:**
   - Format your predictions in the submission format, as the files `Eval/org_*`.
   - Open the script `Eval/evaluate.py` and adjust the paths to point to your prediction files.
   - Execute the script `Eval/evaluate.py` to generate evaluation results.
---

## Data Preparation and Training

### Data Conversion

Before training, convert your annotations into the required formats:

- **NER Conversion:**  
  Run [`Utils/annotations_to_gliner_format.ipynb`](Utils/annotations_to_gliner_format.ipynb) to convert annotations for GLiNER. This produces data in `Train/NER/data`.

- **RE Conversion:**  
  Run [`Utils/annotations_to_atlop_format.ipynb`](Utils/annotations_to_atlop_format.ipynb) to convert annotations for ATLOP. This produces data in `Train/RE/data`.

### NER Fine-tuning

1. **Configure Training:**  
   Navigate to `Train/NER` and open `gliner_interface.py`. Adjust the following as needed:
   - Pre-trained model selection.
   - Confidence threshold for evaluation.
   - Flags for training vs. prediction.
   - Choice of training sets (platinum, gold, silver), training/evaluation steps, batch size, etc.

2. **Run Fine-tuning:**  
   Execute `gliner_interface.py`. If running on Windows, you need to launch that command while executing powershell in administrator mode.
   - **Outputs:**  
     - Evaluation checkpoints are saved in `Train/NER/logs`.
     - The final fine-tuned model is stored in `Train/NER/outputs`.

### RE Training

1. **Prepare Training Data:**  
   In `Train/RE`, open and run `compose_training_sets.py` to decide which sets are “manually annotated” and which are “distantly annotated.”  
   - This script produces:
     - `Train/RE/data/train_annotated.json`
     - `Train/RE/data/train_distant.json`

2. **Configuration Check:**  
   Ensure that the number of relations in `Train/RE/data/meta/rel2id.json` match the `num_class` parameter in `atlop_finetune.sh` and that all the relations in your training set are included.

3. **Run Fine-tuning:**  
   Optionally adjust training parameters in `atlop_finetune.sh` and then run the script to fine-tune ATLOP. If running on Windows, you might need to replace the backslash (\) characters used for multi-line prompting with the backtick (\`) character and paste the command directly into powershell executed in administrator mode.

---

## Generating Predictions

### NER Predictions

1. **Generate Predictions:**  
   In `Train/NER`, set the `generate_predictions` flag to `True` in `gliner_interface.py` and run the script.  
   - The predictions are saved as `Predictions/NER/predicted_entities.json`.

2. **Convert Predictions:**  
   - Run [`Utils/NER_predictions_to_evaluation_format.ipynb`](Utils/NER_predictions_to_evaluation_format.ipynb) to convert GLiNER predictions into evaluation format, producing `Predictions/NER/predicted_entities_eval_format.json`.
   - Next, run [`Utils/NER_predictions_to_atlop_format.ipynb`](Utils/NER_predictions_to_atlop_format.ipynb) to convert these into the ATLOP format, saving the file in `Train/RE/data/predicted_entities_atlop_format.json`.

### RE Predictions

1. **Generate Predictions:**  
   Navigate to `Train/RE` and run `atlop_generate_predictions.sh`. This loads your trained ATLOP model (as configured in the script) and outputs predictions to `Predictions/RE/predicted_relations.json`. 

    If running on Windows, you might need to replace the backslash (\\) characters used for multi-line prompting with the backtick (\`) character and paste the command directly into powershell executed in administrator mode. 
    
    If not done automatically by the script, move the `Train/RE/outputs/results.json` file produced in output to `Predictions/RE/predicted_relations.json`.

2. **Merge Predictions:**  
   Run [`Utils/merge_predictions_to_evaluation_format.ipynb`](Utils/merge_predictions_to_evaluation_format.ipynb) to combine the NER and RE outputs into the evaluation files `Predictions/predictions_eval_format.json`, `Eval/teamID_T61_runID_systemDesc.json`, `Eval/teamID_T621_runID_systemDesc.json`, `Eval/teamID_T622_runID_systemDesc.json`, `Eval/teamID_T623_runID_systemDesc.json`.

---

## Evaluation

With the merged predictions in `Eval/**.json`, navigate into `Eval/` and run:

```bash
python evaluate.py
```

This script calculates evaluation metrics for all subtasks. 
To test your own results, place them in the `Eval` folder, adjust the paths in `evaluate.py`, and run the script.

---

## Using Pre-trained Models

If you prefer to bypass the fine-tuning phase, you can download and use our pre-trained models.

### Importing the GLiNER Model

1. **Download:**  
   Get our fine-tuned GLiNER model from [HERE](https://www.dei.unipd.it/~martinell2/gbie2025models/NER.zip).
2. **Setup:**  
   Unzip and place the folder content in `Train/NER/outputs` and adjust the `model path` variable in `gliner_interface.py`.
3. **Generate Predictions:**  
   Run `gliner_interface.py` to produce NER predictions by setting the flags `finetune_model` to False and `generate_predictions` to True.

### Importing the ATLOP Model

1. **Download:**  
   Get our trained ATLOP model from [HERE](https://www.dei.unipd.it/~martinell2/gbie2025models/RE.zip).
2. **Setup:**  
   Unzip and place the folder content in `Train/RE/outputs` and update the `load_path` and `load_checkpoint` in `atlop_generate_predictions.sh`.
3. **Generate Predictions:**  
   Execute `atlop_generate_predictions.sh` to obtain RE predictions.

---

## Dependencies & Environment Setup
All dependencies are listed in `requirements.txt`. For a consistent environment, we recommend using Conda. 

After initializing your environment, please run the following commands to ensure the correct installation of PyTorch with CUDA support:
```bash
pip uninstall torch
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

---

## Contributions & Contact

Feel free to open issues if you have any questions or improvements. For further inquiries, please reach out at: [martinell2@dei.unipd.it](mailto:martinell2@dei.unipd.it).

---

### Acknowledgement

> Funded by the European Union under grant agreement No 101137074 (HEREDITARY). Views and opinions expressed are however those of the author(s) only and do not necessarily reflect those of the European Union or the European Health and Digital Executive Agency (HaDEA). Neither the European Union nor the granting authority can be held responsible for them.
