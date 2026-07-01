# Spec — patient intake validation & normalization

## Context
Validates patient intake records at a hospital. Runs entirely inside the
hospital environment; patient data must never leave it.

## Input
One record per intake (see synthetic_intake.jsonl), fields:
- `patient_id` (string)
- `dob` (date, mixed formats in raw data)
- `height` (string, mixed units e.g. "180cm", "5ft11")
- `weight` (string, mixed units e.g. "80kg", "176lb")
- `systolic`, `diastolic` (numbers, mmHg)
- `heart_rate` (number, bpm)

## Required output
For each record:
- `valid` (bool) and `errors` (list) — schema validation.
- normalized fields: `dob_iso` (YYYY-MM-DD), `height_cm`, `weight_kg`.
- `flags` — out-of-range vitals per thresholds below.

## Rules
- Required fields must be present and parseable, else `valid: false`.
- Normalize dob to ISO, height to cm, weight to kg.
- Flag `bp_high` if systolic >= 140 or diastolic >= 90.
- Flag `hr_abnormal` if heart_rate < 40 or > 120.

## Constraints
- Fully deterministic and reproducible.
- Thresholds re-audited each cycle by clinical staff (non-engineers).
- No patient data leaves the hospital environment.
