# 🧠 Day 5: Memory & RAG — An Agent That Remembers and Looks Things Up

**Level 2 · Single Agent · Session 3 of 3**  
**Case Study:** Sweet Crumbs Home Bakery

Today completes the **Sweet Crumbs single agent**. It learns two essential skills:
1. **Conversation Memory**: Remembering each customer's chat separately using threads (`InMemorySaver` + `thread_id`).
2. **RAG (Retrieval-Augmented Generation)**: Searching the bakery's policies in a vector database (`ChromaDB`) instead of stuffing everything into the prompt.

---

## 🎯 Today at a Glance

| # | Topic | Focus |
|---|---|---|
| 1 | **Three Kinds of Memory** | Conversation (chat), Customer (long-term), Knowledge (documents) |
| 2 | **Conversation Memory: Threads** | `InMemorySaver` checkpointer + `thread_id` per customer |
| 3 | **Why Not Put It All in the Prompt?** | Token cost, speed, and keeping the model focused |
| 4 | **Embeddings** | Turning text into numbers that capture meaning |
| 5 | **Vector Database: ChromaDB** | Storing texts and searching by meaning |
| 6 | **Chunking** | Splitting long documents into single-idea pieces with overlap |
| 7 | **RAG: Retrieve, Then Answer** | The `search_faq` tool (`Retrieve → Augment → Generate`) |
| 8 | **Single-Agent Sign-Off** | Complete agent (Days 3–5) ready for testing |

---

## 🧠 Core Concepts (Simple Guide)

### 1. Three Kinds of Memory
| Kind | What It Holds | How Long | Bakery Example |
|---|---|---|---|
| **Conversation (short-term)** | Messages in this chat | One conversation | `"Ravi asked for 2 plum cakes a minute ago"` |
| **Customer (long-term)** | Facts saved across chats | Weeks or months | `"Ravi always orders eggless"` |
| **Knowledge** | Business policies & documents | Until documents change | `"Delivery is free above Rs 1,000"` |

### 2. Conversation Memory: `checkpointer` & `thread_id`
Instead of manually passing a `history` list, give the agent a checkpointer and pass a unique `thread_id` per customer:
```python
from langgraph.checkpoint.memory import InMemorySaver

agent = create_agent(
    model=llm,
    tools=[get_price, is_open, log_order, search_faq],
    system_prompt=SYSTEM,
    checkpointer=InMemorySaver(),
)

config = {"configurable": {"thread_id": "ravi"}}
result = agent.invoke({"messages": [{"role": "user", "content": text}]}, config)
```
- Different `thread_id` values keep customers' chats completely separate.
- `InMemorySaver` stores threads in RAM (cleared on runtime restart).

### 3. Why Not Put Everything in the Prompt?
- With ~10 short facts, putting them in the prompt is fine.
- With large policy documents, sending everything on every call wastes tokens, slows responses, and buries the relevant fact.
- **Better approach**: Search first, and send only the 2–3 relevant facts.

### 4. Embeddings
An **embedding** converts text into a list of numbers representing its **meaning**. Sentences with similar meaning (e.g., *"Do you deliver?"* and *"Can you send it to my house?"*) sit close together even when they use different words.

### 5. Vector Database: ChromaDB
ChromaDB stores texts alongside their embeddings and finds the closest matches by meaning:
```python
import chromadb

client = chromadb.Client()
faq = client.get_or_create_collection(name="sweet_crumbs_faq")
faq.upsert(ids=list(FAQS.keys()), documents=list(FAQS.values()))

results = faq.query(query_texts=["Can you send the cake to my house?"], n_results=2)
```

### 6. Chunking
Long documents are split into small **chunks** (one idea / a few sentences each) with a small **overlap** (~15%) so sentences aren't cut in half at boundaries. For FAQs, one FAQ = one chunk.

### 7. RAG: Retrieve → Augment → Generate (`search_faq`)
1. **Retrieve**: Search ChromaDB with the customer's question.
2. **Augment**: Pass the top results back to the model.
3. **Generate**: Answer **only** from those facts—or say `"I'll check with the owner."` if not found.

```python
@tool
def search_faq(question: str) -> str:
    """Search Sweet Crumbs' policies and FAQs: delivery, payment, cancellation,
    photo cakes, allergens, storage, candles and messages on cakes.
    Use for any question about rules or policies, not for prices or opening hours."""
    results = faq.query(query_texts=[question], n_results=3)
    return "\n".join(results["documents"][0])
```

### 8. Single-Agent Sign-Off
Your complete single agent now combines all pieces from Days 3–5:
- **Brain**: `ChatGroq` (low temperature)
- **Instructions**: `SYSTEM` prompt (`Role · Facts · Rules · Format`)
- **Tools**: `get_price`, `is_open`, `log_order`, `search_faq`
- **Memory**: `InMemorySaver` + `thread_id`
- **Knowledge**: `ChromaDB` FAQs (RAG)
- **Checks**: Pydantic validation + `tests.csv`

---

## 📝 Your 30% Practice Tasks

1. **Your knowledge**: Load your 15–20 business facts into a ChromaDB collection (one fact per chunk).
2. **Your search tool**: Write a `search_...` `@tool` with a clear docstring listing the topics it covers.
3. **Your agent**: Add it to your Day 4 agent with `InMemorySaver()` and test two separate `thread_id`s.
4. **Break it**: Ask something not in your documents—make sure it says it will check instead of making up an answer.
5. **Sign-off**: Add new policy questions to `tests.csv`, run all tests, and aim for at least **80% pass rate**.
6. **Before Day 6**: Draw on paper the step-by-step flow your business follows for a customer request (to turn into a LangGraph workflow tomorrow).

---

## 📊 Summary Table

| Idea | One Line | Analogy |
|---|---|---|
| **Three kinds of memory** | Conversation, customer, knowledge | Notepad, register, policy file |
| **Threads** | Checkpointer + one `thread_id` per conversation | Email threads |
| **Not everything in prompt** | Search first, send only what's relevant | An index, not the whole library |
| **Embeddings** | Text → numbers; similar meaning, nearby numbers | Supermarket shelves |
| **Vector database** | Store texts, find closest by meaning | A librarian who knows every book |
| **Chunking** | One idea per chunk, small overlap | Neat cake slices |
| **RAG** | Retrieve → augment → generate; "not found" is allowed | Open-book exam, no bluffing |
| **Sign-off** | Brain, instructions, tools, memory, knowledge, tests | Employee after probation |
