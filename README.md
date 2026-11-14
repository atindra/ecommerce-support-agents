# Evidence Before Action: Support Lab

A single-process e-commerce support simulator comparing five classical agents and an optional local LangChain/Ollama agent on the same frozen case. Records are synthetic and read-only: agents recommend next steps, never issue refunds, change orders, send messages, or contact carriers.

**BITS Pilani WILP | Agentic AI (S2-25_PCAMZG519) | Assignment 1, Set 6**  
Project: **E-commerce Order and Refund Support Assistant**

**Group 21**

| Member | Student ID |
|---|---|
| G.Chandramouli | `2025paml525` |
| Atindra R K Ranjana Singh | `2025paml584` |

Contents: [Results](#recorded-results) | [Setup](#setup) | [Run](#run-and-verify) | [Architecture](#architecture-and-reading-order) | [Mechanisms](#five-mechanisms) | [Cases/policies](#cases-and-ordered-policies) | [Learning](#learning-example) | [LLM checks](#llm-output-and-limits) | [Rubric](#rubric-checklist)

## How To Evaluate This Project

1. **Run the UI:** follow [Setup](#setup), then `python -m streamlit run app.py`. The [run guide](#run-and-verify) covers offline and real-model comparisons.
2. **Compare A-D:** use the scenario selector and expand agent details; the [six-agent results](#six-agent-results) explain agreement and differences, including the reflex's B warning and the LLM's C failure.
3. **Inspect the classical agents:** follow [Five Mechanisms](#five-mechanisms) for links to each implementation, or the [source reading order](#architecture-and-reading-order) for shared contracts and validation.
4. **Check real tool use:** inspect [the LLM adapter](agents/llm.py), its [evidence checks](#llm-output-and-limits), and the [final A-D tool traces](evidence/RESULTS.md#current-results-2026-11-14).
5. **Demonstrate learning:** follow the [two-failure example](#learning-example), then restart the app to confirm feedback persists. Screenshots and matching downloads are linked there.
6. **Review verification:** use the [current evidence index](evidence/RESULTS.md#current-results-2026-11-14), [known limitations](#limitations), and [20-mark rubric checklist](#rubric-checklist). Historical diagnostics are labeled separately in RESULTS.

## Recorded Results

- Latest offline suite: **275 passed, 4 deselected in 2.45s**, using Python 3.13.5 on the same macOS machine ([log](evidence/verification-offline-test-results.txt)).
- Final real `qwen3:4b` A-D batch: **4 integration tests passed in 243.14s**, with both tools called in every case. A/B/D match targets; **C is wrong**, retaining `explain_refund_pending` with `warning`. This is **3/4 fixture agreement, not accuracy**.
- Separate real Streamlit captures cover A-D and agree with the batch outcomes, including C's warning; [screenshots, exports, and timings](evidence/RESULTS.md#current-results-2026-11-14) document these distinct UI runs, not another live pytest batch.
- Desktop A-D offline screenshots, expanded mechanism details, and a 390px mobile offline comparison demonstrate the UI. Wide comparison tables scroll horizontally on mobile; this is not exhaustive mobile or Windows testing.

See [results and evidence](evidence/RESULTS.md) for artifacts, timings, validation details, and limitations.

## Setup

Run commands from the **project directory**. Use Python **3.13**; install [Ollama](https://ollama.com/download) only for local inference. Offline tests and classical comparisons need no Ollama server.

### macOS

```bash
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Windows PowerShell

```powershell
py -3.13 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If activation is blocked, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`, then activate again. Windows instructions have not been verified on Windows.

### Local Model

Install/start Ollama. If its desktop app is not already serving, run `ollama serve` in a separate terminal; do not start a second server on port 11434. Then:

```bash
ollama pull qwen3:4b
ollama list
ollama --version
```

Tested: macOS, Apple M1 Max, 32 GB RAM, Python **3.13.5**, Ollama **0.40.16**, original **`qwen3:4b`** tag (Qwen3 4B Thinking 2507). Dependencies are pinned in `requirements.txt`; alternate model tags need separate validation. See the [runtime diagnosis](evidence/ollama-diagnosis.md) and [full adapter settings](evidence/RESULTS.md#runtime-settings).

## Run And Verify

```bash
python -m streamlit run app.py
python -m pytest -m "not live" -q
python run_classical.py
```

Run these separately. Streamlit prints its URL, normally `http://localhost:8501`. The classical script prints 20 A-D results using seed learning counts, without reading adapted history or writing feedback. Offline tests include substitute-model and Streamlit checks, not real inference.

1. Select A-D and inspect **Shared case and simulation policies**.
2. Uncheck **Run real local LLM** for offline comparisons; its row becomes `not_run`, with no LLM action or score. The checkbox defaults to enabled.
3. For live use, keep `qwen3:4b` and select **Run Comparison**. Calls can take minutes.
4. Inspect the raw action, status, **Policy Eligible**, **Fixture Match**, rationale, citations, and mechanism traces. A warning is not a corrected answer.
5. Under **Learning**, record success/failure, rerun to see the ranking, or reset history. **Download this comparison** exports JSONL; saved evidence is not replayed as a new response.

The UI explains why mechanisms differ, shows belief transitions/search plans/candidate tables, and compares default utility scores with disclosed risk-heavy weights. Feedback confirmation appears immediately with before/after estimates, before a new comparison is required.

With Ollama running, test live integration separately:

```bash
python -m pytest tests/test_llm.py -m live -v -s
python -m pytest "tests/test_llm.py::test_live_qwen3_four_cases[C]" -m live -v -s
```

Allow more than 600 seconds for a batch under contention, or replace `C` with A/B/D to run individual cases. Live tests overwrite `evidence/llm_qwen3-4b_<case>.json`; they require both tools and typed citations but accept `ok` or `warning`, so C's passing test is not a correct recommendation.

## Architecture And Reading Order

| Read | File | Purpose |
|---|---|---|
| 1 | [contracts.py](contracts.py) | Frozen case, action/result types, typed final answer, citations, and fixture loader |
| 2 | [fixtures.json](fixtures.json) | Synthetic A-D records |
| 3 | [domain.py](domain.py) | Derived facts, ordered eligibility constraints, raw-result validation |
| 4 | [agents/](agents/) | Five independent selectors and the local LLM adapter |
| 5 | [run_classical.py](run_classical.py), [app.py](app.py) | Seeded comparison and interactive UI/feedback |
| 6 | [tests/](tests/), [RESULTS.md](evidence/RESULTS.md) | Contract tests, saved results, and measurement limits |

All agents receive the same immutable case snapshot; LLM tools close over that selected case rather than reload a global store. Learning additionally receives a copied history snapshot. Shared eligibility supplies constraints, not a central answer selector. Results remain in session state, not a persistent result cache; case/model/LLM-toggle/count changes invalidate them, and feedback/reset clears them.

## Five Mechanisms

1. **Simple Reflex** ([source](agents/simple_reflex.py)): first matching current-field rule; no history, search, or age/SLA calculation. It selects pending on B, which validation retains as an ineligible warning.
2. **Model-Based Reflex** ([source](agents/model_based.py)): rebuilds beliefs from dated observations, records transitions/conflicts, then reconciles the current snapshot before applying rules. State is local to each invocation.
3. **Goal-Based** ([source](agents/goal_based.py)): breadth-first search over facts, recommendations, and paths using explicit preconditions/effects. Intent sets the goal; unreachable goals trigger clarification/review search. Plan effects are hypothetical workflow steps, not observed commerce facts.
4. **Utility-Based** ([source](agents/utility_based.py)): filters by eligibility, then maximizes `0.40*impact - 0.25*delay - 0.10*effort - 0.25*error_risk`. Features/weights are synthetic teaching assumptions; traces show contributions, exclusions, and ranking. Weight overrides can change eligible tradeoffs.
5. **Learning** ([source](agents/learning.py)): fixed context classifier and persistent outcome counts rank eligible actions by `(success+1)/(success+failure+2)` at full precision. Hard constraints override rates; deterministic ties and unknown-context review keep behavior bounded. This is not general language learning.

## Cases And Ordered Policies

All fixtures evaluate at **2026-11-13**, not the machine date. Targets are evaluation/UI constants, not agent inputs. These are simulation policies (`sim-v1`), not assignment-mandated commerce thresholds.

| Case | Evidence | Fixture target |
|---|---|---|
| A, critical | E-reader delivered Nov 10; receipt denied; no photo/signature | `delivery_investigation` |
| B, service failure | Raincoat returned Oct 29; inspection passed Oct 30; refund pending after Nov 6 deadline | `payments_escalation` |
| C, ambiguous | Kettle delivered Oct 20 with photo; Oct 30 return claimed but uncorroborated; inspection/refund unknown | `request_clarification` |
| D, normal | Keyboard shipped Nov 10; regional hub Nov 12; in transit, expected Nov 16 | `share_tracking` |

### Six-Agent Results

Classical results use the documented seed counts; LLM results are the saved real `qwen3:4b` A-D batch, not a replay in the app. Labels abbreviate the action identifiers above.

| Agent | A | B | C | D |
|---|---|---|---|---|
| Simple Reflex | Investigation | **Explain pending (ineligible)** | Clarification | Tracking |
| Model-Based Reflex | Investigation | Payments escalation | Clarification | Tracking |
| Goal-Based | Investigation | Payments escalation | Clarification | Tracking |
| Utility-Based | Investigation | Payments escalation | Clarification | Tracking |
| Learning, seeded | Investigation | Payments escalation | Clarification | Tracking |
| LLM, real saved run | Investigation | Payments escalation | **Explain pending (ineligible)** | Tracking |

B exposes the reflex's lack of SLA arithmetic. C exposes a model error: a citation to an **unknown** refund status does not establish a pending refund. Both raw errors remain visible. Agreement among other agents does not mean they share a selector: belief replay, BFS, weighted scores, and outcome estimates work differently. On A, risk-heavy utility weights select review; two explicit learning failures also switch investigation to review, while eligibility is unchanged. See [current evidence and test history](evidence/RESULTS.md).

### Policy Precedence

| Order | Policy | Boundary and permitted recommendation |
|---|---|---|
| 1 | `SAFE` | Recommend only; no operational actions. Human review remains available. |
| 2 | `VERIFY` | Missing prerequisites or conflicting/future evidence require clarification/review first. Missing return/refund records alone do not block a delivery complaint. |
| 3 | `DNR` | Non-receipt with consistent delivered evidence: investigate at age 0-7 calendar days inclusive; otherwise review. Proof is optional; delivery is not proof of receipt. |
| 4 | `REFUND` | Require return, passed inspection/date, and known refund status. Explain completed refunds; explain pending through the deadline; escalate still-pending only after it. |
| 5 | `TRACK` | Shipped/in-transit and expected date not passed: explain status/share usable tracking. Missing expected date: clarify; after expected date: review. Missing tracking: clarify/status, never invent it. |
| 6 | Unknown intent | Clarification/review, not an invented operational remedy. |

DNR/refund/tracking are ordered intent branches, not clauses applied to every case. The simple reflex intentionally omits some temporal checks; validation flags rather than replaces its action. Refund deadlines add five Monday-Friday business days after inspection, excluding inspection day and ignoring holidays. `today > deadline` means overdue: a Friday deadline is within SLA Friday and overdue Saturday/Monday. Missing/future dates are not zero elapsed days. After editing `fixtures.json`, restart and compare again.

## Learning Example

All eight actions in each of nine fixed contexts start at **0 successes / 0 failures** (estimate `1/2`), except A's `timely_nonreceipt`: investigation **6/3**, review **5/4**. These are synthetic seed outcomes; B/C/D use neutral defaults and deterministic ties, not historical successes.

Select A, reset learning, and rerun after each explicit failure click:

| State | Investigation counts | Investigation estimate | Review estimate | Selection |
|---|---|---|---|---|
| Seed | 6/3 | `7/11` | `6/11` (counts 5/4) | Investigation |
| First failure | 6/4 | `7/12` | `6/11` | Investigation |
| Second failure | 6/5 | `7/13` | `6/11` | Human review |

Only explicit feedback/reset writes `runtime/learning_counts.json`; counts survive restarts. An absent file uses seeds without writing. Each feedback click updates the displayed context/action once and clears the comparison, requiring a new run before more feedback. Ordinary reruns do not duplicate outcomes. Persistence/reset and this switch are tested with temporary files; the runtime file is a single-user store, not concurrency-safe production storage.

| Textbook component | Actual implementation |
|---|---|
| Performance element | `agents/learning.py:run()` classifies via `context()`, filters with `domain.eligibility()`, and chooses the highest smoothed estimate with deterministic ties. |
| Critic | Explicit user success/failure in `app.py:main()` supplies outcome feedback; there is no automatic customer-outcome assessor. |
| Learning element | `learning.feedback()` updates persistent counts; `load_counts()` reloads them and `run()` recomputes estimates. |
| Problem generator | Not implemented: no exploration, experimental actions, or autonomous feedback generation. |

Browser evidence: [initial scores](evidence/learning_A_stage_0.png), [first failure](evidence/learning_A_stage_1.png), [second failure / review](evidence/learning_A_stage_2.png). Matching downloads: [seed](evidence/learning_A_stage_0.jsonl), [first failure](evidence/learning_A_stage_1.jsonl), [second failure](evidence/learning_A_stage_2.jsonl). The LLM was disabled throughout; recommendations and scores are unaltered.

## LLM Output And Limits

The adapter preserves assistant tool calls and returns actual tool messages for `lookup_case_records` and `lookup_support_policy`, restricted to the selected case. It allows six invocation rounds, including at most one final schema/citation repair. There is no automatic model or classical-answer fallback; connection failures remain `error` without hiding classical results.

The PDF suggests tool functions, not mandatory names. This project groups them into two distinct case-bound LangChain tools:

| Project tool | Suggested functions covered | Returned evidence |
|---|---|---|
| `lookup_case_records(case_id)` | `get_order_status()`, `get_delivery_events()`, `get_refund_status()` | Selected order, dated observations, delivery/return/inspection/refund fields, customer claims, and evaluation date |
| `lookup_support_policy(case_id)` | `search_return_policy()` | Applicable synthetic safety, verification, delivery, refund, and tracking clauses; not external web search |

| Check | What passes | Limitation / failure behavior |
|---|---|---|
| Execution | Both actual tools succeed for the selected case and their outputs reach the model | Transport/retrieval is not correctness; missing coverage yields `incomplete_evidence`. |
| Schema/citations | Typed action, public rationale, policy ID, and 2-8 existing `{tool, field}` references spanning both tools | Existing fields need not support the claim; one bounded schema/citation repair is allowed, then `invalid_output`. |
| Policy support | Eligible action, declared policy cited and compatible, at least one action-relevant record citation | Not full semantic entailment or every prerequisite explicitly cited; failure retains the raw action as `warning`, with no support-driven repair. |

C cites an existing `refund_status` whose value is `unknown`; its wrong action remains a warning, not a forced correction. The stable saved-C offline regression checks this distinction through the real adapter with substitute model messages.

## Rubric Checklist

| Rubric | Implementation / demonstration |
|---|---|
| Five classical agents (5) | [Five mechanisms and source](#five-mechanisms): separate rule matching, belief replay, BFS, weighted utility, and outcome-count selectors; [behavioral tests](tests/test_agents.py) and expanded UI traces |
| LLM/tools (4) | [Adapter](agents/llm.py) and [checks](#llm-output-and-limits): local LangChain/Ollama, two actual case-bound tools, typed citations, independent support checks; [saved A-D traces and live UI exports](evidence/RESULTS.md#current-results-2026-11-14) |
| Shared scenarios/testing (3) | [Frozen A-D fixtures](fixtures.json), common snapshots, failure/ambiguity and [policy boundary tests](tests/test_domain.py) |
| Streamlit/comparison (3) | [app.py](app.py): six rows, reasons, mechanism explanations, inspectable tools, download; [screenshots](evidence/RESULTS.md#current-results-2026-11-14) |
| Learning/evaluation (3) | [Persistent feedback demonstration](#learning-example), tested and browser-captured decision switch, hard eligibility for utility/learning, [honest C failure](#limitations) |
| Documentation (2) | [Setup](#setup), [pinned requirements](requirements.txt), [results/screenshots](evidence/RESULTS.md#current-results-2026-11-14), [comparison](#six-agent-results), and [limitations](#limitations) |

## Limitations

The main correctness limitation is case C: the real model recommended explaining a pending refund when the evidence called for clarification. The raw action is kept and flagged as a warning rather than silently corrected.

- **Scope:** The classical agents work from authored, structured claims rather than general language understanding. Policies, utility features and learning seeds are synthetic, and the learning counts cover fixed contexts, so they do not generalize.
- **Checks:** Evidence validation confirms that the tools ran and that the citations are well-formed and relevant. It does not prove that every claim in the rationale is fully supported.
- **Persistence and platforms:** Learning history is a single-user file, not concurrency-safe storage. Testing was done on macOS only; the Windows steps are untested.
- **LLM results:** Public output shows the rationale and the tool and validation traces, not the model's private reasoning. Four captured completions cannot establish reliability or safety, and results can change with runtime versions, model contents, hardware and server load.

Detailed evidence and remaining gaps are in [RESULTS.md](evidence/RESULTS.md).
