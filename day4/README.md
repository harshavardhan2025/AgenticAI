# 🛠️ Day 4: Tools, Structured Output & Google Sheets

**Notebook:** [day4.ipynb](./day4.ipynb)  
**Level 2 · Single Agent · Session 2 of 3**  
**Case Study:** Sweet Crumbs Home Bakery

Yesterday the assistant could only talk. Today it gets **hands**: Python functions (`@tool`) it can call by itself to check prices and opening hours, validate messy chat into clean **Pydantic** records, and save confirmed orders directly into **Google Sheets**.

---

## 🎯 Today at a Glance

| # | Topic | Focus |
|---|---|---|
| 1 | **Why Agents Need Tools** | Live facts, real actions, and exact maths |
| 2 | **How Function Calling Works** | Model requests a tool → your code runs it → result goes back |
| 3 | **Writing Good Tools** | `@tool`, clear `snake_case` names, type hints, docstrings |
| 4 | **The Agent with Tools** | `create_agent`, running the loop, reading the trace |
| 5 | **Rate Limits & Tool Errors** | `429` errors, `max_retries`, `try/except` polite fallbacks |
| 6 | **Structured Output with Pydantic** | `BaseModel`, `Field`, `Literal`, `with_structured_output` |
| 7 | **Saving to Google Sheets** | `gspread` in Colab (`ws.append_row`) |
| 8 | **Putting It Together** | `log_order` tool: chat → checked order → sheet row |

---

## 🧠 Core Concepts (Simple Guide)

### 1. Why Agents Need Tools
An LLM only produces text from what it learned and what is in its prompt. It cannot check today's prices, read your database, do reliable maths on large numbers, or save an order.
- **Model alone**: Creative writing, explanations, translations.
- **Needs a tool**: Live facts (`get_price`, `is_open`), real actions (`log_order`), and exact maths.

### 2. How Function Calling Works
The model **never runs your code**. Instead:
1. You send tool descriptions (name, description, parameters) with the messages.
2. The model replies with a **tool call** (e.g. `get_price(item="plum cake", qty=3)`).
3. Your Python code executes the function and adds the output as a **tool message**.
4. The model reads the tool result and writes the final reply.

### 3. Writing Good Tools (`@tool`)
- Use `@tool` from `langchain.tools`.
- Give a clear `snake_case` name (`get_price`, not `tool1`).
- Add **type hints** on all parameters and a **docstring** explaining *when* to use it (with examples).
- Keep one job per tool and return a helpful message on bad input (like an item not on the menu) instead of crashing.

### 4. The Agent with Tools (`create_agent`)
```python
agent = create_agent(model=llm, tools=[get_price, is_open, log_order], system_prompt=SYSTEM)
result = agent.invoke({"messages": [{"role": "user", "content": "..."}]})
```
- Final answer: `result["messages"][-1].content`
- Full trace (question → tool calls → tool results → answer): `for m in result["messages"]: m.pretty_print()`

### 5. Rate Limits & Tool Errors
- **Error `429`**: Too Many Requests (rate limit exceeded).
- Use `max_retries=3` on `ChatGroq`, keep prompts/tests small, use your own API key, and wrap `agent.invoke(...)` in `try/except` so customers see a polite message if something fails.

### 6. Structured Output with Pydantic
Pydantic acts like a form validator before data reaches your sheet:
```python
class Order(BaseModel):
    name: str = Field(description="Customer's first name")
    item: Literal["plum cake", "chocolate cake", "cupcake"] = Field(description="Menu item")
    qty: int = Field(description="How many", ge=1)
    date: str = Field(description="Pickup date as the customer said it")
    eggless: bool = Field(description="True if the customer asked for eggless")
```
- `llm.with_structured_output(Order)` returns a validated `Order` object (or raises `ValidationError` on invalid values like `item="donut"` or `qty=0`).

### 7. Saving to Google Sheets (`gspread`)
In Google Colab:
1. `auth.authenticate_user()` → sign in with Google
2. `gc = gspread.authorize(creds)` → authorize client
3. `sh = gc.create("Sweet Crumbs Orders")` (first time) / `gc.open("Sweet Crumbs Orders")` (later)
4. `ws.append_row([name, item, qty, date, eggless, total])` → saves one order as one row.

### 8. Putting It Together (`log_order`)
`log_order` validates the order with `Order(...)` inside the tool, calculates `total = MENU[order.item] * order.qty`, appends the row to Google Sheets, and confirms back to the customer. If details (like pickup date) are missing, the agent asks the customer first.

---

## 🛠️ Today's Build

The Sweet Crumbs agent in [day4.ipynb](./day4.ipynb) uses:
- **`get_price(item, qty)`**: Looks up `MENU` and returns exact total in Rs.
- **`is_open(day)`**: Checks `HOURS` for a day of the week.
- **`log_order(name, item, qty, date, eggless)`**: Validates with Pydantic `Order`, calculates total, saves to Google Sheets, and confirms.

---

## 📝 Your 30% Practice Tasks

1. **Your tools**: Convert your two Day 3 homework functions into `@tool` functions with clear docstrings.
2. **Your record**: Create a Pydantic `BaseModel` for your business and test `with_structured_output` on 5 messy messages.
3. **Your sheet**: Create a Google Sheet and a `log_...` tool that saves your record.
4. **Break it**: Rename one tool to `tool1` with docstring `"does stuff"` to see if the agent still picks it, then change it back.
5. **Read the trace**: Use `pretty_print()` to count tool calls and model calls for a question.
6. **Before Day 5**: Write 15–20 facts about your business (policies, timings, delivery areas, FAQs) in a text file for RAG.

---

## 📊 Summary Table

| Idea | One Line | Analogy |
|---|---|---|
| **Tools** | Functions for live facts, actions and exact work | Phone and register for the exam student |
| **Function calling** | Model asks, your code runs, result goes back as a message | Waiter and kitchen |
| **Good tools** | `snake_case` name, type hints, docstring with "when to use" | Labelled kitchen drawers |
| **create_agent** | Model + tools + prompt; the trace shows every step | Delivery tracking timeline |
| **Rate limits** | `429` = slow down; own key, small tests, `try/except` | Toll plaza |
| **Pydantic** | Fields + types + rules; rejects bad data | Deposit slip checked at the counter |
| **Google Sheets** | `append_row` = one order, one row | A register that fills itself |
| **log_order** | Check → total → save → confirm, only when complete | Careful billing clerk |
