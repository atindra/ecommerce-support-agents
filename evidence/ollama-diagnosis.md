# Ollama Diagnosis

Verified 2026-11-14 against the existing localhost:11434 server, version 0.40.16.
No server restart, model creation, Modelfile change, or application/agent edit.

## Proven Configuration

```python
ChatOllama(
    model="qwen3:4b",
    base_url="http://127.0.0.1:11434",
    temperature=0,
    reasoning=None,
    num_predict=1800,
    num_ctx=4096,
    client_kwargs={"timeout": 100, "trust_env": False},
).bind_tools([lookup_records, lookup_policy])
```

Use the original tag and its original template. No custom template needed.
Keep the assistant tool-call message and append a ToolMessage for each actual
execution, preserving tool_call_id, before invoking again. Do not apply a final
JSON grammar to the initial tool-selection turn. This test uses a JSON instruction
and json.loads plus semantic validation, not constrained decoding.

Reproduce from this project directory:

```sh
.venv/bin/python diagnose_ollama.py --reasoning none --tokens 1800
```

Passing evidence: diagnostic-qwen3-4b-none-tools-1800-1794663110942812000.json.
Both functions were selected by the model, executed locally with case_id=A,
and their returned data supplied to the model. The first generation used 600
tokens; the final generation used 206 tokens. Both ended with done_reason=stop.
The final JSON was:

```json
{"case_id":"A","action":"investigate","refund_allowed":false}
```

The prompt specified the decision rule, not this case's result. A previous,
less explicit prompt also executed both tools and produced valid JSON but chose
"refund denied" instead of "investigate". Thus tool transport works; decision
quality still depends on precise instructions and validation.

## Findings

- /api/show identifies these actual weights as Qwen3 4B Thinking 2507, with
  general.finetune=Thinking and thinking.values=[true], default=true.
- think=false was accepted with HTTP 200, but a 64-token single-word probe
  returned reasoning prose in message.content and done_reason=length, not OK.
  See diagnostic-qwen3-4b-false-raw-64.json for exact headers/body. Disabling the
  client's reasoning handling does not make these weights a nonthinking model.
- The 1800-token reasoning=False tool probe timed out before any response.
  See diagnostic-qwen3-4b-false-tools-1800-1794663217334950000.json. This timeout
  cannot independently establish whether that setting eventually calls tools.
- The custom support-qwen3-4b tag reports template_source=gguf_chat_template,
  unlike the original tag's go_template. Runtime log line 7715 explicitly says
  selected=gguf_chat_template, go_template="[completion tools]", and
  chat_template="[tools thinking completion]". The edited closed-thinking Go
  template therefore was not selected for that custom tag.
- Shared-server scheduling matters. The successful final call took 91.95 s,
  including 85.35 s reported load_duration but only 5.83 s generation. A timeout
  is not evidence that the model cannot use tools. Longer client timeouts may
  be needed under contention; 100 s was the tested setting, not a guarantee.
- /api/ps reported context_length=32768 despite requests explicitly carrying
  num_ctx=4096. Logs show both 32768 and 4096 runner launches across requests.
  Concurrent requests and runner reuse prevent attributing every loaded runner
  to this probe. This is not a demonstrated cause of the tool failure, and the
  passing test succeeded despite the observation.
- Earlier log lines 8078-8080 show client disconnect, canceled load, and HTTP
  499 after 2m20s. That is a concrete possible explanation for a curl request
  yielding no body, not proof about an unidentified earlier curl invocation.

The reliable demonstrated path is original qwen3:4b, reasoning=None, sufficient
generation budget, correct ToolMessage round trip, and explicit decision rules.
No smaller-model fallback was attempted.
