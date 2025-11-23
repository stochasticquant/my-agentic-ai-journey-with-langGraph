# LangGraph Agentic AI – Intro Course

This repository contains materials for a beginner-friendly introduction to **agentic AI** using the **LangGraph** framework.

The course is organized into five core lessons, each focusing on a specific **graph pattern** and a small, concrete use case. Together, these lessons prepare learners for a larger, real-world capstone project.

---

## Repository Structure

A suggested structure for this repo:

```text
langgraph-agentic-intro/
├─ README.md
├─ notebooks/
│  ├─ 01_single_node_email_subject_booster.ipynb
│  ├─ 02_multiple_inputs_daily_expense_summary.ipynb
│  ├─ 03_sequential_bug_report_triage.ipynb
│  ├─ 04_conditional_customer_support_router.ipynb
│  └─ 05_looping_social_post_refinement.ipynb
├─ src/
│  ├─ __init__.py
│  ├─ lesson1_single_node/
│  │  └─ subject_booster.py
│  ├─ lesson2_multiple_inputs/
│  │  └─ expense_summary.py
│  ├─ lesson3_sequential/
│  │  └─ bug_triage.py
│  ├─ lesson4_conditional/
│  │  └─ support_router.py
│  └─ lesson5_looping/
│     └─ post_refinement.py
└─ data/
   └─ (optional sample data if needed later)
```

You can adjust paths and filenames as you implement each lesson.

---

## Lesson 1 — Single Node Pattern  
**Use case:** *Email Subject Booster Agent*

### 1. Objective

By the end of this lesson, students can:

- Explain what a **state** and a **node** are in LangGraph.
- Build a **single-node graph** that transforms an email subject line into a slightly improved subject.

### 2. Prerequisites

- Basic Python (functions, dicts, types).
- Very high-level understanding of “graphs” (nodes + edges) is helpful but not required.

### 3. Lesson Flow

**Step 1 – Concept warm-up (5–10 min)**  
- Introduce the core mental model:
  - *State* = a Python object holding information (we’ll start with a TypedDict).
  - *Node* = a function that reads and writes state.
  - *Graph* = wiring nodes together. Today: just 1 node.
- Show a simple before/after example:
  - Input: `"meeting"`  
  - Output: `"Project Update – meeting"`

**Step 2 – Define the state (10 min)**  

```python
class SubjectState(TypedDict):
    original_subject: str
    improved_subject: str
```

- Discuss:
  - Why do we separate `original_subject` and `improved_subject`?
  - (Teaches immutability-ish thinking and clear data flow.)

**Step 3 – Write the node function (15 min)**  

```python
def improve_subject(state: SubjectState) -> SubjectState:
    original = state["original_subject"].strip()
    if len(original) < 10:
        improved = f"Update: {original.capitalize()}"
    else:
        improved = original[0].upper() + original[1:]
    state["improved_subject"] = improved
    return state
```

- Emphasize:
  - Node signature.
  - Read → transform → write → return.

**Step 4 – Build and run the graph (15 min)**  
- Show how to:
  - Create `StateGraph(SubjectState)`
  - Add node: `graph.add_node("improve_subject", improve_subject)`
  - Set entry and finish:
    - `graph.set_entry_point("improve_subject")`
    - `graph.set_finish_point("improve_subject")`
  - Compile: `app = graph.compile()`
  - Invoke with initial state:

    ```python
    result = app.invoke({"original_subject": "meeting", "improved_subject": ""})
    ```

- Inspect result, show how `improved_subject` changed.

**Step 5 – Wrap up with mental model (5 min)**  
- Reiterate:
  - “One state + one node + one edge = a minimal agent.”
  - This pattern will show up inside more complex graphs later.

### 4. Exercises

**Core exercises**

1. **Shortener vs booster**  
   - Add a `mode: str` field (`"boost"` vs `"shorten"`) and modify `improve_subject` to:
     - If `mode == "shorten"` and subject > 40 chars → truncate and add `"..."`.

2. **Prefix by category**  
   - Add `category: str` (e.g. `"project"`, `"invoice"`).
   - If category is known, add prefix:
     - `"invoice"` → `"Invoice – {subject}"`.

**Stretch exercises**

1. **Emoji enhancer**  
   - Add emojis for certain keywords:
     - `"deadline"` → ⏰
     - `"invoice"` → 💸  

2. **Validation**  
   - If `original_subject` is empty, set `improved_subject` to `"No subject provided"`.


---

