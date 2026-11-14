# Results And Evidence

Historical installation/live logs use `<project-directory>` in place of original local filesystem paths. Recorded results and timings are unchanged.

## Current Results (2026-11-14)

This section supersedes the historical measurements below wherever they differ. Earlier logs are preserved; the A-D model JSON and current screenshot files were refreshed from final code.

| Check | Actual result | Evidence / boundary |
|---|---|---|
| Latest offline suite | 275 passed, 4 live tests deselected in 2.45s | [Offline verification log](verification-offline-test-results.txt); includes the stable saved-C replay |
| Real Ollama/LangChain A-D | 4 integration tests passed; A/B/D match, C fails support | [Completed batch](final-live-test-results.txt), `llm_qwen3-4b_{A,B,C,D}.json`; passing integration is not four correct decisions |
| Browser UI | Separate real A-D inference; A-D offline desktop and A mobile captures | `final-ui-{A,B,C,D}-comparison.jsonl`, `scenario_{A,B,C,D}_live.png`; not exhaustive platform testing |
| Learning | Browser-captured initial / first failure / second failure switch; reload/reset also tested | `learning_A_stage_{0,1,2}.{png,jsonl}`; LLM disabled and original absent runtime state restored |

- [Earlier final offline log](final-offline-test-results.txt): **274 passed, 4 deselected in 3.53s**. Includes immediate feedback display/reset, mechanism explanation/weight-sensitivity UI, action-policy compatibility, uncited declared policy, and irrelevant-record regression checks. An initial run exposed two fake answers citing unrelated `order_status` for refund/tracking; those test inputs now cite relevant facts rather than weakening the checks.
- [Final live batch log](final-live-test-results.txt): **4 passed, 64 deselected in 243.14s**, one completed sequential A-D run using original `qwen3:4b` and the existing shared service without restarting/reconfiguring it.

| Case | Raw action | Status | Latency seconds | Support/fixture |
|---|---|---|---:|---|
| A | delivery_investigation | ok | 45.712 | Supported / match |
| B | payments_escalation | ok | 129.840 | Supported / match |
| C | explain_refund_pending | warning | 37.757 | Ineligible / mismatch |
| D | share_tracking | ok | 29.633 | Supported / match |

All four final traces (`llm_qwen3-4b_A.json` through D) contain both successful model-initiated tools, returned evidence, typed citations, two rounds and zero repairs. Integration passing does not imply correctness: **C still fails policy support**, with the unmodified model action preserved. This is 3/4 fixture agreement, not accuracy.

Browser verification used separately owned Streamlit instances on port 18517 and existing Chrome via optional Playwright. [Live A screenshot](scenario_A_live.png) and [downloaded six-agent result](final-ui-A-comparison.jsonl) retain the genuine A run. B/C/D each have one separate live UI run, distinct from the live pytest batch; the shared Ollama service was not modified:

| New UI case | Raw action / status | Latency seconds | Evidence |
|---|---|---:|---|
| B | `payments_escalation` / `ok` | 58.949 | [Screenshot](scenario_B_live.png), [six-agent export](final-ui-B-comparison.jsonl) |
| C | `explain_refund_pending` / **`warning`** | 33.150 | [Screenshot](scenario_C_live.png), [six-agent export](final-ui-C-comparison.jsonl) |
| D | `share_tracking` / `ok` | 25.926 | [Screenshot](scenario_D_live.png), [six-agent export](final-ui-D-comparison.jsonl) |

All three used both actual tools, complete retrieval, typed citations, two rounds and zero repairs. Actions/statuses agree with the earlier pytest batch; timings differ. C again fails eligibility/support despite successful execution and valid references. No output was selected for being more favorable, and no retry was made. Screenshots focus on the expanded LLM outcome; downloads preserve all six rows and full traces.

