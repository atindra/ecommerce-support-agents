# Results And Evidence

## Current Results (2026-11-14)

Start here for evidence of the final verified implementation. These are recorded runs, not new verification of subsequent documentation edits. They supersede the [historical measurements](#historical-verification-superseded) wherever they differ. Early diagnostics and older logs remain supplementary; no evidence files have been moved or removed.

| Check | Actual result | Evidence / boundary |
|---|---|---|
| Latest offline suite | 275 passed, 4 live tests deselected in 2.45s | [Offline verification log](verification-offline-test-results.txt); includes the stable saved-C replay |
| Fresh-package verification | 275 passed, 4 deselected in 15.54s; classical runner succeeded | [Package test log](package-offline-test-results.txt), [verification scope](#clean-package-verification); fresh venv on the same machine |
| Real Ollama/LangChain A-D | 4 integration tests passed; A/B/D match, C fails support | [Completed batch](final-live-test-results.txt); tool traces: [A](llm_qwen3-4b_A.json), [B](llm_qwen3-4b_B.json), [C (warning)](llm_qwen3-4b_C.json), [D](llm_qwen3-4b_D.json) |
| Live UI screenshots | Separate real A-D inference | [A](scenario_A_live.png), [B](scenario_B_live.png), [C (warning)](scenario_C_live.png), [D](scenario_D_live.png); not replays of pytest results |
| Six-agent UI exports | Decisions, reasons, and full traces from those UI runs | [A](final-ui-A-comparison.jsonl), [B](final-ui-B-comparison.jsonl), [C](final-ui-C-comparison.jsonl), [D](final-ui-D-comparison.jsonl) |
| Classical agents | 20 seed-based results; Simple Reflex misses B | [Comparison](#classical-a-d-comparison), [raw results](classical-results.jsonl), [mechanisms and source](../README.md#five-mechanisms) |
| Learning | Initial / first failure / second failure switch; reload/reset also tested | [Demonstration](#learning-demonstration); three screenshots and matching exports, with LLM disabled |
| Offline UI | A-D desktop comparisons and A mobile capture | [Screenshot scope](#screenshot-scope); not exhaustive platform testing |

Read the [LLM evaluation](#real-llm-evaluation) for C's unsupported recommendation and the [limitations](#limitations) for remaining gaps. Integration passing is not recommendation correctness.

## Classical A-D Comparison

Source: [classical-results.jsonl](classical-results.jsonl), produced by `python run_classical.py`. The script uses `learning.seed_counts()` without reading adapted history or writing feedback. Entries preserve actions, rationales, citations, issues, and mechanism traces.

| Agent | A | B | C | D |
|---|---|---|---|---|
| Fixture target | `delivery_investigation` | `payments_escalation` | `request_clarification` | `share_tracking` |
| Simple Reflex | `delivery_investigation`, ok | `explain_refund_pending`, **warning** | `request_clarification`, ok | `share_tracking`, ok |
| Model-Based Reflex | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |
| Goal-Based | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |
| Utility-Based | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |
| Learning, seeded | `delivery_investigation`, ok | `payments_escalation`, ok | `request_clarification`, ok | `share_tracking`, ok |

Simple Reflex reads B's current `pending` label without SLA arithmetic. With return Oct 29, passed inspection Oct 30, and a five-business-day deadline of Nov 6, pending on Nov 13 warrants escalation. Validation retains the raw answer and adds `Raw action explain_refund_pending is ineligible for the supplied evidence.`

The [README explains each mechanism](../README.md#five-mechanisms); the raw results show belief transitions, BFS operators/plans, utility contributions, and learning estimates. Saved goal plans have lengths A=3, B=5, C=2, D=4; C's requested refund goal is unreachable and falls back to clarification. Utility scores are A=0.2525, B=0.2775, C=0.1025, D=0.2925 under the documented synthetic weights. Shared eligibility limits candidates but does not select a common answer.

## Real LLM Evaluation

The [final live batch](final-live-test-results.txt) records **4 passed, 64 deselected in 243.14s**, one completed sequential A-D run using original `qwen3:4b`. Each trace contains both successful model-initiated tools, returned evidence, typed citations, two invocation rounds, and zero repairs.

| Case | Raw action | Status | Latency seconds | Support/fixture |
|---|---|---|---:|---|
| A | delivery_investigation | ok | 45.712 | Supported / match |
| B | payments_escalation | ok | 129.840 | Supported / match |
| C | explain_refund_pending | warning | 37.757 | Ineligible / mismatch |
| D | share_tracking | ok | 29.633 | Supported / match |

**Why C fails:** `refund_status` exists but is `unknown`, and the claimed return and passed inspection are unverified. A valid citation therefore does not support explaining a pending refund; clarification/review is required. The adapter keeps the model's raw action and flags it as a warning, without classical substitution or support-driven repair. The live test accepts `ok` or `warning` while requiring both tools and typed citations. Thus this is **3/4 fixture agreement, not accuracy** or a reliability estimate.

Execution, reference validity, and policy support are separate checks; see [LLM Output And Limits](../README.md#llm-output-and-limits). Compatible policy and relevant fields do not prove every rationale claim or explicitly cite every prerequisite. The saved-C offline regression preserves this distinction through the real adapter with substitute model messages.

### Separate Live UI Runs

Browser verification used separately owned Streamlit instances on port 18517 and Chrome via optional Playwright, without modifying the shared Ollama service. [A's screenshot](scenario_A_live.png) and [six-agent export](final-ui-A-comparison.jsonl) retain its genuine run. The other UI measurements are:

| UI case | Raw action / status | Latency seconds | Evidence |
|---|---|---:|---|
| B | `payments_escalation` / `ok` | 58.949 | [Screenshot](scenario_B_live.png), [six-agent export](final-ui-B-comparison.jsonl) |
| C | `explain_refund_pending` / **`warning`** | 33.150 | [Screenshot](scenario_C_live.png), [six-agent export](final-ui-C-comparison.jsonl) |
| D | `share_tracking` / `ok` | 25.926 | [Screenshot](scenario_D_live.png), [six-agent export](final-ui-D-comparison.jsonl) |

These B/C/D runs used both tools, complete retrieval, typed citations, two rounds, and zero repairs. Their outcomes agree with the pytest batch, including C's failure, but their timings belong to separate runs. No favorable-output selection or retry was made. Screenshots focus on expanded LLM outcomes; downloads preserve all six rows and full traces.

## Learning Demonstration

The browser sequence used actual feedback buttons and a new comparison between clicks, without LLM calls. Initial counts are synthetic: all eight actions in each of nine fixed contexts start at 0/0, except A's investigation at 6/3 and review at 5/4. B/C/D use neutral estimates and deterministic ties, not fabricated successful experience.

| Explicit A feedback state | Investigation counts | Investigation estimate | Review estimate | Selection | Evidence |
|---|---|---|---|---|---|
| Seed | 6/3 | `7/11` | `6/11` (counts 5/4) | Investigation | [Screenshot](learning_A_stage_0.png), [export](learning_A_stage_0.jsonl) |
| First failure | 6/4 | `7/12` | `6/11` | Investigation | [Screenshot](learning_A_stage_1.png), [export](learning_A_stage_1.jsonl) |
| Second failure | 6/5 | `7/13` | `6/11` | Human review | [Screenshot](learning_A_stage_2.png), [export](learning_A_stage_2.jsonl) |

Exports preserve full-precision counts and estimates. Feedback persists to `runtime/learning_counts.json`; absent history uses seeds without writing, ordinary reruns do not duplicate outcomes, and reset restores seeds. Adaptation/reload/reset and the two-feedback UI sequence are tested with temporary files. Follow the [README learning example](../README.md#learning-example) to reproduce it.

The original absent runtime state was restored after capture. Screenshot results are unchanged; blank bottom space was cropped and the personal-path caption masked in neutral grey.

## Screenshot Scope

| File | What it documents |
|---|---|
| [scenario_A_offline.png](scenario_A_offline.png) | Desktop A comparison; five classical recommendations; LLM `not_run` |
| [scenario_B_offline.png](scenario_B_offline.png) | Desktop B comparison, including the reflex limitation |
| [scenario_C_offline.png](scenario_C_offline.png) | Desktop C offline comparison |
| [scenario_D_offline.png](scenario_D_offline.png) | Desktop D offline comparison |
| [mechanisms_A_offline.png](mechanisms_A_offline.png) | Expanded utility/learning details |
| [scenario_A_mobile_offline.png](scenario_A_mobile_offline.png) | A comparison at 390x844, sidebar closed; wide tables scroll horizontally |

Live screenshots and learning stages are linked above. This is limited rendering verification, not exhaustive mobile usability testing. The [initial mobile page](mobile_initial.png) is a historical capture with no comparison run.

For optional capture, install Playwright in a separate tools environment (`python3.13 -m venv ../capture-tools`, then `../capture-tools/bin/python -m pip install playwright`); installed Chrome is required. Run Streamlit in its application environment, then `../capture-tools/bin/python capture_screenshots.py --url http://127.0.0.1:18517 --skip-offline --live --live-cases B C D --learning`. Use a new evidence copy to avoid overwriting captures. Omit `--live` for no model calls. The script does not replay evidence; `--learning` temporarily resets and restores history. Do not run it concurrently with another feedback writer.

## Clean Package Verification

The recorded submission ZIP was extracted into a new directory and dependencies installed from its `requirements.txt` in a fresh Python 3.13.5 venv without system site packages. Offline tests passed **275 passed, 4 deselected in 15.54s** ([log](package-offline-test-results.txt)); `python run_classical.py` also succeeded.

Checks covered archive CRC/integrity, local Markdown targets and anchors, A-D images/exports, all three learning stages, and absence of personal absolute paths and runtime feedback. Packaging excluded environments, caches, bytecode, runtime history, and backups. This verifies the recorded package on the same machine, not a later ZIP rebuild, fresh-machine/Windows setup, or new inference.

## Runtime Settings

Recorded environment: **macOS, Apple M1 Max, 32 GB RAM, Python 3.13.5, Ollama 0.40.16**, original `qwen3:4b` tag (Qwen3 4B Thinking 2507). Dependencies are pinned in [requirements.txt](../requirements.txt). Full-adapter settings in the A-D traces and [agents/llm.py](../agents/llm.py):

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

The timeout is per request, not a whole-case deadline. Shared-server loading/queue contention affected timings; they are not isolated generation-speed benchmarks. There is no automatic model or classical-answer fallback. Earlier runtime investigations are [supplementary diagnostics](#runtime-and-thinking-diagnosis).

## Limitations

- Authored structured claims drive classical reasoning; raw customer text does not imply general NLP parsing.
- Policies, utility features, and learning seeds are synthetic, not empirical commerce policy or calibrated probabilities. Fixed-context learning does not generalize; public holidays are not modeled.
- Citation validity and action/policy checks are not full semantic entailment; C is the concrete counterexample described above.
- Shared frozen cases provide common evidence, not identical processing. Learning also consumes disclosed history; the reflex deliberately omits temporal reasoning and misses B.
- Learning persistence is a single-user file, not atomic or concurrency-safe storage. Feedback is explicit and simulated, not assessed real customer outcomes.
- No real refund, replacement, order mutation, carrier contact, or customer communication occurs. Plan effects and action labels are recommendations only. Public traces contain rationale and tool/validation evidence, not private reasoning.
- Captured A-D completions do not establish broad accuracy, robustness, safety, or repeatability. Model contents, runtime versions, hardware, and load can change results. Windows, fresh-machine setup, and exhaustive mobile usability remain unverified.

## Historical Verification (Superseded)

The following records preserve development history from **2026-11-14**, not final-result claims. All A-D fixtures evaluate simulated records as of **2026-11-13**. Historical installation/live logs replace personal filesystem paths with `<project-directory>` without changing recorded results or timings.

### Earlier Offline Verification

- [Original offline log](offline-test-results.txt): **270 passed, 4 deselected in 2.35s**, with no expected failures.
- [Fresh installation](fresh-install.txt) and [fresh-venv test log](fresh-test-results.txt): pinned dependencies installed in Python 3.13.5 `.venv-verify` without system site packages; **270 passed, 4 deselected in 13.50s**, on the same macOS machine.
- [Earlier final offline log](final-offline-test-results.txt): **274 passed, 4 deselected in 3.53s**. Covers feedback/reset, mechanism/weight-sensitivity UI, policy compatibility, and citation relevance regressions. Two initial fake answers cited unrelated order status for refund/tracking; their test inputs were corrected rather than weakening checks.
- [Earlier offline log](submission-offline-test-results.txt): **274 passed, 4 deselected in 2.57s**.

The 275-test result at the top supersedes these counts. Tests cover contracts/isolation, shared-case tool parity, policy/date boundaries, missing/conflicting/history evidence, belief replay, BFS paths, utility sensitivity, learning persistence/ranking, tool/output failures, bounded repair, unavailable Ollama, session invalidation, and feedback. Substitute clients test control flow, not real-model reliability.

### Historical Local LLM Runs

These earlier measurements used original `qwen3:4b`. Their similarly named JSON files were subsequently replaced by the final runs linked at the top; current JSON does not substantiate these older timestamps or latencies.

| Historical case | Captured At (UTC, 2026-11-14) | Raw action | Status | Latency (s) | Fixture match |
|---|---|---|---|---:|---|
| A | 13:39:32.393595 | `delivery_investigation` | `ok` | 203.345 | Yes |
| B | 13:41:57.272488 | `payments_escalation` | `ok` | 144.875 | Yes |
| C | 13:45:19.173158 | `explain_refund_pending` | **`warning`** | 201.897 | **No** |
| D | 13:47:20.502098 | `share_tracking` | `ok` | 58.074 | Yes |

Each completed case called `lookup_case_records` and `lookup_support_policy` for its selected ID, returned evidence through tool messages, and recorded complete retrieval, two rounds, zero repairs, and citations spanning both tools. Earlier citation/support observations were:

| Case | Cited records | Policy | Interpretation |
|---|---|---|---|
| A | `order_status` | `DNR` | Supported; the citation alone is narrower than all investigation prerequisites |
| B | `refund_status`, `inspection_date` | `REFUND` | Supported escalation; rationale states the Nov 6 deadline |
| C | `refund_status` | `REFUND` | Unsupported; public rationale was `Refund status unknown; inspection pending verification`, but raw action remained pending explanation |
| D | `tracking` | `TRACK` | Supported sharing of recorded tracking |

### Live Log Boundary

[live-test-results.txt](live-test-results.txt) records the earlier partial sequential batch: A-C passed, then the harness stopped at D's heading after **600 seconds**. This is not evidence of a D assertion failure. [D's separate rerun](live-D-test-results.txt) records **1 passed in 58.46s**. These are not one uninterrupted four-case pass; the final completed batch supersedes them. The integration-versus-correctness distinction is explained in [Real LLM Evaluation](#real-llm-evaluation).

### Runtime And Thinking Diagnosis

[ollama-diagnosis.md](ollama-diagnosis.md) records original-tag thinking-only metadata (`general.finetune=Thinking`). A `think=false` short probe returned reasoning prose and hit its length limit despite HTTP 200; a `reasoning=False` tools probe timed out, which alone does not establish eventual tool behavior. The custom tag's edited Go template was not selected; no custom model/template was needed for the demonstrated original-tag path. External runtime-log line references are historical observations; the full server log is not bundled.

The diagnostic used `reasoning=None`, 1800 tokens, context 4096, and timeout 100 with `trust_env=False`. Its [successful two-tool probe](diagnostic-qwen3-4b-none-tools-1800-1794663110942812000.json) has smaller settings and schema than the final adapter. Other `diagnostic-*.json` files remain available as early probes, not final evaluation evidence.

The [initial smoke result](smoke_qwen3-4b.json) used a smaller contract. Private reasoning was removed, but its wrong public recommendation (no refund instead of investigation) remains. Neither the smoke result nor diagnostic probes supersede final A-D evidence.
