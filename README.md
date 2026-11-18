# PLINK / GWAS QC Workflow Agent

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

## Running the final app in Google Colab

1. Upload/open `notebooks/05_final_gradio_app_v5_rate_limit_safe.ipynb`.
2. Add a Colab Secret named `Groq`.
3. Enable notebook access for the secret.
4. Run all cells.
5. Open the Gradio interface.
6. Enter study information and select **Run Workflow**.

The public `gradio.live` URL is temporary. A permanent deployment can be added after final testing.

## Current project status

**MVP development: complete.**  
**Normal-case evaluation: complete.**  
**Injected FAIL → revision → PASS evaluation: complete.**

Remaining work is mainly project packaging, final deployment, and presentation/report preparation.