The offline learning sequence used actual feedback buttons, rerunning comparison between clicks without LLM calls: [seed](learning_A_stage_0.png), [one failure](learning_A_stage_1.png), [two failures / review](learning_A_stage_2.png). Corresponding `learning_A_stage_{0,1,2}.jsonl` downloads preserve full-precision counts and estimates. Investigation falls from 7/11 to 7/12 to 7/13, below review's 6/11 only after the second failure. The original absent runtime state was restored. Screenshots retain their original resolution and results; blank bottom space is cropped and the personal-path caption uses a neutral grey mask.

Desktop A-D offline images remain current; [expanded utility/learning details](mechanisms_A_offline.png) show mechanism information. [Mobile A comparison](scenario_A_mobile_offline.png) uses 390x844 with the sidebar closed. Wide tables require horizontal scrolling; this is limited rendering verification, not exhaustive mobile usability testing.

For optional browser capture, install Playwright in a separate tools environment if not already available (`python3.13 -m venv ../capture-tools`, then `../capture-tools/bin/python -m pip install playwright`); installed Google Chrome is also required. Run Streamlit in its application environment, then `../capture-tools/bin/python capture_screenshots.py --url http://127.0.0.1:18517 --skip-offline --live --live-cases B C D --learning`. Use a new evidence copy to avoid overwriting existing captures. Omit `--live` for no model calls. The script never replays saved evidence; `--learning` temporarily resets counts and restores original state. Do not use it concurrently with another feedback writer.

Support now distinguishes action eligibility from declared-policy/citation relevance. Its checks are intentionally bounded: relevant fields and compatible policy do not prove every rationale claim or every prerequisite is explicitly cited. No policy changes, scenario-specific prompt tuning, or automatic action correction were introduced. Windows and fresh-machine validation remain unverified.

### Clean Package Verification

The submission ZIP was extracted into a new directory, a fresh Python 3.13.5 venv was created without system site packages, and dependencies were installed from the extracted `requirements.txt`. The complete offline suite passed **275 passed, 4 deselected in 15.54s** ([log](package-offline-test-results.txt)). `python run_classical.py` also succeeded. Archive CRC/integrity, local Markdown targets and anchors, live/offline A-D images and exports, all three learning stages, and absence of personal absolute paths and runtime feedback were checked. This is same-machine fresh-environment verification, not fresh-machine/Windows testing or new live inference.

Packaging excludes environments, caches, bytecode, runtime feedback, and backups. Useful failure records, linked diagnostics (including `ollama-diagnosis.md` and its JSON probes), and historical test logs are retained. The diagnosis's external runtime-log line references are historical observations; their full server log is not bundled. Personal project paths in the final live log were replaced with `<project-directory>` without changing measurements.

## Historical Verification (Superseded)

The following details preserve earlier development/test history from **2026-11-14**, not current-run claims. All A-D cases evaluate simulated records as of **2026-11-13**. Setup, architecture, mechanisms, policy precedence, and commands are in [README.md](../README.md).

## Summary

| Evidence class | Observed result | Boundary |
|---|---|---|
| Offline suite | `270 passed, 4 deselected in 2.35s` | Deterministic tests and substitutes do not establish real-model reliability. |
| Fresh venv | Pinned requirements installed in Python 3.13.5 `.venv-verify`; `270 passed, 4 deselected in 13.50s` | Same machine, not a fresh-machine or Windows verification. |
| Classical comparison | 20 results, five agents across A-D, seeded learning history | Simple Reflex misses B's overdue refund; raw pending action remains a warning. |
| Real local model | A-D each completed both tools and typed cited final JSON | A/B/D match targets; C recommends an ineligible pending-refund explanation with warning. |
| Earlier live pytest logs | A-C passed sequentially before batch cutoff; D passed separately | Superseded by the completed final batch above; C's pass tests integration, not correctness. |
| Earlier UI coverage | Offline A-D desktop and limited mobile captures | Superseded by refreshed desktop/mobile images and real live A UI evidence above. |