## Lesson 2 — Multiple Inputs Pattern  
**Use case:** *Daily Expense Summary Agent*

### 1. Objective

Students will:

- Design a state with **multiple input fields**.
- Implement a node that computes derived values (total, count, remaining budget).
- Produce a human-readable summary string from state.

### 2. Prerequisites

- Lesson 1 completed.
- Comfortable with Python lists and simple arithmetic.

### 3. Lesson Flow

**Step 1 – Recap & motivation (5–10 min)**  
- Recap Lesson 1: one input → one output.
- Now: we handle **multiple inputs at once**.
- Story: User logs daily expenses; agent summarizes.

**Step 2 – Design the state (10 min)**  

```python
class ExpenseState(TypedDict):
    expenses: List[float]   # e.g. [12.5, 5.0, 20.0]
    currency: str           # "EUR"
    daily_budget: float     # e.g. 50.0
    total_spent: float
    num_transactions: int
    summary: str
```

- Discuss:
  - Input fields: `expenses`, `currency`, `daily_budget`.
  - Output/derived fields: `total_spent`, `num_transactions`, `summary`.

**Step 3 – Implement the summarizer node (15–20 min)**  

```python
def summarize_expenses(state: ExpenseState) -> ExpenseState:
    expenses = state["expenses"]
    total = sum(expenses)
    num = len(expenses)
    budget = state["daily_budget"]
    currency = state["currency"]

    remaining = budget - total
    if budget > 0:
        if remaining >= 0:
            status = f"You are {remaining:.2f} {currency} under budget."
        else:
            status = f"You are {-remaining:.2f} {currency} over budget."
    else:
        status = "No budget set."

    summary = (
        f"You spent {total:.2f} {currency} today across {num} transactions. {status}"
    )

    state["total_spent"] = total
    state["num_transactions"] = num
    state["summary"] = summary
    return state
```

- Show that the node:
  - Reads multiple fields.
  - Writes multiple derived fields.

**Step 4 – Graph construction (10 min)**  
- Very similar to Lesson 1:
  - `StateGraph(ExpenseState)`
  - One node: `"summarize"`
  - Same node as entry and finish.
- Invoke with example inputs and inspect `summary`.

**Step 5 – Reflection (5 min)**  
- Highlight:
  - Many real agents work off multiple inputs.
  - State is your **single source of truth**.

### 4. Exercises

**Core exercises**

1. **Zero-expense day**  
   - Handle empty `expenses` list:
     - `total_spent = 0`, `num_transactions = 0`
     - `summary = "No expenses recorded today."`

2. **Multiple currencies**  
   - Add `target_currency` and a simple fake conversion function (just multiply by 1.1).
   - Extend `summary` to mention both original and converted totals.

**Stretch exercises**

1. **Category breakdown**  
   - Change state to:

     ```python
     class Expense(TypedDict):
         amount: float
         category: str  # "food", "transport", ...

     class ExpenseState(TypedDict):
         expenses: List[Expense]
         ...
     ```

   - Compute total per category and mention top 1 category in summary.

2. **Alert flag**  
   - Add `alert: bool` to state.
   - Set it to `True` if user is > 20% over budget.

---

## Lesson 3 — Sequential Pattern  
**Use case:** *Bug Report Cleaning & Triage Pipeline*

### 1. Objective

Students will:

- Build a **multi-node sequential** LangGraph.
- See how each node adds one step of processing in a pipeline.
- Understand how to debug by inspecting state between steps.

### 2. Prerequisites

- Lessons 1–2.
- Comfortable with basic string handling and if/else logic.

### 3. Lesson Flow

**Step 1 – Problem framing (5–10 min)**  
- Show raw bug report example:

  > “App keeps freezing when I click Export with CSV on Mac. Urgent!!!”

- Desired outcome:
  - Clean, normalized description.
  - Extracted attributes: severity, platform, feature.
  - Recommendation message.

**Step 2 – State design (10 min)**  

```python
class BugState(TypedDict):
    raw_report: str
    clean_report: str
    severity: str       # "low" | "medium" | "high"
    platform: str       # "web" | "mac" | "windows" | "unknown"
    feature: str        # "export" | "login" | "unknown"
    recommendation: str
```

**Step 3 – Node 1: cleaning (10 min)**  

```python
def clean_report_node(state: BugState) -> BugState:
    raw = state["raw_report"]
    clean = " ".join(raw.split())  # collapse whitespace
    clean = clean.strip()
    state["clean_report"] = clean
    return state
```

