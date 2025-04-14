# InstructLabCICD
Complete CICD pipeline to update LLMs when new QnA added, train the model and publish the model with updated SDG to be served.

# MLOps + DevOps Workflow Plan for Self-Hosted LLM using InstructLab

## Overview
This document outlines a complete MLOps and DevOps pipeline for building a self-hosted LLM fine-tuned using InstructLab. The pipeline automates model training, synthetic data generation, evaluation, and deployment using GitHub Actions, AWS EC2 runners, and custom scripts. The project is designed to be updated continuously via GitOps workflows and integrates evaluation and version control mechanisms.

---

## 1. Repository Structure

- **`taxonomy/`** – Forked from InstructLab. Holds `skills` and `knowledge` QnA YAMLs.
- **`dev-ml-ops/`** – Custom repo for CI/CD pipelines, EC2 orchestration, training configs, and evaluation.
- _(Optional)_ **`improvement-notes/`** – Future RAG, GGUF export enhancements, monitoring plans, etc.

---

## 2. CI/CD + GitOps Workflow

### Trigger:
- Changes pushed to `taxonomy` repo (`qna.yaml` updates)

### GitHub Actions:
1. **Detect changes in taxonomy QnA**
2. **Spin up EC2 Runner with CUDA support**
3. **Run model training using InstructLab Training Library**
4. **Generate Synthetic Data (SDG)** and store:
   - `s3://llm-models-bucket/sdg/version-x/`
5. **Run Evaluation:**
   - `MT-Bench`, `MMLU`, and `Branch Tests`
   - Store evaluation scores (CSV/JSON)
6. **Convert to GGUF (optional)**
   - Prepares model for quantized CPU/GPU use
7. **Model Registry Update (version tagging)**
8. **Shut down EC2 Runner**
9. **Notify via Slack/Webhook/email (optional)**

---

## 3. Training Pipeline

Using `instructlab-training`:

```python
from instructlab.training import run_training, TorchrunArgs, TrainingArgs
```

- `model_path`: base LLM (e.g. `ibm-granite/granite-7b-base`)
- `data_path`: generated `messages-format` dataset
- `ckpt_output_dir`: final checkpoints
- `data_output_dir`: intermediate processed data
- `num_epochs`, `batch_size`, `learning_rate`, etc.

Support for **LoRA**, **DeepSpeed**, and **FSDP** available.

---

## 4. Synthetic Data Generation (SDG)
- Auto-generated during training
- Saved separately for reuse or future retraining
- Helps prevent **catastrophic forgetting** by maintaining training history

---

## 5. Evaluation Pipeline

Run using the `eval/` repo:

- `MT-Bench` – tests skill learning (multi-turn QnA)
- `MMLU` – tests factual knowledge (multiple-choice)
- `Branch Bench` – tests on user-contributed QnAs
- Evaluation scores are:
  - Stored in `eval_output/`
  - Compared with previous runs

Optional:
- Visualized via **MLflow**, **TensorBoard**, or **custom Streamlit dashboard**

---

## 6. Model Serving

Once trained, models are:
- Optionally converted to **GGUF** format
- Deployed on:
  - Local LLM inference server (vLLM, llama.cpp)
  - API endpoints
- Integrated into internal chatbots or apps

---

## 7. Infra Overview

- **Training**: On-demand **AWS EC2 GPU** instance
- **Artifacts & Data**: Stored in **S3** (or Azure Blob)
- **CI/CD**: GitHub Actions + EC2 runner registration
- **Serving**: Lightweight server for **latest trained model**
- **Monitoring**: (Optional) Evaluation dashboards + Prometheus/Grafana

---

## 8. Future Enhancements

- **RAG** integration (Retrieval-Augmented Generation)
- **Containerized LLM** (Docker + llama.cpp or Ollama)
- **Scheduled retraining**
- **Slack/Teams chatbot using the model**
- **Custom UI for contributions and evaluation**

---

## Summary
This plan provides a production-grade, automated pipeline to support LLM development using InstructLab. It supports continuous updates, auto-evaluation, version control, and internal deployment of the latest models. All infrastructure is controlled using GitHub workflows and kept modular for future upgrades.