Environment used: **macOS, Apple M1 Max, 32 GB RAM, Python 3.13.5, Ollama 0.40.16**. Live logs identify Darwin, pytest 9.2.0, and the project's Python environment. A fresh `.venv-verify` was verified on this same machine; no Windows rerun or clean-machine setup validation is claimed. Direct dependency pins are recorded in `requirements.txt`.

## Classical A-D Comparison

Source: [classical-results.jsonl](classical-results.jsonl), produced by `python run_classical.py`. That script explicitly uses `learning.seed_counts()` and prints validated raw results without writing runtime history. Entries include the action, rationale, citations, issues, and detailed mechanism trace.

| Agent | A | B | C | D |
|---|---|---|---|---|
| Fixture target | `delivery_investigation` | `payments_escalation` | `request_clarification` | `share_tracking` |
| Simple Reflex | `delivery_investigation`, ok | `explain_refund_pending`, **warning** | `request_clarification`, ok | `share_tracking`, ok |
| Model-Based Reflex | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |
| Goal-Based | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |
| Utility-Based | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |
| Learning, seeded | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |

Simple Reflex selects B's current `pending` label without computing an SLA. The evaluator adds `Raw action explain_refund_pending is ineligible for the supplied evidence.` It does not rewrite the action. B's actual return date is Oct 29, passed inspection date Oct 30, and five-business-day deadline Nov 6; pending on Nov 13 warrants escalation under the assumed policy.

Model-Based Reflex exposes dated belief transitions followed by snapshot reconciliation, conflict records, and derived dates. Goal-Based exposes intent-derived search, prerequisites/effects, explored states, and blocked operators. Its saved plans have lengths A=3, B=5, C=2, D=4; C's requested refund goal is unreachable and its fallback is `request_clarification -> communicate_review_addressed`. These steps identify a support workflow, not completed real operations.

Utility-Based exposes eligible candidate features, contributions, exclusions, and rankings. Its selected scores are A=0.2525, B=0.2775, C=0.1025, D=0.2925 under synthetic weights `(impact, delay, effort, error risk) = (0.40, 0.25, 0.10, 0.25)`. Learning exposes counts, smoothed rates, exclusions, and deterministic tie preference. Shared eligibility limits candidates but is not a shared answer-selection function.

## Historical Local LLM Runs

The tested tag was the **original `qwen3:4b`**, not a custom tag or a smaller fallback. This table retains earlier measurements; the similarly named JSON files were replaced by the current runs listed at the top. Do not use the linked current JSON to substantiate these older timestamps or latencies. The partial/separate terminal logs below preserve the earlier execution history.

| Case / artifact | Captured At (UTC, 2026-11-14) | Raw action | Status | Latency (s) | Fixture match |
|---|---|---|---|---:|---|
| [A](llm_qwen3-4b_A.json) | 13:39:32.393595 | `delivery_investigation` | `ok` | 203.345 | Yes |
| [B](llm_qwen3-4b_B.json) | 13:41:57.272488 | `payments_escalation` | `ok` | 144.875 | Yes |
| [C](llm_qwen3-4b_C.json) | 13:45:19.173158 | `explain_refund_pending` | **`warning`** | 201.897 | **No** |
| [D](llm_qwen3-4b_D.json) | 13:47:20.502098 | `share_tracking` | `ok` | 58.074 | Yes |

Every completed case records exactly these successful, selected-case calls: `lookup_case_records({"case_id":"<case>"})` and `lookup_support_policy({"case_id":"<case>"})`. Both outputs were sent back through tool messages. Each has `retrieval_complete: true`, two invocation rounds, zero repair attempts, and a final typed citation set spanning both tools.