**Step 4 – Node 2: metadata extraction (15 min)**  

```python
def extract_metadata_node(state: BugState) -> BugState:
    text = state["clean_report"].lower()
    # severity
    if "urgent" in text or "crash" in text or "freez" in text:
        severity = "high"
    else:
        severity = "medium"

    # platform
    if "mac" in text:
        platform = "mac"
    elif "windows" in text:
        platform = "windows"
    elif "web" in text or "browser" in text:
        platform = "web"
    else:
        platform = "unknown"

    # feature
    if "export" in text:
        feature = "export"
    elif "login" in text:
        feature = "login"
    else:
        feature = "unknown"

    state["severity"] = severity
    state["platform"] = platform
    state["feature"] = feature
    return state
```

**Step 5 – Node 3: recommendation (10–15 min)**  

```python
def make_recommendation_node(state: BugState) -> BugState:
    severity = state["severity"]
    platform = state["platform"]
    feature = state["feature"]

    recommendation = (
        f"{severity.title()} severity bug "
        f"on {platform} in {feature}."
    )

    if severity == "high":
        recommendation += " Assign to on-call engineer immediately."
    else:
        recommendation += " Add to regular sprint backlog."

    state["recommendation"] = recommendation
    return state
```

**Step 6 – Build the sequential graph (10 min)**  
- Nodes wired as:

  `clean_report` → `extract_metadata` → `make_recommendation` → END

- Run with several example bug reports and print state after each node (optional debugging).

### 4. Exercises

**Core exercises**

1. **More severity rules**  
   - Add `"data loss"` as a trigger for `"high"` severity.

2. **Unknown handling**  
   - If `platform == "unknown"` or `feature == "unknown"`, append a note:
     > “Need more info on platform/feature.”

**Stretch exercises**

1. **Third-party flag**  
   - Add `is_third_party: bool` field:
     - If text contains `"plugin"` or `"extension"` → `True`
     - Update recommendation to route to integration team when `True`.

2. **Add a 4th node**  
   - `assign_team_node` that maps `(feature, platform)` to a team name and adds `assigned_team` to state.


---

## Lesson 4 — Conditional Pattern  
**Use case:** *Customer Support Router Agent*

### 1. Objective

Students will:

- Use **conditional edges** to route state to different nodes.
- Implement a simple classifier node + specialized handler nodes.

### 2. Prerequisites

- Lessons 1–3.
- Basic understanding of if/elif/else.

### 3. Lesson Flow

**Step 1 – Scenario intro (5–10 min)**  
- Support system receives messages:
  - “I need a refund for my last invoice.”  
  - “My app shows an error when I click save.”  
  - “I forgot my password.”
- The agent should:
  - Classify topic.
  - Route to the appropriate handler.

**Step 2 – State design (10 min)**  

```python
class SupportState(TypedDict):
    user_message: str
    topic: str        # "billing" | "technical" | "account" | "unknown"
    response: str
```

**Step 3 – Topic classifier node (15 min)**  

```python
def classify_topic_node(state: SupportState) -> SupportState:
    text = state["user_message"].lower()
    if "invoice" in text or "refund" in text or "payment" in text:
        topic = "billing"
    elif "error" in text or "bug" in text or "crash" in text:
        topic = "technical"
    elif "password" in text or "username" in text or "login" in text:
        topic = "account"
    else:
        topic = "unknown"
    state["topic"] = topic
    return state
```

**Step 4 – Handler nodes (15–20 min)**  

- **Billing handler:**

  ```python
  def billing_handler_node(state: SupportState) -> SupportState:
      state["response"] = (
          "Thanks for reaching out about billing. "
          "Please share your invoice number so we can assist."
      )
      return state
  ```

- **Technical handler:**

  ```python
  def technical_handler_node(state: SupportState) -> SupportState:
      state["response"] = (
          "Thanks for reporting a technical issue. "
          "Please send a screenshot and steps to reproduce."
      )
      return state
  ```

- **Account handler:**

  ```python
  def account_handler_node(state: SupportState) -> SupportState:
      state["response"] = (
          "For account issues, please use the password reset link "
          "or confirm your email address so we can help."
      )
      return state
  ```

- **Fallback:**

  ```python
  def fallback_handler_node(state: SupportState) -> SupportState:
      state["response"] = (
          "We're not sure which team should handle this. "
          "A human agent will review your message shortly."
      )
      return state
  ```

