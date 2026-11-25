# 🧁 Day 1: LLM Foundations, Prompt Engineering & Prompt Guardrails

**Notebook:** [sweet_crumbs_bakery_day1.ipynb](./sweet_crumbs_bakery_day1.ipynb)

Day 1 introduces building a domain-specific AI assistant using **LangChain + Groq** along with **prompt injection detection** using Meta's Llama Prompt Guard and an interactive Jupyter UI.

---

## 🎯 What You Learn in Day 1

1. **Fast LLM Inference with LangChain & Groq**
   - Installing and using `langchain_groq` (`ChatGroq`) with `openai/gpt-oss-120b` (`temperature=0`).
   - Flexible API key loading across environments:
     1. Local `.env` file (`python-dotenv`)
     2. Google Colab Secrets (`google.colab.userdata`)
     3. Interactive fallback prompt (`getpass`)

2. **System Prompt Engineering**
   - Constraining the assistant's behavior using a structured `SYSTEM` prompt for **Sweet Crumbs Home Bakery**:
     - **Facts**: Bake to order, 24 hours' notice required, eggless versions available for all cakes.
     - **Rules**: Never promise delivery times; admit when something is unknown.
     - **Format**: Polite replies under 60 words.
   - Single-turn invocation via `ask(question)`.

3. **AI Safety & Prompt Injection Guardrails**
   - Using the `groq` Python SDK with `meta-llama/llama-prompt-guard-2-22m` to score user inputs for prompt injection / jailbreak attempts.
   - Classifying inputs with score `< 0.5` as **SAFE** and `>= 0.5` as **UNSAFE**.

4. **Interactive Widget Tester (`ipywidgets`)**
   - A visual notebook interface to switch between **Bakery LLM** and **Prompt Guard** modes.
   - Quick-test preset buttons for order notice, eggless cakes, and prompt injection testing.

---

## ▶️ How to Run

1. Make sure `GROQ_API_KEY` is set in the root `.env` file (or enter it when prompted by `getpass`).
2. Open [sweet_crumbs_bakery_day1.ipynb](./sweet_crumbs_bakery_day1.ipynb) in Jupyter / VS Code / Google Colab and run the cells in order.
