# E-commerce Order and Refund Support Assistant

Agentic AI (S2-25_PCAMZG519), Assignment 1, Set 6 | **Group 21**

| Member | Student ID |
|---|---|
| G.Chandramouli | `2025paml525` |
| Atindra R K Ranjana Singh | `2025paml584` |

A Streamlit simulator that compares five classical agents (Simple Reflex, Model-Based Reflex, Goal-Based, Utility-Based, Learning) and one local LangChain/Ollama LLM agent on the same synthetic order/refund case. Agents only recommend support actions. They never issue refunds or change orders.

## Setup

Requires Python 3.13.

```bash
python3.13 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

## Run

```bash
python -m streamlit run app.py          # UI at http://localhost:8501
python -m pytest -m "not live" -q       # offline tests, no Ollama needed
python run_classical.py                 # the five classical agents on cases A-D
```

In the UI, pick scenario A, B, C or D and click **Run Comparison**. Uncheck **Run real local LLM** to run without Ollama.

## Optional: local LLM

Install [Ollama](https://ollama.com/download) and make sure it is serving (`ollama serve` if the desktop app isn't running), then:

```bash
ollama pull qwen3:4b
python -m pytest tests/test_llm.py -m live -v -s    # real model calls, can take minutes
```

## Layout

`agents/` (six agents) | `app.py` (UI) | `domain.py`, `contracts.py`, `cases.py`, `fixtures.json` (shared case data and policies) | `tests/`
