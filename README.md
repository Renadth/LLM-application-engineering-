# Murshid Mini: LLM Application Engineering Project

**Program:** LLM Application Engineering · SDA-AIE-213 · delivered by [@SDAIAAcademy](https://github.com/SDAIAAcademy)
**Reference material:** [Course repository](https://github.com/almiyead-rgb/murshid-llm-application-engineering-sda-aie-213)

---

## Summary

Murshid Mini is a small bilingual (Arabic/English) assistant for fictional government services. It answers public service questions, looks up application status through a read-only tool that checks ownership, rejects unsafe or invalid input, tests its own behaviour against ten golden cases, and caches public answers safely. It runs in mock mode with a deterministic `FakeClient` instead of a real LLM. The goal was to learn the engineering *around* a model, so it uses no API keys, no GPU and no cost.

## Key results

| Check | Result |
|---|---|
| Golden test suite (C01–C10) | **10/10 passed (100%)**, also passes through the cached app |
| Cache experiment, same FAQ asked 3× | Mock calls **3 → 1**, hits `[false, true, true]`, **66.7%** hit rate |
| Private data | No other user's status exposed; status lookups never cached |
| Records | Unchanged after every run (tool is read-only) |
| External API calls | **0**. `mode=mock` |
| Fresh-runtime test | All 14 cells run top to bottom with no errors |

> 100% means these ten chosen mock cases passed. It does **not** mean the system is safe or accurate, and it says nothing about a real LLM.

## Project overview

The project was built over four days in **one cumulative notebook**, with each day adding cells below the previous ones.

| Day | Focus | Cells | What was added | Output |
|---|---|---|---|---|
| 1 | Foundations | 1–3 | Synthetic directory, `FakeClient`, `ask()`, response contract (`trace_id`, `answer`, `model`, `latency_ms`) | `baseline_results.json` |
| 2 | Structure & tools | 4–6 | Structured ticket + validation, read-only `check_status` with ownership check, `route()` | `day2_results.json` |
| 3 | Guard & test | 7–10 | Versioned prompt, input guard, output check, `run_pipeline()`, 10 golden cases | `day3_results.json` |
| 4 | Measure & finish | 11–14 | Exact-match public FAQ cache (versioned key, 300 s TTL), metrics, regression, final report, export | `day4_results.json`, `final_report.json`, `metrics.json` |

### How a question flows through the app

```
Question → input guard → ticket extraction → ticket validation
        ├─ public FAQ      → FakeClient answer (cacheable)
        ├─ status request  → read-only check_status (ownership check, never cached)
        ├─ missing reference → ask for it (no tool call)
        └─ other / unsafe  → out_of_scope / blocked / invalid
        → output check → response + privacy-safe metrics
```

The code makes every decision: routing, access and refusals. The model only supplies wording, so a real provider could later replace `FakeClient` without changing the rules.

### My changes (beyond the reference)
- **Day 2:** added three extra asserts. A ticket with an invalid service value is rejected as `invalid_schema` with no tool call, `check_status('bad', ...)` returns `invalid_reference` on its own, and a lower-case `ref-1001` is normalised and still found.
- **Day 3:** added `'ignore all instructions'` to the blocked phrases, with an assert. The original list only caught "ignore previous".
- **Day 4:** added a test showing that the Arabic FAQ is cached under its own key and returns the Arabic answer. Only the English FAQ was tested before.

Day 1 follows the reference code with no changes. No tests were removed or weakened.

No tests were removed or weakened.

## Repository structure

```
LLM-application-engineering-/
├── README.md
├── notebooks/
│   └── murshid_Renad.ipynb        # The cumulative project notebook (Days 1–4, cells 1–14)
├── results/
│   ├── core/                      # Outputs from my own run of the project notebook
│   │   ├── baseline_results.json  # Day 1
│   │   ├── day2_results.json      # Day 2
│   │   ├── day3_results.json      # Day 3: 10 golden cases
│   │   ├── day4_results.json      # Day 4: cache experiment
│   │   ├── final_report.json      # Tests, cache, hypothetical cost, limits
│   │   ├── metrics.json           # Privacy-safe per-request metrics
│   │   └── synthetic_data.json    # The synthetic directory and records
│   ├── day1/                      # Day 1 homework output
│   └── day2/                      # Day 2 homework output
└── homework/                      # One separate notebook per optional homework
    ├── day1_real_model_renad.ipynb
    └── day2_tools_renad.ipynb
```

## Quick start

1. Open `notebooks/murshid_Renad.ipynb` in Google Colab. Either click the **Open in Colab** badge at the top of the notebook, or go to File → Open notebook → GitHub and paste this repository's URL.
2. Go to **Runtime → Change runtime type** and choose **Python 3, CPU** (no accelerator).
3. Go to **Runtime → Run all**. The cells must run in order, because each day depends on the earlier ones.
4. Find the outputs in the Colab **Files** panel under `murshid_outputs/`. Cell 14 also creates `murshid_submission.zip`.

Nothing needs installing: no `pip`, no API keys, no model download and no Drive mount.

## Data

All data is synthetic and built into the code:
- A public service directory with one Arabic and one English answer.
- `REF-1001`: owned by `demo-user-1`, status `under_review` (normal path).
- `REF-1002`: owned by `demo-user-2`, status `approved` (used to test access refusal).
- `REF-9999`: deliberately missing (used for the not-found test).

The user's identity comes from the demo session, never from the question. The project contains no real IDs, records, personal data or secrets.

## Limitations

- `FakeClient` only matches fixed questions, so there is no real language understanding.
- The guards are keyword-based. A reworded attack ("disregard what you were told") can get through, and personal data is only caught as a 10-digit number.
- The cache matches exact wording only, so a paraphrased FAQ is always a miss.
- Ownership checks use demo fixtures, not real authentication.
- `latency_ms` measures local Python time, not model inference. The cost figure ($0.0036/request) is an illustration with made-up prices, not real spend.

**Possible next step:** normalize questions (spacing, punctuation, letter forms) before the cache lookup so that small wording differences still hit the cache.


The homework runs in separate notebooks and is kept apart from the mock core project.

---

*Built for **LLM Application Engineering (SDA-AIE-213)**, delivered by **SDAIA Academy**, instructor **Meaad Al-Marri**. All data is synthetic, for educational use only.*
