# Cleaning & Quality Sprint 3-4 Handoff

Owner: Cleaning / Quality

## Sprint 3 output

Completed:

- created one intentional bad run: `sprint3-bad`
- kept one clean comparison run: `sprint2-smoke-conda`
- captured expectation evidence in logs
- captured before/after cleaned CSV files

Key evidence to cite:

- bad log: `artifacts/logs/run_sprint3-bad.log`
- good log: `artifacts/logs/run_sprint2-smoke-conda.log`
- bad cleaned CSV: `artifacts/cleaned/cleaned_sprint3-bad.csv`
- good cleaned CSV: `artifacts/cleaned/cleaned_sprint2-smoke-conda.csv`

Most important finding:

- the refund cleaning rule is material
- when disabled, `refund_no_stale_14d_window` fails with `violations=1`
- when enabled, the same check passes with `violations=0`

## Metric impact table rows for group report

| Rule / Expectation new | Before | After / when injected | Evidence |
|---|---|---|---|
| `exported_at_iso8601_for_all_rows` | no explicit guarantee in baseline logs | passes with `non_iso_exported_at_rows=0` in clean and bad runs after normalization | `run_sprint2-smoke-conda.log`, `run_sprint3-bad.log` |
| `no_mojibake_artifacts_in_chunk_text` | no explicit signal before Sprint 2 | warning metric now exists and shows `mojibake_rows=0` on sample data | `run_sprint2-smoke-conda.log` |
| `refund_no_stale_14d_window` as inject evidence | clean run has `violations=0` | bad run has `violations=1` when refund fix is disabled | both run logs + cleaned CSV diff |
| `normalize_exported_at_iso8601` | raw export timestamps were not normalized | `exported_at_normalized_count=6` on both runs | both run logs |

## Text for runbook

Symptom:

- agent or reviewer sees refund guidance mentioning `14 ngay` instead of `7 ngay`

Detection:

- `expectation[refund_no_stale_14d_window] FAIL`
- cleaned CSV contains stale refund text
- retrieval `hits_forbidden=yes` should appear once Chroma storage is working again

Diagnosis:

1. Open the latest run log and check refund expectation status.
2. Open the matching cleaned CSV and search for `14 ngay`.
3. Confirm whether the pipeline was run with `--no-refund-fix`.

Mitigation:

- rerun the pipeline without `--no-refund-fix`
- do not publish stale embed artifacts from demo/inject runs

Prevention:

- keep `refund_no_stale_14d_window` at `halt`
- retain cleaned CSV diff in artifacts for peer review

## Sprint 4 handoff to docs owner

Please copy from `docs/quality_report.md` into:

- `reports/group_report.md` section 2 for metric impact
- `reports/group_report.md` section 3 for inject narrative
- `docs/runbook.md` for refund stale incident example

Current blocker to note:

- retrieval CSV could not be generated yet because Chroma in the `vinai` environment fails with `disk I/O error`
- this does not invalidate the cleaning evidence, but it blocks the embed-based before/after screenshot
