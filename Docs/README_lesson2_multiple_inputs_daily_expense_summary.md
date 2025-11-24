# Lesson 2 – Multiple Inputs Pattern: Daily Expense Summary Agent

This README explains the notebook:

`02_multiple_inputs_daily_expense_summary.ipynb`

It is the second lesson in the **LangGraph Agentic AI – Intro Course** and introduces the **multiple inputs pattern** using a simple, practical use case: summarizing daily expenses.

---

## 1. Learning Goals

After working through this notebook, you should be able to:

- Design a `TypedDict` state that carries **multiple input fields** and **derived fields**.
- Implement a **single-node LangGraph app** that:
  - Aggregates a list of values (expenses),
  - Computes derived metrics (total, count, remaining budget),
  - Produces a human-readable summary string.
- Extend the state and node behavior to support:
  - Multiple currencies,
  - Per-expense categories,
  - Simple alert flags.

---

## 2. Conceptual Overview

### 2.1 Why multiple inputs?

In real-world agentic systems, you rarely work with just one scalar input. Instead, agents often need to:

- Aggregate lists (transactions, events, messages, etc.).
- Combine configuration or context values (currency, thresholds, budgets).
- Produce richer outputs that summarize this information.

This lesson turns a **daily expense tracker** into a LangGraph agent, so you can see how multiple inputs flow through a single node.

### 2.2 Use case: Daily Expense Summary Agent

The notebook models a tiny “analytics assistant” for personal finance:

- **Inputs**:
  - `expenses: List[float]` – amounts you spent today.
  - `currency: str` – e.g. `"EUR"`.
  - `daily_budget: float` – how much you planned to spend today.

- **Derived**:
  - `total_spent: float` – sum of all expenses.
  - `num_transactions: int` – number of recorded expenses.
  - `summary: str` – plain-language description of how you did vs your budget.

Example summary:

> "You spent 37.50 EUR today across 5 transactions. You are 12.50 EUR under budget."

The logic is deliberately simple and rule-based, so you can focus on the **pattern** rather than complex modeling.

---

## 3. Notebook Structure

The notebook is structured in a similar way to Lesson 1, but with richer state.

### 3.1 Intro & Objectives

- States the learning goals for the lesson.
- Connects Lesson 2 to Lesson 1 (single-node → multi-input single-node).

### 3.2 Environment Setup

- Reminds you to create/activate a virtual environment.
- Install dependencies from `env/requirements.txt`.
- Ensure you’re using the correct Jupyter kernel.

### 3.3 Concept Warm-Up

- Recaps that Lesson 1 worked with a small state.
- Explains why multiple inputs are common and useful (e.g., expense logs).
- Provides a scratch cell for simple Python experiments.

### 3.4 State Definition

Defines the base `ExpenseState`:

```python
class ExpenseState(TypedDict):
    expenses: List[float]
    currency: str
    daily_budget: float
    total_spent: float
    num_transactions: int
    summary: str
```

- Inputs: `expenses`, `currency`, `daily_budget`.
- Outputs: `total_spent`, `num_transactions`, `summary`.

This emphasizes **state as a single source of truth** for what the agent knows and produces.

### 3.5 Node Implementation

Defines the `summarize_expenses` function:

```python
def summarize_expenses(state: ExpenseState) -> ExpenseState:
    expenses = state["expenses"]
    total = sum(expenses)
    num = len(expenses)
    budget = state["daily_budget"]
    currency = state["currency"]

    remaining = budget - total
    if budget > 0:
        if remaining > 0:
            status = f"You are {remaining:.2f} {currency} under budget."
        elif remaining == 0:
            status = "You hit your budget exactly today."
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

Key ideas:

- Multiple input fields feed into the computation.
- Derived fields are written back into the state.
- The function can be tested in isolation, without involving LangGraph.

### 3.6 Building the LangGraph App

- Imports and uses `StateGraph`:

  ```python
  from langgraph.graph import StateGraph
  ```

- Standard pattern:

  1. `graph = StateGraph(ExpenseState)`
  2. `graph.add_node("summarize_expenses", summarize_expenses)`
  3. `graph.set_entry_point("summarize_expenses")`
  4. `graph.set_finish_point("summarize_expenses")`
  5. `app = graph.compile()`
  6. `result = app.invoke(initial_state)`

- Demonstrates how the same single-node pattern scales to richer state.

### 3.7 Playing with Inputs

You are encouraged to vary:

- The expense list (many small vs few large amounts).
- The daily budget (0, small, large).
- The currency string.

This helps build intuition about how **input fields** influence the logic and summary.

### 3.8 Exercises

The end of the notebook contains exercises with `TODO` cells.

#### Exercise 1 – Zero-expense Day

Handle the case where `expenses` is empty:

- `total_spent = 0`
- `num_transactions = 0`
- `summary = "No expenses recorded today."`

This ensures the agent behaves sensibly when there is no data.

#### Exercise 2 – Multiple Currencies

Extend the state and logic to handle:

- `target_currency: str`
- `converted_total: float`

Use a simple fake conversion rate (e.g. multiply by 1.1) and mention both original and converted totals in the summary.

#### Stretch Exercise 1 – Category Breakdown

Introduce a richer structure:

```python
class Expense(TypedDict):
    amount: float
    category: str
```

Then:

- Change `expenses` to `List[Expense]`.
- Compute totals per category.
- Mention the top spending category in the summary.

#### Stretch Exercise 2 – Alert Flag

Add an `alert: bool` field:
- `alert = True` if total spent is more than 120% of the daily budget.
- Optionally append a warning to the summary when `alert` is `True`.

---

## 4. How to Run the Notebook

From the repo root (`langgraph-agentic-intro/`):

1. **Create and activate a virtual environment (if needed):**

   ```bash
   python -m venv .venv
   source .venv/bin/activate        # on Windows: .venv\Scripts\activate
   ```

2. **Install dependencies:**

   ```bash
   pip install -r env/requirements.txt
   ```

3. **Start Jupyter:**

   ```bash
   jupyter notebook
   ```

4. **Open the notebook:**

   Navigate to:

   `notebooks/02_multiple_inputs_daily_expense_summary.ipynb`

5. **Select the correct kernel:**

   Choose the environment you just created (e.g. `langgraph-intro`).

6. **Run the cells in order:**

   - Read the markdown explanations.
   - Execute the code cells.
   - Implement and test your solutions in the exercise cells.

---

## 5. How This Lesson Fits in the Course

This lesson builds directly on Lesson 1:

- Lesson 1: single-node graph with a simple state.
- Lesson 2: single-node graph with **richer, multi-field state**.

Upcoming lessons:

- **Lesson 3** – Sequential pipelines (multiple nodes in series).
- **Lesson 4** – Conditional routing (different paths based on state).
- **Lesson 5** – Looping graphs (iterative refinement).

By the time you finish this notebook (including exercises), you will be comfortable designing and working with non-trivial state and deriving meaningful summaries from it.

---

## 6. Suggested Next Steps

After completing Lesson 2:

1. Compare your implementation with the solutions notebook (if desired).
2. Think of other situations where a single-node, multi-input agent is useful (e.g., simple analytics dashboards, KPI summaries).
3. Move on to **Lesson 3 – Sequential Pattern: Bug Report Cleaning & Triage Pipeline**, where you will chain multiple nodes to form a processing pipeline.
