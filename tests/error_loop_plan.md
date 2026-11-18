# Error Loop Test — QC Plan

## Purpose

Verify that the Planning Agent is not a one-shot generator and can revise a plan after rule-based validation fails.

## Injected errors

1. Remove the required **Relatedness** step.
2. Replace the source ID of the first step with `尚待驗證`.

## Validator result

Expected failure messages included:

- `Missing required step: Relatedness.`
- invalid source format for `尚待驗證`

## Revision loop

`Bad QC Plan → Plan Validator FAIL → Groq Revision → Recheck`

## Final result

**PASS**

The revised plan restored the missing required step and replaced the invalid source with an allowed Wiki Source ID.