**Step 5 – Graph wiring with conditional edges (15–20 min)**  
- Build graph:
  - Add all nodes.
  - Entry: `classify_topic_node`.
  - Conditional edges from `classify_topic_node` based on `topic`:
    - `"billing"` → `billing_handler_node`
    - `"technical"` → `technical_handler_node`
    - `"account"` → `account_handler_node`
    - `"unknown"` → `fallback_handler_node`
  - Each handler then goes to `END`.

- Test with different sample messages.

### 4. Exercises

**Core exercises**

1. **Add a “shipping” topic**  
   - Keywords: `"shipping"`, `"delivery"`, `"tracking"`.
   - Create `shipping_handler_node` and route to it.

2. **Improve classifier priority**  
   - If a message contains both billing and technical keywords, decide priority rules (e.g. billing wins).

**Stretch exercises**

1. **Multi-topic detection**  
   - Extend state with `topics: List[str]` to support multi-label classification.
   - “Primary topic” still used for routing, but response mentions secondary topics.

2. **Language fallback**  
   - If message has obvious non-English words, route to `fallback_handler_node` with a message about language support.

---

## Lesson 5 — Looping Pattern  
**Use case:** *Social Post Draft Refinement Agent*

### 1. Objective

Students will:

- Implement a **looping graph** with a clear stop condition.
- Model iterative “evaluate → improve” cycles in LangGraph.

### 2. Prerequisites

- Lessons 1–4.
- Comfort with while-loop logic and simple scoring rules.

### 3. Lesson Flow

**Step 1 – Real-world analogy (5–10 min)**  
- Show a rough LinkedIn post draft:

  > “Learning AI. Any tips?”

- Desired behavior:
  - Evaluate quality.
  - Suggest & apply improvements.
  - Repeat a few times until “good enough.”

**Step 2 – State design (10 min)**  

```python
class PostState(TypedDict):
    draft: str
    quality_score: int      # 0–100
    iteration: int
    max_iterations: int
    history: List[str]
```

**Step 3 – Evaluation node (15 min)**  

```python
def evaluate_post_node(state: PostState) -> PostState:
    text = state["draft"]
    score = 50  # base

    if len(text) < 50:
        score -= 20
    if "comment" in text.lower() or "let me know" in text.lower():
        score += 20
    if "learning" in text.lower() or "sharing" in text.lower():
        score += 10

    score = max(0, min(100, score))
    state["quality_score"] = score
    return state
```

- Explain: extremely simple scoring, but enough to show the concept.

**Step 4 – Improvement node (15–20 min)**  

```python
def improve_post_node(state: PostState) -> PostState:
    draft = state["draft"]
    history = state["history"]
    history.append(draft)

    new_draft = draft

    if len(draft) < 50:
        new_draft += " I'm exploring this topic in depth and will share what I learn."
    if "comment" not in draft.lower() and "let me know" not in draft.lower():
        new_draft += " Let me know your thoughts in the comments."

    state["draft"] = new_draft
    state["history"] = history
    state["iteration"] += 1
    return state
```

**Step 5 – Loop logic with conditional edges (20 min)**  
- Desired condition:
  - Continue loop if `quality_score < 70` and `iteration < max_iterations`.
  - Stop otherwise.

- Graph idea:
  - Entry → `evaluate_post_node`
  - From `evaluate_post_node`:
    - If stop condition met → END
    - Else → go to `improve_post_node`
  - From `improve_post_node` → back to `evaluate_post_node`

- Run with:
  - `draft = "Learning AI. Any tips?"`
  - `iteration = 0`, `max_iterations = 3`, `history = []`
- Print `draft`, `quality_score`, and `iteration` at each cycle.

### 4. Exercises

**Core exercises**

1. **Smarter scoring**  
   - Add bonuses if:
     - Draft contains a question mark → encourages engagement.
     - Draft mentions time horizon, e.g. “this year”, “over the next months”.

2. **Stop reason**  
   - Add `stop_reason: str` to state:
     - `"good_quality"` or `"max_iterations_reached"`.

**Stretch exercises**

1. **User feedback loop**  
   - Add `feedback: str` to state (e.g. `"shorter"`, `"more examples"`).
   - Change `improve_post_node` behavior depending on `feedback`.

2. **Multi-platform style**  
   - Add `platform: str` (`"linkedin"`, `"twitter"`).
   - `improve_post_node` adjusts style:
     - Shorter for Twitter.
     - Longer & more formal for LinkedIn.
