# PLINK / GWAS QC Workflow Agent

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/b156698-commits/Plink-GWAS-QC-Workflow-Agent/blob/main/notebooks/05_final_gradio_app_v5_rate_limit_safe.ipynb)

A teaching-oriented AI Agent that uses a PLINK / GWAS Wiki as a RAG knowledge source to build and validate a study-specific GWAS quality-control workflow, then generate and validate PLINK command examples.

## Project goal

The system is designed for beginner-friendly GWAS workflow planning. It does **not** simply ask an LLM to produce commands once. Instead, it separates knowledge retrieval, planning, deterministic validation, revision, command generation, and final reporting.

## Main workflow

```text
Study Information
→ RAG Retrieval
→ Retrieval Coverage Check
→ QC Planning
→ Plan Validator
→ FAIL → Revise → Recheck
→ PASS
→ PLINK Command Generator
→ Command Validator
→ FAIL → Revise → Recheck
→ PASS
→ Final Report
→ Gradio UI
```

See [`workflow/agent-graph.md`](workflow/agent-graph.md) for the Mermaid graph.

## Current supported study configuration

The current MVP focuses on:

- Case-control GWAS
- PLINK BED/BIM/FAM input
- Logistic association analysis
- Optional PCA
- Optional relatedness checking
- Covariates such as Age and Sex

## Conditional workflow

The Agent changes the workflow according to the study settings:

| PCA | Relatedness | Conditional steps |
|---|---|---|
| Yes | Yes | LD Pruning → Relatedness → PCA |
| Yes | No | LD Pruning → PCA |
| No | Yes | LD Pruning → Relatedness |
| No | No | No LD Pruning / Relatedness / PCA requirement |

LD pruning is used to prepare a relatively LD-independent SNP subset for PCA and/or relatedness. The final association analysis is not automatically restricted to only the LD-pruned subset.

## Repository structure

```text
.
├── notebooks/
│   ├── 01_state_and_plan_validator.ipynb
│   ├── 02_rag_retrieval.ipynb
│   ├── 03_planning_agent_with_groq_v3_stable.ipynb
│   ├── 04_command_generator_and_validator.ipynb
│   └── 05_final_gradio_app_v5_rate_limit_safe.ipynb
├── wiki/
├── raw/
├── tests/
│   ├── assets/
│   ├── case_A_pca_relatedness.md
│   ├── case_B_pca_only.md
│   ├── case_C_relatedness_only.md
│   ├── case_D_basic.md
│   ├── error_loop_plan.md
│   └── error_loop_command.md
├── workflow/
│   └── agent-graph.md
├── requirements.txt
└── README.md
```

## Notebook stages

### 01 — State + Plan Validator
Defines the study state and deterministic validation rules.

### 02 — RAG Retrieval
Loads Wiki material, creates embeddings, builds the FAISS index, and retrieves relevant PLINK / GWAS knowledge.

### 03 — Planning Agent
Uses Groq to generate a structured QC plan, then checks it with the Plan Validator. Failed plans are sent back for revision.

### 04 — Command Generator
Converts the verified plan into PLINK commands and validates flags, dependencies, and study-specific command logic.

### 05 — Final Gradio App
Combines retrieval, planning, validation loops, command generation, final reporting, progress feedback, and Groq rate-limit handling into one interface.

## Evaluation

Four normal conditional-workflow cases were tested:

| Case | PCA | Relatedness | Plan Check | Command Check |
|---|---:|---:|---|---|
| A | Yes | Yes | PASS | PASS |
| B | Yes | No | PASS | PASS |
| C | No | Yes | PASS | PASS |
| D | No | No | PASS | PASS |

All four recorded runs completed with **0 plan revisions** and **0 command revisions**.

Two additional injected-error tests demonstrate the actual Agent loops:

1. **Plan error test**
   - Missing Relatedness
   - Invalid Source ID
   - Validator FAIL → Groq revision → PASS

2. **Command error test**
   - Hallucinated `--remove-bad-snps`
   - Missing `--covar`
   - Validator FAIL → Groq revision → PASS

See the files under [`tests/`](tests/).

## Knowledge base

The `wiki/` folder contains the normalized teaching chapters used by RAG. The `raw/` folder records the ingested source material and source manifest.

Important design principle: QC thresholds shown by the system are examples rather than universal rules. Study design, sample properties, ancestry, genotyping platform, and analysis goals may require different settings.

## Run the final app in Google Colab

The recommended way to run the final project is Google Colab.

### 1. Open the final notebook

Click the badge at the top of this README, or open:

[`notebooks/05_final_gradio_app_v5_rate_limit_safe.ipynb`](notebooks/05_final_gradio_app_v5_rate_limit_safe.ipynb)

### 2. Add the Groq API key to Colab Secrets

In Colab:

1. Open the **Secrets** panel in the left sidebar.
2. Create a new secret named:

```text
Groq
```

3. Paste your Groq API key as the value.
4. Enable **Notebook access** for that secret.

> Do not paste the API key directly into the notebook or GitHub repository.

### 3. Run the notebook

Use:

```text
Runtime → Run all
```

The notebook will:

```text
Load the Wiki
→ Build embeddings and FAISS index
→ Start the RAG pipeline
→ Build the QC Planning Agent
→ Enable Plan / Command validation loops
→ Launch the Gradio Web App
```

### 4. Use the Gradio interface

Enter the study settings and click:

```text
Run Workflow
```

The interface shows:

- Retrieved Knowledge
- QC Plan
- Plan Check
- PLINK Commands
- Command Check
- Final Report

The workflow also displays progress while retrieval, planning, validation, and command generation are running.

### 5. Gradio link note

When the notebook is launched with `share=True`, Gradio may provide a public URL such as:

```text
https://xxxxx.gradio.live
```

This URL is **temporary** and is intended for demonstration/testing.  
The permanent project source, Wiki, test evidence, and workflow documentation are stored in this GitHub repository.

## Recommended demo procedure

For a presentation or grading demo:

1. Open the final notebook in Colab.
2. Add the `Groq` secret.
3. Run all cells.
4. Open the generated Gradio interface.
5. Demonstrate one normal case.
6. Show the stored FAIL → revision → PASS evidence under `tests/`.

The four normal test cases and the injected-error loop tests are already documented in the repository, so the live demo does not need to rerun every case.

## Current project status

**Deployment strategy:** GitHub for permanent project storage + Google Colab / Gradio for executable demo.


**MVP development: complete.**  
**Normal-case evaluation: complete.**  
**Injected FAIL → revision → PASS evaluation: complete.**

Remaining work is mainly project packaging, final deployment, and presentation/report preparation.
