# Quality report - Lab Day 10

**run_id (bad/inject):** `sprint3-bad`  
**run_id (good/clean):** `sprint2-smoke-conda`  
**date:** `2026-04-15`

---

## 1. Summary metrics

| Metric | Before (inject bad) | After (clean run) | Notes |
|--------|---------------------|-------------------|-------|
| raw_records | 10 | 10 | Same source export |
| cleaned_records | 6 | 6 | Cleaning scope unchanged |
| quarantine_records | 4 | 4 | Duplicate, empty row, stale HR row, unknown doc_id |
| text_repaired_count | 0 | 0 | No mojibake repair triggered on this sample export |
| exported_at_normalized_count | 6 | 6 | All cleaned rows normalized to ISO-8601 UTC |
| future_effective_date_count | 0 | 0 | No future-dated rows in this sample |
| Expectation halt? | Yes | No | `refund_no_stale_14d_window` fails only in inject run |

Evidence files:

- `artifacts/logs/run_sprint3-bad.log`
- `artifacts/logs/run_sprint2-smoke-conda.log`
- `artifacts/cleaned/cleaned_sprint3-bad.csv`
- `artifacts/cleaned/cleaned_sprint2-smoke-conda.csv`

---

## 2. Before / after quality evidence

Primary corruption scenario: disable the refund-window cleaning rule by running:

```bash
conda run -n vinai python etl_pipeline.py run --run-id sprint3-bad --no-refund-fix --skip-validate
```

Observed effect in the bad run:

- `expectation[refund_no_stale_14d_window] FAIL (halt) :: violations=1`
- cleaned output still contains the stale refund text `14 ngay lam viec`

Observed effect in the clean run:

- `expectation[refund_no_stale_14d_window] OK (halt) :: violations=0`
- cleaned output rewrites stale refund text to `7 ngay lam viec` and appends `[cleaned: stale_refund_window]`

Concrete cleaned CSV difference:

- Bad run file `artifacts/cleaned/cleaned_sprint3-bad.csv` contains one refund row with `14 ngay lam viec`
- Good run file `artifacts/cleaned/cleaned_sprint2-smoke-conda.csv` contains the corrected refund row with `7 ngay lam viec`

HR versioning remained stable in both runs:

- stale 2025 HR row stayed quarantined
- cleaned HR row kept the 2026 policy value `12 ngay phep nam`
- `expectation[hr_leave_no_stale_10d_annual] OK (halt) :: violations=0`

---

## 3. Retrieval / embed status

The intended next step was to run `eval_retrieval.py` for before/after retrieval CSV evidence on top of Chroma.

Current blocker:

- `chromadb.PersistentClient(...)` fails in the `vinai` environment with `disk I/O error`
- this is a storage/runtime issue in Chroma, not a cleaning/expectation logic failure

Impact:

- quality evidence for Sprint 3 is available through cleaned CSV diff and expectation logs
- retrieval CSV evidence is pending until the Chroma storage issue is fixed by the embed owner

Recommended follow-up after Chroma is healthy:

1. Run the bad/inject pipeline again with the same `--no-refund-fix --skip-validate` flags.
2. Run `python eval_retrieval.py --out artifacts/eval/after_inject_bad.csv`.
3. Run the clean pipeline normally.
4. Run `python eval_retrieval.py --out artifacts/eval/after_clean_fix.csv`.

---

## 4. Corruption inject summary

Inject type used in Sprint 3:

- intentionally skip the stale refund cleaning rule
- keep validate disabled so the bad row can reach the publish boundary during demo

How it is detected:

- expectation suite catches the stale refund policy before publish in normal mode
- cleaned CSV diff shows the stale `14 ngay` row directly
- once Chroma storage is fixed, retrieval should degrade for `q_refund_window`

---

## 5. Limits / unfinished work

- Retrieval CSV evidence could not be generated yet because Chroma storage is failing with `disk I/O error`.
- `text_repaired_count` stayed `0` on the sample export, so mojibake repair still needs an injected case to demonstrate measurable impact.
- `future_effective_date_count` stayed `0` on the sample export, so that rule should be demonstrated with a synthetic future-dated row in Sprint 3 extension.
