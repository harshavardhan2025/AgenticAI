# 🤖 Day 3: Your First Agent — What an Agent Is & LangChain

**Notebook:** [day3.ipynb](./day3.ipynb)  
**Level 2 · Single Agent · Session 1 of 3**  
**Case Study:** Sweet Crumbs Home Bakery

A chatbot talks. An agent gets things done. Day 3 covers what turns a chatbot into an agent, how an agent thinks in a loop, when you should (and shouldn't) use one, how **LangChain** simplifies the code, and how to test your assistant using `tests.csv`.

---

## 🎯 Today at a Glance

| # | Topic | Focus |
|---|---|---|
| 1 | **Chatbot vs Agent** | Talking vs doing (goals, tools, actions, checks) |
| 2 | **The Four Parts of an Agent** | Brain, instructions, tools, memory |
| 3 | **The Agent Loop (ReAct)** | Think → act → observe → repeat (with a round limit) |
| 4 | **Three Levels of AI** | Traditional software, Generative AI, Agentic AI |
| 5 | **Agent or Not?** | Pick the simplest thing that works |
| 6 | **Why LangChain** | One universal adapter for every model + agent building blocks |
| 7 | **LangChain Basics** | Chat models (`invoke`), `ChatPromptTemplate`, LCEL chains (`\|`) |
| 8 | **Testing Your Assistant** | Automated evals with `tests.csv` and pass rate |

---

## 🧠 Core Concepts

### 1. Chatbot vs Agent
- **Chatbot**: Answers with words only (e.g., explains how to cancel an order).
- **Agent**: Given a goal, uses **tools** and decides the steps itself until the goal is done (e.g., looks up the order, cancels it, verifies it worked, and confirms).

### 2. The Four Parts of an Agent
| Part | What It Is | At Sweet Crumbs |
|---|---|---|
| **Brain** | The LLM that reads and decides | `openai/gpt-oss-20b` on Groq |
| **Instructions** | System prompt (role, facts, rules, format) | `"You are the friendly assistant for Sweet Crumbs..."` |
| **Tools** | Python functions the agent can call | `get_price`, `is_open`, `log_order` (Day 4) |
| **Memory** | What it keeps between steps and messages | `MessagesPlaceholder("history")` |

### 3. The Agent Loop (ReAct: Reason + Act)
1. **Think**: Decide what is needed next.
2. **Act**: Call a tool (your Python code executes the tool).
3. **Observe**: Read the tool result returned to the model.
4. **Repeat**: Continue until a final answer is reached or a maximum round limit is hit.

### 4. Three Levels of AI & "Agent or Not?"
- **Traditional Software** → Fixed rules written by a programmer (e.g., GST / bill calculator).
- **Generative AI** → Creates text/content in one LLM call (e.g., drafting a Diwali offer or translating a menu).
- **Agentic AI** → Plans and acts with tools when steps cannot be known in advance.
- **Rule of thumb**: *Known steps → plain code. One-off text → one LLM call. Unknown steps + tools → agent.*

---

## 🛠️ LangChain Basics & Today's Build

### LCEL Chain Pattern
```python
chain = prompt | llm | StrOutputParser()
answer = chain.invoke({"history": history, "question": question})
```
- **Model**: `ChatGroq(model="openai/gpt-oss-20b", temperature=0.3)`
- **Template**: `ChatPromptTemplate.from_messages([("system", SYSTEM), MessagesPlaceholder("history"), ("human", "{question}")])`
- **Memory**: `history.extend([("human", question), ("ai", answer)])`
- **Agent Sneak Peek**: `create_agent(model=llm, tools=[], system_prompt=SYSTEM)` (ready for real tools on Day 4).

> **Common Trap**: Because `{ }` marks template variables in `ChatPromptTemplate`, write literal curly braces as `{{ }}` or use plain text lines instead of raw JSON in system prompts.

### Automated Testing (`tests.csv`)
Store real customer questions with an expected `must_include` keyword/phrase and re-run all tests after every prompt change to track the **pass rate**:

```csv
question,must_include
How much is a plum cake?,500
Do you make eggless cakes?,eggless
Can I get a cake today evening?,24 hours
Do you deliver to Kakinada?,check with the owner
What is the price of 6 cupcakes?,480
```

---

## 📝 Your 30% Practice Tasks

1. **Your business**: Rewrite `SYSTEM` for a business of your choice.
2. **Your tests**: Create `tests.csv` with 10 customer questions and `must_include` checks; record your pass rate.
3. **Improve**: Edit one line in the prompt, re-run tests, and check if the pass rate improves.
4. **Temperature test**: Change `temperature=1.0`, run tests twice, and compare consistency.
5. **Agent or not?**: List 3 jobs in your business and classify each as *plain code*, *one LLM call*, or *agent*.
6. **Before Day 4**: Write two Python functions with type hints and docstrings (e.g., price lookup and availability check) to turn into agent tools tomorrow.

---

## 📊 Summary Table

| Idea | One Line | Analogy |
|---|---|---|
| **Chatbot vs agent** | Answers vs acts to reach a goal | Receptionist vs personal assistant |
| **Four parts** | Brain, instructions, tools, memory | A new shop assistant |
| **Agent loop** | Think → act → observe → repeat | Cooking and tasting |
| **Three levels** | Fixed rules → creates → creates and acts | Calculator, writer, intern |
| **Agent or not** | Use the simplest option that works | Don't hire a manager to switch on a fan |
| **LangChain** | One interface for many models + building blocks | Universal travel adapter |
| **Template & chain** | Blanks in `{ }`; `prompt \| llm \| parser` | Form on a conveyor belt |
| **Tests** | Real questions + `must_include`; re-run every change | Practice test with answer key |
