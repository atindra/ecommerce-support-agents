# E-commerce Order and Refund Support Assistant

A Streamlit app that compares how five classical agents and one local LLM agent handle the same order, delivery, return, or refund case. Each agent recommends a next step and explains its decision. All records are simulated; no real refunds, order changes, messages, or carrier contacts occur.

The classical agents use Python decision logic. The LLM agent uses **LangChain** to connect to **Qwen**, a language model run locally by **Ollama**, and retrieve evidence through tools. New to these concepts? See the [Beginner's Guide](BEGINNER_GUIDE.md).

**BITS Pilani WILP | Agentic AI (S2-25_PCAMZG519) | Assignment 1, Set 6**

**Group 21**

| Member                    | Student ID    |
| ------------------------- | ------------- |
| G.Chandramouli            | `2025paml525` |
| Atindra R K Ranjana Singh | `2025paml584` |

Contents: [Results](#recorded-results) | [Setup](#setup) | [Run](#run-and-verify) | [Project structure](#architecture-and-reading-order) | [Mechanisms](#five-mechanisms) | [Cases/policies](#cases-and-ordered-policies) | [Learning](#learning-example) | [LLM checks](#llm-output-and-limits) | [Rubric](#rubric-checklist)

## How To Evaluate This Project

For a quick review, start with the saved results; no setup or model calls are needed.

1. **Compare the cases:** read the [six-agent results](#six-agent-results). A-C are the required scenarios; D is the normal tracking case. B shows the Simple Reflex warning, and C shows the LLM's unsupported recommendation.
2. **Check the five mechanisms:** [Five Mechanisms](#five-mechanisms) explains each approach and links to its source. The [project structure](#architecture-and-reading-order) shows how the files fit together.
3. **Check actual LLM tools:** inspect the [final A-D tool traces](evidence/RESULTS.md#current-results-2026-11-14) and the [evidence checks](#llm-output-and-limits).
4. **See learning change:** the [two-failure example](#learning-example) includes before/after scores, screenshots, and matching downloads.
5. **Review verification:** use the [final evidence index](evidence/RESULTS.md#current-results-2026-11-14), [limitations](#limitations), and [20-mark rubric checklist](#rubric-checklist). Older diagnostics are labeled separately.
6. **Try it yourself:** follow [Setup](#setup), then `python -m streamlit run app.py`. The [run guide](#run-and-verify) covers offline and real-model comparisons.

## Recorded Results

- Latest offline suite: **275 passed, 4 deselected in 2.45s**, using Python 3.13.5 on the same macOS machine ([log](evidence/verification-offline-test-results.txt)).
- Final real `qwen3:4b` A-D batch: **4 integration tests passed in 243.14s**, with both tools called in every case. A/B/D match the expected actions; **C is incorrect**, retaining `explain_refund_pending` with `warning`. This is **3/4 agreement with the prepared case targets**, not a general accuracy measure.
- Separate real Streamlit captures cover A-D and agree with the batch outcomes, including C's warning; [screenshots, exports, and timings](evidence/RESULTS.md#current-results-2026-11-14) document these distinct UI runs, not another live pytest batch.
- Desktop A-D offline screenshots, expanded mechanism details, and a 390px mobile offline comparison demonstrate the UI. Wide comparison tables scroll horizontally on mobile. Testing scope is documented in [RESULTS.md](evidence/RESULTS.md#screenshot-scope).

See [results and evidence](evidence/RESULTS.md) for artifacts, timings, validation details, and limitations.

## Setup

Run commands from the **project directory**, which contains `app.py`. Use Python **3.13**. Start with the classical agents and offline tests; install [Ollama](https://ollama.com/download) only if you want to run the local LLM.

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

Start Ollama, then download the model below. If the desktop app is not already running its server, use `ollama serve` in a separate terminal. The adapter connects to `http://127.0.0.1:11434`.

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

Run these separately: the first starts the UI, the second runs offline tests, and the third prints 20 classical A-D results. Streamlit prints its URL, normally `http://localhost:8501`. The classical script uses the starting learning counts, without reading or updating your saved feedback. Offline tests check model behavior with controlled substitute messages, alongside classical and Streamlit checks; they do not run Qwen.

1. Select A-D and inspect **Shared case and simulation policies**.
2. Uncheck **Run real local LLM** for offline comparisons; its row becomes `not_run`, with no LLM action or score. The checkbox defaults to enabled.
3. For live use, keep `qwen3:4b` and select **Run Comparison**. Calls can take minutes.
4. Inspect the action, status, explanation, citations, and mechanism traces. **Policy Eligible** means the action meets the simulation rules; **Fixture Match** means it matches the prepared case target. A warning keeps the original answer visible.
5. Under **Learning**, record success/failure, rerun to see the ranking, or reset history. **Download this comparison** exports JSONL; saved evidence is not replayed as a new response.

The UI shows belief updates, search plans, and candidate scores. It also compares default utility scores with a separate risk-heavy weight example. After feedback, it immediately shows the estimate change; run a new comparison to see the updated decision.

With Ollama running, test live integration separately:

```bash
python -m pytest tests/test_llm.py -m live -v -s
python -m pytest "tests/test_llm.py::test_live_qwen3_four_cases[C]" -m live -v -s
```

Allow more than 600 seconds for a batch if the model server is busy, or replace `C` with A/B/D to run one case. Live tests overwrite `evidence/llm_qwen3-4b_<case>.json`; use a separate project copy to preserve the saved evidence. They require both tools and correctly structured citations but accept `ok` or `warning`, so C's passing test does not mean its recommendation is correct.

## Architecture And Reading Order

The main flow is **prepared case -> six agent decisions -> evidence checks -> UI comparison**.

```text
app.py                 Streamlit UI and feedback controls
agents/                Five classical agents and the LLM adapter
contracts.py           Input/output types and case loader
fixtures.json          Prepared cases A-D
domain.py              Shared facts, policies, and result checks
run_classical.py       Terminal comparison of classical agents
tests/                 Automated agent, policy, LLM, and UI checks
evidence/              Saved results, screenshots, and RESULTS.md index
requirements.txt       Pinned Python dependencies
runtime/               Local learning counts, created on feedback/reset
```

Suggested source reading order: [contracts.py](contracts.py) -> [fixtures.json](fixtures.json) -> [domain.py](domain.py) -> [agents/](agents/) -> [app.py](app.py) / [run_classical.py](run_classical.py). Use [tests/](tests/) for automated checks and [RESULTS.md](evidence/RESULTS.md) for saved evidence.

All agents use the same read-only case. The LLM retrieves that case through tools restricted to the selection; Learning also receives a copy of the feedback counts. Shared eligibility limits the available actions but does not choose an answer for every agent.

Results stay in the current UI session. Changing the case, model, LLM toggle, or learning counts clears the old comparison, as does feedback/reset. Only learning counts persist across restarts. Supporting scripts handle optional screenshots and earlier Ollama diagnostics; the [Beginner's Guide](BEGINNER_GUIDE.md#5-folders-and-files) describes them and the remaining files.

## Five Mechanisms

1. **Simple Reflex** ([source](agents/simple_reflex.py)): chooses the first matching rule based on current fields. It does not use history, search, or age/deadline calculations. In B, it chooses pending explanation, which validation keeps and flags as ineligible.
2. **Model-Based Reflex** ([source](agents/model_based.py)): reconstructs beliefs from dated observations, records changes/conflicts, and checks them against the current records before applying rules. It rebuilds this state for each run.
3. **Goal-Based** ([source](agents/goal_based.py)): uses breadth-first search (BFS) to find a shortest workflow toward the customer's goal. Steps have explicit prerequisites and effects. If the goal is unreachable, it searches for clarification/review instead. Plan steps are hypothetical recommendations, not completed operations.
4. **Utility-Based** ([source](agents/utility_based.py)): scores policy-eligible actions and chooses the maximum: `0.40*impact - 0.25*delay - 0.10*effort - 0.25*error_risk`. Features and weights are teaching assumptions. Traces show each contribution, excluded action, and ranking; different weights can change the choice.
5. **Learning** ([source](agents/learning.py)): groups cases into fixed contexts and ranks eligible actions using saved success/failure counts: `(success+1)/(success+failure+2)`. It compares unrounded estimates, uses a fixed tie-break order, and chooses review for unknown contexts. Policy constraints still apply. It learns action preferences, not language.

## Cases And Ordered Policies

All prepared cases (fixtures) use **2026-11-13**, not the machine date. Expected actions are used only for evaluation/UI comparison, not given to agents. Policy thresholds are defined for this simulation (`sim-v1`); the assignment does not prescribe them.

| Case               | Evidence                                                                                                | Fixture target           |
| ------------------ | ------------------------------------------------------------------------------------------------------- | ------------------------ |
| A, critical        | E-reader delivered Nov 10; receipt denied; no photo/signature                                           | `delivery_investigation` |
| B, service failure | Raincoat returned Oct 29; inspection passed Oct 30; refund pending after Nov 6 deadline                 | `payments_escalation`    |
| C, ambiguous       | Kettle delivered Oct 20 with photo; Oct 30 return claimed but uncorroborated; inspection/refund unknown | `request_clarification`  |
| D, normal          | Keyboard shipped Nov 10; regional hub Nov 12; in transit, expected Nov 16                               | `share_tracking`         |

### Six-Agent Results

Classical results use the documented seed counts; LLM results are the saved real `qwen3:4b` A-D batch, not a replay in the app. Labels abbreviate the action identifiers above.

| Agent               | A             | B                                | C                                | D        |
| ------------------- | ------------- | -------------------------------- | -------------------------------- | -------- |
| Simple Reflex       | Investigation | **Explain pending (ineligible)** | Clarification                    | Tracking |
| Model-Based Reflex  | Investigation | Payments escalation              | Clarification                    | Tracking |
| Goal-Based          | Investigation | Payments escalation              | Clarification                    | Tracking |
| Utility-Based       | Investigation | Payments escalation              | Clarification                    | Tracking |
| Learning, seeded    | Investigation | Payments escalation              | Clarification                    | Tracking |
| LLM, real saved run | Investigation | Payments escalation              | **Explain pending (ineligible)** | Tracking |

B shows why refund-deadline calculations matter: the Simple Reflex agent only reads the pending status. C shows an LLM error: citing an **unknown** refund status does not establish a pending refund. Both original answers remain visible.

The other agents often agree, but use different mechanisms: belief updates, BFS, weighted scores, and outcome estimates. In A, risk-heavy utility weights choose review; two learning failures also switch investigation to review, without changing eligibility. See [current evidence and test history](evidence/RESULTS.md).

### Policy Precedence

| Order | Policy         | Boundary and permitted recommendation                                                                                                                                                                  |
| ----- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | `SAFE`         | Recommend only; no operational actions. Human review remains available.                                                                                                                                |
| 2     | `VERIFY`       | Missing prerequisites or conflicting/future evidence require clarification/review first. Missing return/refund records alone do not block a delivery complaint.                                        |
| 3     | `DNR`          | Non-receipt with consistent delivered evidence: investigate at age 0-7 calendar days inclusive; otherwise review. Proof is optional; delivery is not proof of receipt.                                 |
| 4     | `REFUND`       | Require return, passed inspection/date, and known refund status. Explain completed refunds; explain pending through the deadline; escalate still-pending only after it.                                |
| 5     | `TRACK`        | Shipped/in-transit and expected date not passed: explain status/share usable tracking. Missing expected date: clarify; after expected date: review. Missing tracking: clarify/status, never invent it. |
| 6     | Unknown intent | Clarification/review, not an invented operational remedy.                                                                                                                                              |

Delivery non-receipt (`DNR`), refund, and tracking rules apply according to the customer's intent. The Simple Reflex agent omits some date checks; validation flags its answer rather than replacing it.

Refund deadlines add five Monday-Friday business days after inspection, excluding inspection day and ignoring holidays. `today > deadline` means overdue: a Friday deadline is still within the service-level agreement (SLA) on Friday and overdue on Saturday/Monday. Missing or future dates are not treated as zero elapsed days. After editing `fixtures.json`, restart and compare again.

## Learning Example

All eight actions in each of nine fixed contexts start at **0 successes / 0 failures** (estimate `1/2`), except A's `timely_nonreceipt`: investigation **6/3**, review **5/4**. These are synthetic seed outcomes; B/C/D use neutral defaults and deterministic ties, not historical successes.

Select A, reset learning, and rerun after each explicit failure click:

| State          | Investigation counts | Investigation estimate | Review estimate     | Selection     |
| -------------- | -------------------- | ---------------------- | ------------------- | ------------- |
| Seed           | 6/3                  | `7/11`                 | `6/11` (counts 5/4) | Investigation |
| First failure  | 6/4                  | `7/12`                 | `6/11`              | Investigation |
| Second failure | 6/5                  | `7/13`                 | `6/11`              | Human review  |

Only feedback/reset writes `runtime/learning_counts.json`; counts survive restarts. If the file is absent, the agent uses starting counts without creating it. Each feedback click updates the displayed context/action once and clears the comparison. Rerun before recording more feedback; ordinary comparisons do not add outcomes. Persistence, reset, and the decision switch are tested with temporary files. The history file supports one user at a time.

| Textbook component  | Actual implementation                                                                                                                                          |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Performance element | `agents/learning.py:run()` classifies via `context()`, filters with `domain.eligibility()`, and chooses the highest smoothed estimate with deterministic ties. |
| Critic              | Explicit user success/failure in `app.py:main()` supplies outcome feedback; there is no automatic customer-outcome assessor.                                   |
| Learning element    | `learning.feedback()` updates persistent counts; `load_counts()` reloads them and `run()` recomputes estimates.                                                |
| Problem generator   | Not implemented: no exploration, experimental actions, or autonomous feedback generation.                                                                      |

Browser evidence: [initial scores](evidence/learning_A_stage_0.png), [first failure](evidence/learning_A_stage_1.png), [second failure / review](evidence/learning_A_stage_2.png). Matching downloads: [seed](evidence/learning_A_stage_0.jsonl), [first failure](evidence/learning_A_stage_1.jsonl), [second failure](evidence/learning_A_stage_2.jsonl). The LLM was disabled throughout; recommendations and scores are unaltered.

## LLM Output And Limits

The adapter executes the model's requests for `lookup_case_records` and `lookup_support_policy`, then sends their results back as tool messages. Both tools are restricted to the selected case. It allows up to six model invocation rounds and one answer-format/citation repair. It does not substitute another model or a classical answer; connection failures appear as `error`, while classical results remain available.

The PDF suggests tool functions, not mandatory names. This project groups them into two distinct case-bound LangChain tools:

| Project tool                     | Suggested functions covered                                          | Returned evidence                                                                                                  |
| -------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `lookup_case_records(case_id)`   | `get_order_status()`, `get_delivery_events()`, `get_refund_status()` | Selected order, dated observations, delivery/return/inspection/refund fields, customer claims, and evaluation date |
| `lookup_support_policy(case_id)` | `search_return_policy()`                                             | Applicable synthetic safety, verification, delivery, refund, and tracking clauses; not external web search         |

| Check            | What passes                                                                                                | Limitation / failure behavior                                                                                                                         |
| ---------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Execution        | Both tools succeed for the selected case and their outputs reach the model                                 | Tool execution alone does not establish correctness. Missing evidence at finalization or tool-loop exhaustion yields `incomplete_evidence`.           |
| Schema/citations | Valid action, public rationale, policy ID, and 2-8 existing `{tool, field}` references covering both tools | A real field may still be misinterpreted. One format/citation repair is allowed; unresolved issues yield `invalid_output`.                            |
| Policy support   | Eligible action, matching cited policy, and at least one action-relevant record citation                   | These checks do not prove every claim or require every prerequisite to be cited. Unsupported answers remain `warning`, without a policy-based repair. |

C cites an existing `refund_status` whose value is `unknown`; its wrong action remains a warning, not a forced correction. The stable saved-C offline regression checks this distinction through the real adapter with substitute model messages.

## Rubric Checklist

| Rubric                       | Implementation / demonstration                                                                                                                                                                                                                          |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Five classical agents (5)    | [Five mechanisms and source](#five-mechanisms): separate rule matching, belief replay, BFS, weighted utility, and outcome-count selectors; [behavioral tests](tests/test_agents.py) and expanded UI traces                                              |
| LLM/tools (4)                | [Adapter](agents/llm.py) and [checks](#llm-output-and-limits): local LangChain/Ollama, two actual case-bound tools, typed citations, independent support checks; [saved A-D traces and live UI exports](evidence/RESULTS.md#current-results-2026-11-14) |
| Shared scenarios/testing (3) | [Frozen A-D fixtures](fixtures.json), common snapshots, failure/ambiguity and [policy boundary tests](tests/test_domain.py)                                                                                                                             |
| Streamlit/comparison (3)     | [app.py](app.py): six rows, reasons, mechanism explanations, inspectable tools, download; [screenshots](evidence/RESULTS.md#current-results-2026-11-14)                                                                                                 |
| Learning/evaluation (3)      | [Persistent feedback demonstration](#learning-example), tested and browser-captured decision switch, hard eligibility for utility/learning, [honest C failure](#limitations)                                                                            |
| Documentation (2)            | [Setup](#setup), [pinned requirements](requirements.txt), [results/screenshots](evidence/RESULTS.md#current-results-2026-11-14), [comparison](#six-agent-results), and [limitations](#limitations)                                                      |

## Limitations

The main correctness limitation is case C: the real model recommended explaining a pending refund when the evidence called for clarification. The raw action is kept and flagged as a warning rather than silently corrected.

- **Scope:** Classical agents use prepared, structured claims. Policies, utility features, and learning seeds are synthetic; learning adjusts preferences within fixed contexts rather than handling arbitrary language or new situations.
- **Checks:** Evidence validation confirms that the tools ran and that the citations are well-formed and relevant. It does not prove that every claim in the rationale is fully supported.
- **Persistence:** Learning history is stored locally and supports one user at a time.
- **LLM results:** Public output shows the rationale and tool/validation traces, not private model reasoning. The four-case batch demonstrates these scenarios, not broad model performance. Results may vary with runtime versions, model contents, hardware, and server load.

Detailed evidence and remaining gaps are in [RESULTS.md](evidence/RESULTS.md).
