# Error Loop Test — PLINK Commands

## Purpose

Verify that generated PLINK commands are checked before being accepted.

## Injected errors

1. Add a hallucinated PLINK flag: `--remove-bad-snps`.
2. Remove `--covar` from the Association Analysis command while retaining `--covar-name`.

## Validator result

Expected failure messages included:

- `Unknown PLINK flag: --remove-bad-snps`
- `--covar-name requires --covar`
- Association Analysis has study covariates but no `--covar`

## Revision loop

`Bad Commands → Command Validator FAIL → Groq Revision → Recheck`

## Final result

**PASS**

The revision removed the unsupported flag and restored the required covariate dependency.
