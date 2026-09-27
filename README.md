# DSC2026 – Team UIT-IntelliJQK

The official repository of team IntelliJQK for the UIT-DSC 2026 competition Subtask 1: Legal Information Retrieval (LegalIR) and Subtask 2: Legal Question Answering (LegalQA). It contains the source code, models, and instructions to reproduce the results for the competition's tasks.

## 1. Method Overview

Main repository structure:

| File / Directory | Function |
| :--- | :--- |
| `data/raw/` | Initial data provided by the Organizers |
| `data/processed/` | Data after running preprocessing |
| `logs/` | Directory containing execution logs of the processes |
| `outputs/` | Output data after running the notebooks |
| `requirements_task1.txt` | Required Python libraries for task 1 |
| `requirements_task2.txt` | Required Python libraries for task 2 |
| `ensemble_rrf.ipynb` | Post-processing: Cross-ensemble the outputs of the 2 retrieval models (BCE and Listwise versions) using the RRF algorithm to optimize the final result list |
| `7task2_vileg17.ipynb` | Train the text generation model (ViLegalQwen3-1.7B) to automatically generate answers (QA) from the retrieved contexts (Task 2) |
| `dsc2026_77.ipynb` | Train the document retrieval model (Task 1) using Pointwise BCE loss (scoring each question-document pair independently) |
| `Dsc2026_7list33.ipynb` | Train the document retrieval model (Task 1) using Listwise loss (directly optimizing the ranking order of a group of documents) |

## 2. Source Code Publication

### Task 1: Legal Information Retrieval
All code in this repository is self-developed by our team for the DSC 2026 Task 1 round. We commit to not using any data outside the permitted scope of the organizers and to having no fraudulent behavior during the training/evaluation process.

**The pipeline consists of 3 notebooks:**
- `dsc2026_77.ipynb`: Hybrid Retrieval (BM25 + Dense) + fine-tune reranker with **pointwise BCE** loss.
- `Dsc2026_7list33.ipynb`: Same pipeline, fine-tune reranker with **listwise softmax** loss.
- `ensemble_rrf.ipynb`: Combine the results of the 2 notebooks above using **Reciprocal Rank Fusion (RRF)** to produce the final submission.