| Case | Cited records fields | Cited policy / policy ID | Support check and interpretation |
|---|---|---|---|
| A | `order_status` | `DNR` | `support_valid: true`; delivered Nov 10 and denial of receipt at Nov 13 support investigation within seven days; proof is optional. The citation itself is narrower than all the facts needed for this conclusion. |
| B | `refund_status`, `inspection_date` | `REFUND` | `support_valid: true`; pending, corroborated return/passed inspection, and deadline Nov 6 support escalation. Rationale states that deadline. |
| C | `refund_status` | `REFUND` | **`support_valid: false`**; field exists but its value is `"unknown"`, not pending. Missing return and passed inspection require `VERIFY` clarification/review. |
| D | `tracking` | `TRACK` | `support_valid: true`; recorded in-transit tracking and Nov 16 expected date support sharing tracking, not inventing delay. |

C's exact public rationale is **`Refund status unknown; inspection pending verification`**. Its raw action is still **`explain_refund_pending`**, with issue **`Raw action explain_refund_pending is ineligible for the supplied evidence.`** The correct next step under this simulation is clarification/review because the claimed return is unsupported, inspection is unknown, and refund status is unknown. A syntactically valid citation to `refund_status` does not prove a pending refund. There was **no forced correction**, classical substitution, or support-driven repair. All four structured outputs passed the schema/basic citation stage; only three passed action eligibility and matched their fixture targets.

This is **3/4 fixture agreement**, not an accuracy estimate or a safety/reliability result. Four authored examples and single captured completions do not establish performance on a population of support requests.

### Live Log Boundary

[live-test-results.txt](live-test-results.txt) is the earlier partial sequential A-D invocation of `tests/test_llm.py` with `-m live`. It records A, B, and C as passed and ends at D's test heading. The external command harness stopped the batch at **600 seconds**; this is not proof that D failed an assertion. D was rerun separately: [live-D-test-results.txt](live-D-test-results.txt) records **`1 passed in 58.46s`**. The completed final batch above supersedes this partial run.

The live test intentionally accepts `ok` **or** `warning` while requiring an action, both actual tools, retrieval completion, and typed citations from both tools. Thus C passed integration while failing the support/fixture comparison. The earlier partial logs are not one uninterrupted four-case pass; the final log is a completed batch, still not four correct answers.

### Runtime And Thinking Diagnosis

Current full-adapter settings in every A-D JSON and `agents/llm.py`:

```text
model=qwen3:4b
base_url=http://127.0.0.1:11434
temperature=0
reasoning=None
num_predict=2400
num_ctx=8192
client_kwargs={timeout: 180, trust_env: False}
max_rounds=6
```

The timeout is per client request, not a 180-second cap on the whole multi-round case. Case A/C totals over 180 seconds are therefore not contradictory. Shared-server loading/queue contention contributed substantially to the captured durations; these are not isolated generation-speed benchmarks. No automatic fallback model or fallback classical answer was used.

[ollama-diagnosis.md](ollama-diagnosis.md) records that the installed original tag identifies as **Qwen3 4B Thinking 2507**, `general.finetune=Thinking`, with thinking-only metadata. A `think=false` short probe returned reasoning prose and hit its length limit despite HTTP 200; another `reasoning=False` tools probe timed out, which alone cannot prove eventual tool behavior. The custom tag's edited Go template was not the selected template. No custom model/template was needed for the demonstrated original-tag path.

The diagnostic's working configuration used `reasoning=None`, 1800 tokens, context 4096, and timeout 100 with `trust_env=False`. Its passing two-tool evidence is `diagnostic-qwen3-4b-none-tools-1800-1794663110942812000.json`. Those diagnostic settings and smaller final schema differ from the current 2400/8192/180 full adapter. The historical [smoke_qwen3-4b.json](smoke_qwen3-4b.json) is marked as initial minimal integration with a smaller contract. Private reasoning was removed; its wrong public recommendation (no refund rather than the required investigation) is retained. It is not evidence that the final adapter or live UI worked at that stage. Neither historical artifact supersedes the current A-D traces.

## Offline Verification And Learning

[offline-test-results.txt](offline-test-results.txt) contains **`270 passed, 4 deselected in 2.35s`**, with no expected failures reported. Reproduce separately from real inference with `python -m pytest -m "not live" -q`.

