# 💬 Day 2: Multi-Turn Conversational Memory, Multilingual Prompts & Tool Helpers

**Notebook:** [day2.ipynb](./day2.ipynb)

Day 2 evolves the **Sweet Crumbs Home Bakery** assistant from a single-turn Q&A bot into a **stateful, multi-turn conversational chatbot** with a structured menu, logging decorator, price calculation helper, error handling, and multilingual support.

---

## 🎯 What You Learn in Day 2

1. **OpenAI-Compatible Groq Client**
   - Connecting directly to Groq using the standard `openai` Python SDK by pointing `base_url` to `https://api.groq.com/openai/v1` with model `openai/gpt-oss-120b`.

2. **Structured Domain Data & Enhanced System Prompt**
   - Embedding a live `MENU` dictionary (prices in INR ₹) into the `SYSTEM` prompt:
     - Plum cake (₹500), Chocolate cake (₹650), Cupcake (₹80), Pineapple cake (₹200), Red Velvet cake (₹800).
   - Structured prompt sections (**ROLE**, **FACTS**, **RULES**, **FORMAT**):
     - Multilingual replies matching the customer's language (**English**, **Telugu**, or **Hindi**).
     - Concise responses under **40 words**.
     - Escalation rules: `"I'll check with the owner."` for unknown facts, and `"contact to our team."` when asked for invoices.

3. **Python Decorators & Price Calculation Helper**
   - `@log_call` decorator using `functools.wraps` to trace function calls and return values.
   - `get_price(item: str, qty: int = 1) -> int` helper to compute total item prices from `MENU`.

4. **Multi-Turn Conversation Memory (Sliding Window)**
   - Maintaining conversation context with `history` (`[{"role": "system", "content": SYSTEM}, ...]`).
   - Sliding window (`MAX_TURNS = 8`) keeping the system prompt anchored plus the last 8 messages (`[history[0]] + history[1:][-MAX_TURNS:]`).

5. **Resilient API Error Handling & CLI Chat Loop**
   - Gracefully catching `RateLimitError` and `APIConnectionError` inside `chat(user_text)`.
   - Interactive `while True:` terminal loop supporting `bye`, `exit`, and `quit` to end the session.

---

## ▶️ How to Run

1. Add your `GROQ_API_KEY` in Google Colab Secrets (or adapt `userdata.get("GROQ_API_KEY")` to `os.environ.get("GROQ_API_KEY")` with `dotenv` for local runs).
2. Open [day2.ipynb](./day2.ipynb) and run the cells to start chatting with the bakery bot.
