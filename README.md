# Murshid Mini — LLM Application Engineering Project

**Course:** LLM Application Engineering · SDA-AIE-213
**Instructor:** Meaad Al-Marri
**Student:** Renad
**Reference material:** [Course repository](https://github.com/almiyead-rgb/murshid-llm-application-engineering-sda-aie-213) · [SDAIA Academy](https://github.com/SDAIAAcademy)


## About the project

Murshid Mini is a small, fictional assistant for government-style services. It answers known service questions, looks up the status of an application through a read-only tool that checks ownership, rejects unsuitable inputs, tests its own behaviour, and caches public answers safely. I built it over four days in **one cumulative Colab notebook**, adding new cells below the earlier ones each day.

> **Important: this is a mock (`mode=mock`).** The "model" is `FakeClient`, a deterministic simulator that returns fixed answers. It is **not a live language model**. The project teaches the engineering around an LLM-shaped interface: input validation, a consistent response format, tools, guards, tests, caching and logging. All data is synthetic.

## Architecture

```
User question → input guard → ticket extraction → validation
             → public answer (FakeClient)  OR  read-only status tool (with ownership check)
             → output check → response + minimal metrics
```

The interface, the business logic and the model client are kept separate, so a real provider could replace `FakeClient` later without rewriting the rest of the app.

## Progress

| Day | What I built | Notebook cells | Result file | Status |
|---|---|---:|---|---|
| 1 · Foundations | Synthetic directory, `FakeClient`, `ask()`, response contract | 1–3 | `results/core/baseline_results.json` | ✅ Done |
| 2 · Structure & tools | Validated ticket, read-only `check_status`, ownership check | 4–6 | `results/core/day2_results.json` | ⬜ |
| 3 · Guard & test | Versioned prompt, guards, pipeline, 10 fixed test cases | 7–10 | `results/core/day3_results.json` | ⬜ |
| 4 · Measure & finish | Public FAQ cache, metrics, regression checks, export | 11–14 | `results/core/day4_results.json`, `final_report.json` | ⬜ |

## Repository structure

```
notebooks/murshid.ipynb        # The one cumulative project notebook (Days 1–4)
homework/                      # One separate notebook per homework
results/core/                  # JSON outputs from my own runs
results/day1/, results/day4/   # Homework reports
docs/project-notes.md          # Data, design decisions and observations
```

## How to run

1. Open `notebooks/murshid.ipynb` in Google Colab (File → Open notebook → GitHub, then paste this repository's URL).
2. Choose **Runtime → Change runtime type → Python 3, CPU** (no accelerator).
3. Run all the cells in order from the top. Each day depends on the cells before it.
4. Outputs are saved to `murshid_outputs/` in the Colab Files panel.

Nothing needs installing. The core project uses only the Python standard library, with no API keys, no GPU, no model downloads and no Drive mount.

## Day 1 results

| Input | Answer |
|---|---|
| `كيف أتابع طلبي؟` | `يمكنك متابعة حالة طلبك باستخدام رقم المرجع.` |
| `How can I check my application status?` | `You can check your application status using its reference number.` |
| `ساعدني` | `ما الخدمة التي تحتاج مساعدة بشأنها؟` |
| *(empty input)* | `Please enter a valid question.`, with no client call |

Every response has `trace_id`, `answer`, `model=training-mock` and `latency_ms`. Trace IDs and timings change from run to run.

## Data

All the data is synthetic and built into the code:

- A public service directory with one Arabic and one English answer.
- `REF-1001`: owned by `demo-user-1`, status `under_review`.
- `REF-1002`: owned by `demo-user-2`, status `approved` (used to test access refusal).
- `REF-9999`: deliberately missing (used for the not-found test).

The project contains no real IDs, records, personal data or secrets.

## Limitations

- `FakeClient` matches fixed questions exactly. Any other wording gets a clarification reply, so there is no real language understanding.
- Passing the tests with the mock **does not** show anything about the quality of a real LLM.
- `latency_ms` measures local Python execution, not model inference.
- The input guards are teaching examples, not full security or privacy protection.
- Identity and ownership checks use demo fixtures, not real authentication.

## Homework

| Homework | Notebook | Status |
|---|---|---|
| Day 1: real model (FLAN-T5 small, CPU) | `homework/day1_real_model.ipynb` | ⬜ |
| Day 2: tool validation and routing | `homework/day2_tools.ipynb` | ⬜ |
| Day 3: error handling | `homework/day3_errors.ipynb` | ⬜ |
| Day 4: evaluating real-model answers | `homework/day4_evaluation.ipynb` | ⬜ |

The real-model homework uses **actual local model inference** in a separate notebook, kept apart from the mock core project.
