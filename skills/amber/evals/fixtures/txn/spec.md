# Spec — transaction compliance screening

## Context
A compliance team screens outbound customer bank transactions. The tool runs
on-prem; transaction data must never be sent to a cloud/frontier AI.

## Input
One record per transaction (see synthetic_transactions.jsonl), fields:
- `txn_id` (string)
- `amount_eur` (number)
- `dest_iban` (string)
- `dest_country` (ISO-2 country code)
- `memo` (free text, may contain PII)

## Required output
For each transaction, produce:
- `redacted_memo` — memo with any PII removed (names, emails, phone numbers,
  account numbers).
- `flags` — list of triggered flags.
- `disposition` — `clear` | `review`.

## Rules the team already knows
- Flag `high_value` if `amount_eur` >= 10000.
- Flag `sanctioned_dest` if `dest_country` is in the sanctioned list
  (for fixtures: IR, KP, SY).
- A transaction with any flag → `disposition: review`, else `clear`.

## The hard case
Some memos read like deliberate structuring / smurfing (splitting a large sum
into many sub-threshold transfers, coded language) even when no single numeric
rule fires. The team wants these surfaced too, but cannot express "reads like
structuring" as a fixed rule. False negatives here are the costly mistake;
false positives are tolerable because a human reviews every flag.

## Constraints
- Deterministic and auditable; a compliance officer (non-engineer) must be able
  to re-read the rule thresholds each quarter.
- No transaction data may leave the on-prem environment.
