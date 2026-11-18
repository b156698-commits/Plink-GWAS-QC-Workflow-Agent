# Case D — Basic QC only

## Input

- Study design: Case-control GWAS
- Data format: PLINK BED/BIM/FAM
- Sample size: 3000
- Phenotype: Disease Status
- Covariates: Age, Sex
- Association method: Logistic Regression
- Need PCA: False
- Need Relatedness Check: False

## Expected workflow

`Load PLINK Binary Files → Variant Missingness → Sample Missingness → MAF Filtering → HWE Filtering → Association Analysis`

## Actual result

- Plan Check: **PASS**
- Plan revisions: **0**
- Command Check: **PASS**
- Command revisions: **0**
- Workflow behavior: **consistent with the conditional workflow design**
- Recorded runtime: **1 m 28 s**
- Final result: **PASS**

## Evidence

![Case D — Basic QC only](assets/case_D_basic.png)

## Interpretation

This case verifies that the Agent changes the QC workflow according to the PCA and Relatedness requirements instead of always returning one fixed template.