**Pretrained models used:**
- Dense embedding: [AITeamVN/Vietnamese_Embedding_v2](https://huggingface.co/AITeamVN/Vietnamese_Embedding_v2)
- Reranker (cross-encoder): [BAAI/bge-reranker-v2-m3](https://huggingface.co/BAAI/bge-reranker-v2-m3)

Fine-tuned using LoRA (not full fine-tune) on the training data provided by the organizers (no full fine-tune, no additional external data used).

**Reproducibility:**
- Fixed seed `SEED = 42` for the entire pipeline.
- Time-consuming steps are cached to disk to avoid recalculating from scratch: short-chunked corpus, BM25 index, candidate pool mining (positive/hard-negative), tuned weights, and Stage-2 filter.
- Weights (hybrid BM25 + dense) and Stage-2 filtering threshold are fully tuned on the **dev set split from train**, without touching the public/private sets.
- The RRF constant `k=60` during ensembling is a standard value common in literature, not a parameter tuned on the test set.

*Known limitations:* BM25/hybrid results may slightly vary between runs due to the dependency on the `underthesea` tokenization library version; reranker fine-tuning has minor randomness from dropout/LoRA initialization despite the fixed seed.

### Task 2: Legal Question Answering
All code in this repository is self-developed by our team for the DSC 2026 Task 2 round. We commit to not using any data outside the permitted scope of the organizers and to having no fraudulent behavior during the training/evaluation process.

**Pretrained models used:**
- Dense embedding: [AITeamVN/Vietnamese_Embedding_v2](https://huggingface.co/AITeamVN/Vietnamese_Embedding_v2)
- Reranker (cross-encoder): [BAAI/bge-reranker-v2-m3](https://huggingface.co/BAAI/bge-reranker-v2-m3)
- Answer generation (Stage-3, Task 2): [ntphuc149/ViLegalQwen3-1.7B-Base](https://huggingface.co/ntphuc149/ViLegalQwen3-1.7B-Base)

Fine-tuned using LoRA (not full fine-tune). Fixed seed `SEED=42` to ensure reproducibility; time-consuming steps (chunking, mining) are cached to disk. Ensure the environment meets the requirements for correct reproduction.

## 3. Environment and Hardware

**Hardware used:** 
The training and inference processes were performed on **NVIDIA RTX A4000** and **NVIDIA RTX A5000** GPUs.

**Environment:**
Running the source code requires **2 separate Python environments** to avoid library conflicts. Strictly do not run them together in the same environment.

## 4. Execution Guide

### Run Task 1

```bash
cd DSC2026-IntelliJQK

# Create a separate environment for task 1
python -m venv venv_task1
source venv_task1/bin/activate # Windows: venv_task1\Scripts\activate

# Install libraries
pip install -r requirements_task1.txt

# Pre-download models from Hugging Face (since the notebook runs in an OFFLINE environment - 
# if network is enabled, the notebook will auto-download during from_pretrained()/SentenceTransformer(), no manual step needed)
hf download AITeamVN/Vietnamese_Embedding_v2 --local-dir models/Vietnamese_Embedding_v2
hf download BAAI/bge-reranker-v2-m3 --local-dir models/bge-reranker-v2-m3

# Data preprocessing 
git clone [https://github.com/anhduc1526/DSC-preprocessing.git](https://github.com/anhduc1526/DSC-preprocessing.git)
python DSC-preprocessing/main.py

# Create logs directory (if not exists)
mkdir -p logs

# Convert the 2 notebooks to Python scripts
cd notebook
jupyter nbconvert --to script "dsc2026_77.ipynb"
jupyter nbconvert --to script "dsc2026_7list33.ipynb"
cd ..

# Run notebook 1 (BCE loss) in the background, logging to a separate file
nohup python -u notebook/dsc2026_77.py > logs/dsc2026_77.log 2>&1 &

# Wait for notebook 1 to finish (monitor logs) before running notebook 2 (listwise loss), 
# to avoid VRAM contention if running simultaneously on 1 GPU:
tail -f logs/dsc2026_77.log
nohup python -u notebook/dsc2026_7list33.py > logs/dsc2026_7list33.log 2>&1 &
tail -f logs/dsc2026_7list33.log

# After both notebooks have generated submission_77.json and submission_7list33.json 
# in the outputs/ directory, run ensemble using RRF (rrf_k = 60):
jupyter nbconvert --to script "notebook/ensemble_rrf.ipynb"
python -u notebook/ensemble_rrf.py

### Run Task 2
```bash
cd DSC2026-IntelliJQK

# Create a separate environment for task 2
python -m venv venv_task2
source venv_task2/bin/activate # Windows: venv_task2\Scripts\activate

# Pre-download models from Hugging Face (since the notebook runs in an OFFLINE environment - 
# if network is enabled, the notebook will auto-download during from_pretrained()/SentenceTransformer(), no manual step needed)
hf download AITeamVN/Vietnamese_Embedding_v2 --local-dir models/Vietnamese_Embedding_v2
hf download BAAI/bge-reranker-v2-m3 --local-dir models/bge-reranker-v2-m3
hf download ntphuc149/ViLegalQwen3-1.7B-Base --local-dir models/ViLegalQwen3-1.7B-Base

# Install libraries
pip install -r requirements_task2.txt

# (If using the pre-trained reranker checkpoint from Task 1, copy/symlink to the exact 
# FT_RERANKER_DIR path read by the task 2 notebook, for example:)
cp -r <path_to_reranker_checkpoint_from_Task_1> checkpoints/bge-reranker-v2-m3-ft

# Create logs directory (if not exists)
mkdir -p logs

# Convert notebook to Python script
cd notebook
jupyter nbconvert --to script "7task2_vileg17.ipynb"
cd ..

# Run notebook in the background, logging to a separate file
nohup python -u notebook/7task2_vileg17.py > logs/7task2_vileg17.log 2>&1 &
# (name the log 7task2_vileg17.log to avoid overwriting existing logs)

# Monitor the process
tail -f logs/7task2_vileg17.log