A fresh Python **3.13.5** `.venv-verify` (without system site packages) installed the pinned `requirements.txt`; [fresh-install.txt](fresh-install.txt) records successful installation. [fresh-test-results.txt](fresh-test-results.txt) records **`270 passed, 4 deselected in 13.50s`** in that environment. This verifies a fresh venv on the same macOS machine, not a new machine, Windows, or fresh live inference.

Tests cover immutable/isolation contracts, supplied-case/tool parity, business-day Friday/Saturday/Monday semantics, DNR boundaries and proof, missing/future/conflicting evidence, history regressions, distinct belief transitions, BFS reachability/minimum paths, weight-sensitive utility, eligible learning rankings, malformed/unknown/wrong-case/error tool calls, output/citation validation, bounded repair/exhaustion, unavailable Ollama, session invalidation, and feedback behavior. Fake LLM clients verify control flow; they are not real model evidence.

The learner's nine fixed contexts each initialize **all eight actions to 0 successes / 0 failures**, except `timely_nonreceipt|delivery_investigation` at **6/3** and `timely_nonreceipt|human_review` at **5/4**. These two nonzero entries are synthetic demo outcomes. B/C/D start with neutral eligible estimates and use declared tie-breaking, not fabricated successful experience.

| Explicit A feedback state | Investigation counts | Investigation smoothed estimate | Review estimate | Selected action |
|---|---|---|---|---|
| Seed | 6 successes, 3 failures | `7/11` | `6/11` (5 successes, 4 failures) | `delivery_investigation` |
| One failure, rerun | 6 successes, 4 failures | `7/12` | `6/11` | `delivery_investigation` |
| Second failure, rerun | 6 successes, 5 failures | `7/13` | `6/11` | `human_review` |

The adaptation/reload/reset test and Streamlit two-feedback test exercise that switch using temporary files. Actual app feedback persists to **`runtime/learning_counts.json`**; absent history uses the seed without writing, explicit feedback/reset creates or updates the file, and reruns do not duplicate outcomes. The three-stage browser sequence is documented above. These are simulated explicit outcomes, not real customer outcomes. No runtime history file is required for the saved seed-based classical comparison.

## Screenshot Scope

| File | What it documents |
|---|---|
| [scenario_A_offline.png](scenario_A_offline.png) | Desktop A comparison; five classical recommendations; LLM `not_run` |
| [scenario_B_offline.png](scenario_B_offline.png) | Desktop B offline comparison, including the reflex limitation |
| [scenario_C_offline.png](scenario_C_offline.png) | Desktop C offline comparison |
| [scenario_D_offline.png](scenario_D_offline.png) | Desktop D offline comparison |
| [mobile_initial.png](mobile_initial.png) | New separate browser context at 390x844; initial A page, collapsed sidebar, no comparison run |

See Current Results for expanded mechanism details, real live A-D inference, the mobile A comparison, and browser feedback progression.

## Limitations

- Authored structured claims drive classical reasoning; raw customer text does not imply an implemented general NLP parser.
- Policies, utility estimates, and initial learning outcomes are synthetic assumptions, not empirical commerce policy or calibrated probabilities. Public holidays are not modeled.
- Tool execution and basic citation checks establish transport/reference validity, not semantic entailment. C remains a concrete counterexample to treating clean citations as proof of a supported answer.
- A shared frozen case provides common available evidence, not identical processing. Learning also consumes disclosed history; the reflex intentionally omits temporal reasoning.
- No real refund, replacement, order mutation, carrier contact, or customer communication occurs. Plans and action labels are recommendations only.
- Single captured A-D completions do not measure robustness, broad accuracy, safety, or repeatability. Model-tag contents, runtime versions, hardware, and contention can change results.
- The earlier live batch was partial; it has been superseded by a completed A-D batch and separate real A-D UI evidence. Neither establishes complete mobile usability. Fresh-venv installation/tests were verified on the same machine; Windows and fresh-machine setup were not verified.
