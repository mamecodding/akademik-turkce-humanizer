# Local Edit Pilot v1

This package contains a 20-sample feasibility set for testing constrained local edits instead of full-paragraph rewriting.

## Composition
- 3 legacy real-test blocks
- 17 production-validation records
- 5 development samples / 15 evaluation samples

## Rules
1. Do not use `reference_text` to generate edit proposals. It is evaluation context only.
2. A proposal may contain at most 3 local edits per paragraph.
3. No edit may overlap a `protected_spans` range.
4. Total inserted + deleted words should be <=20% of input word count.
5. New numbers, citations, dates, names, technical claims or examples are not allowed.
6. Rejected proposals or unchanged fallbacks do not count as successful transformations.
7. `proposals.jsonl` is intentionally empty until proposals are generated under a separately approved inference step.

## Protected spans
Regex extraction covers numbers/measures, dates, common citation forms, structural references and direct quotes. Proper-name and technical-term coverage is intentionally not claimed as complete. Every sample therefore retains `review_status: pending` and requires human review before any product-level conclusion.

## Review file
`reviews.csv` contains one blank review row per sample. Reviewers should judge preservation of propositions, bindings between claims and citations/numbers, scope/negation/causality/certainty, naturalness, and usefulness.

## Success thresholds for the later 15-sample evaluation
- 15/15: zero changes/additions/deletions to protected factual elements
- 15/15: zero added, deleted or meaning-changed propositions by human review
- >=12/15: naturalness improved
- >=12/15: safe and useful non-trivial edit produced

These are pilot thresholds, not proof of production reliability.
