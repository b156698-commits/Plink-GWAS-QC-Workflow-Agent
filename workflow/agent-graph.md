# PLINK / GWAS QC Workflow Agent — Graph

```mermaid
flowchart TD
    A[User Study Information] --> B[RAG Retrieval]
    B --> C[Retrieval Coverage Check]
    C --> D[QC Planning Agent]
    D --> E[Rule-based Plan Validator]

    E -->|FAIL| F[Revise QC Plan]
    F --> E
    E -->|PASS| G[PLINK Command Generator]

    G --> H[Rule-based Command Validator]
    H -->|FAIL| I[Revise PLINK Commands]
    I --> H
    H -->|PASS| J[Final Report]
    J --> K[Gradio Web App Output]

    L[(PLINK / GWAS Wiki)] --> B
```

## Node responsibilities

- **RAG Retrieval**: retrieves relevant Wiki chapters and Source IDs.
- **Retrieval Coverage Check**: ensures study-required topics are present before planning.
- **QC Planning Agent**: creates a study-specific QC workflow.
- **Plan Validator**: checks required steps, ordering, source IDs, threshold wording, and LD-pruning logic.
- **Command Generator**: converts the verified QC plan into PLINK command examples.
- **Command Validator**: checks supported flags, parameter dependencies, association method, `--bfile`, `--out`, and misuse of LD-pruned data.
- **Revision nodes**: receive validator errors and regenerate only when needed.
- **Final Report**: summarizes the study settings, workflow, validation status, and knowledge sources.
