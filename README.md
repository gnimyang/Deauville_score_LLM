# Deauville Score LLM

This repository provides the core training and inference scripts for an LLM-based framework for automated Deauville scoring from longitudinal PET/CT reports in lymphoma. The framework reformulates Deauville scoring as a structured clinical reasoning task over paired prior and current PET/CT reports, including lesion-level analysis, candidate lesion identification, main-lesion selection, and final Deauville score determination.

## Overview

Deauville scoring is widely used for PET/CT-based lymphoma treatment response assessment. In clinical practice, scoring requires longitudinal interpretation of prior and current PET/CT findings, assessment of whether FDG-avid lesions are lymphoma-related, selection of lesions eligible for scoring, and comparison of lesion uptake with mediastinal blood pool and liver background uptake.

This project implements a report-based LLM pipeline for:

1. structuring longitudinal PET/CT report information;
2. identifying candidate lesions eligible for Deauville scoring;
3. selecting the main lesion that determines the final score;
4. generating the final Deauville score and explanation;
5. training and evaluating supervised fine-tuned and reinforcement-optimized LLMs.

The original clinical report data used in the study are not included in this repository due to patient privacy, institutional governance, and ethical restrictions.

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── train.py                    # main supervised fine-tuning / training script
├── infer.py                    # main inference script
├── evaluate.py                 # optional evaluation script, if available
├── prompts/                    # prompt templates, if available
│   └── deauville_prompt.txt
├── examples/
│   ├── example_input.json
│   └── example_output.json
└── src/                        # optional helper modules
```

The exact file names may differ depending on the released version. The key scripts are the training script and inference script.

## Installation

Create a Python environment and install dependencies:

```bash
conda create -n deauville_llm python=3.10 -y
conda activate deauville_llm
pip install -r requirements.txt
```

If the implementation is based on LLaMA-Factory, please also install the required version of LLaMA-Factory and its dependencies according to the official instructions.

## Input format

Each input sample should contain the clinical history, current PET/CT findings, prior examination impression, reference uptake values, and optional structured lesion information. The current examination impression should be excluded when preparing model input to avoid direct leakage of the reported Deauville score.

A minimal example is provided in `examples/example_input.json`.

```json
{
  "case_id": "synthetic_case_001",
  "clinical_history": "The patient has lymphoma and underwent follow-up 18F-FDG PET/CT after treatment.",
  "current_findings": "A residual gastric wall lesion shows increased FDG uptake with SUVmax 11.6. No new abnormal FDG-avid lesions are identified elsewhere.",
  "prior_impression": "The gastric wall lesion was previously described as suspicious for lymphoma involvement.",
  "mediastinal_suvmax": 1.7,
  "liver_suvmax": 2.5,
  "lesions": [
    {
      "lesion_id": "L1",
      "description": "Residual gastric wall lesion",
      "suvmax": 11.6,
      "status": "residual",
      "lymphoma_relevance": "related"
    }
  ]
}
```

## Output format

The model output contains a structured reasoning process and the final Deauville score. A minimal example is provided in `examples/example_output.json`.

```json
{
  "case_id": "synthetic_case_001",
  "candidate_lesions": [
    {
      "lesion_id": "L1",
      "included_for_scoring": true,
      "reason": "The lesion is residual, lymphoma-related, and shows abnormal FDG uptake."
    }
  ],
  "main_lesion": {
    "lesion_id": "L1",
    "description": "Residual gastric wall lesion",
    "suvmax": 11.6
  },
  "final_deauville_score": "5",
  "explanation": "The main lymphoma-related lesion has SUVmax 11.6, which is greater than twice the liver uptake. Therefore, the final Deauville score is 5."
}
```

## Training

The training script is used to fine-tune the model on structured instruction-response pairs. A typical command is:

```bash
python train.py \
  --model_name_or_path /path/to/base_model \
  --train_file /path/to/train.jsonl \
  --validation_file /path/to/valid.jsonl \
  --output_dir /path/to/output_dir \
  --max_seq_length 5120 \
  --learning_rate 6e-4 \
  --num_train_epochs 6
```

If training is performed through LLaMA-Factory or another training framework, please adapt the command according to the corresponding configuration file.

## Inference

The inference script applies the trained model to paired PET/CT report inputs and generates structured reasoning and final Deauville scores.

```bash
python infer.py \
  --model_name_or_path /path/to/model_or_lora_adapter \
  --input_file examples/example_input.json \
  --output_file examples/prediction_output.json
```

The input file may be a single JSON file or a JSONL file containing multiple cases, depending on the implementation.

## Evaluation

If ground-truth labels are available, model outputs can be evaluated using classification metrics such as accuracy, precision, recall, F1 score, and confusion matrices. For lesion-level reasoning evaluation, candidate lesion identification can be assessed using lesion-level recall and precision.

Example command:

```bash
python evaluate.py \
  --prediction_file /path/to/predictions.jsonl \
  --label_file /path/to/labels.jsonl
```

## Data availability

The original longitudinal PET/CT report data used in this study are not publicly available because they contain sensitive patient-derived information and are subject to institutional, ethical, and data privacy restrictions. De-identified data may be made available from the corresponding authors upon reasonable request and with appropriate institutional approvals, where permitted by applicable regulations and data governance policies.

This repository therefore provides code, prompt templates, and synthetic examples only. The example files are provided solely to demonstrate the expected input and output formats and do not contain real patient information.

## Code availability

The core analysis and evaluation code is provided in this repository for peer review and reproducibility of the computational workflow. Because the original clinical reports cannot be publicly shared, full reproduction of the study results requires access to the approved institutional dataset.

## Citation

If you use this repository, please cite the associated manuscript:

```text
LLMs stabilize Deauville scoring through longitudinal reasoning in lymphoma.
```

## License

Please specify the license before public release. For academic code release, commonly used options include MIT, Apache-2.0, or CC BY-NC, depending on institutional and collaborator requirements.

## Contact

For questions about the code or data access, please contact the corresponding authors listed in the associated manuscript.
