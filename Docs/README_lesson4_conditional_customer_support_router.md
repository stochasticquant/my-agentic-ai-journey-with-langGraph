# Lesson 4 – Conditional Pattern: Customer Support Router Agent

This README explains the notebook:

`04_conditional_customer_support_router.ipynb`

It is the fourth lesson in the **LangGraph Agentic AI – Intro Course** and introduces the **conditional pattern**, where the graph routes to different nodes based on **state-dependent decisions**.

---

## 1. Learning Goals

After working through this notebook, you should be able to:

- Design a `TypedDict` state for a router-style agent.
- Implement a classifier node that sets a `topic` field based on message content.
- Implement separate handler nodes for each topic.
- Use `add_conditional_edges` (or similar) to route to different nodes depending on `topic`.
- Reason about routing priorities and multi-topic messages.

---

## 2. Conceptual Overview

### 2.1 Why conditional routing?

In many agentic systems, you want to:

- Inspect an input once.
- Decide **which specialized skill/handler** should deal with it.
- Run only the relevant branch of logic.

This is the essence of the **conditional pattern**:

- A **router** node (or collection of nodes) chooses a route.
- Different paths may use different tools, data, or prompts.
- The graph uses state to decide which edge to follow.

### 2.2 Use case: Customer Support Router Agent

In this lesson, you implement a small support router that:

1. Reads a `user_message` (free text).
2. Classifies it into a `topic`:
   - `"billing"`
   - `"technical"`
   - `"account"`
   - `"shipping"` (exercise)
   - `"unknown"`
3. Routes to a handler tailored for that topic.
4. Writes a `response` string into the state.

Example:

- Input:  
  > "I need a refund for my last invoice."

- Classification: `topic = "billing"`.
- Routing: → `billing_handler_node`.
- Response: asks for invoice number to proceed.

This pattern mirrors real helpdesk bots and internal routing systems.

---

## 3. Notebook Structure

### 3.1 Intro & Objectives

- Highlights the step from **sequential** to **branching** flows.
- Lists specific goals for routing and handlers.

### 3.2 Environment Setup

- Reminds you to:
  - Create and activate a virtual environment.
  - Install dependencies with `pip install -r env/requirements.txt`.
  - Use the correct Jupyter kernel.

### 3.3 Concept Warm-Up

- Explains the idea of a router/dispatcher agent.
- Gives several example support messages.
- Provides a scratch cell for experiment.

### 3.4 State Definition

Defines the base `SupportState`:

```python
class SupportState(TypedDict):
    user_message: str
    topic: str
    response: str
```

Later in the solutions, a `topics: List[str]` field is added to track all matched topics.

### 3.5 Classifier Node – `classify_topic_node`

- Reads `user_message` and lowercases it.
- Applies simple keyword-based rules to set `topic`:
  - Billing keywords: `"invoice"`, `"refund"`, `"payment"`.
  - Technical keywords: `"error"`, `"bug"`, `"crash"`.
  - Account keywords: `"password"`, `"username"`, `"login"`.
  - Default: `"unknown"`.
- Writes `topic` back into the state.

This node is responsible for **deciding which branch** of the graph to take.

### 3.6 Handler Nodes

Implements four handlers:

- `billing_handler_node`
- `technical_handler_node`
- `account_handler_node`
- `fallback_handler_node` (for `"unknown"` topics)

Each handler:

- Reads the state (mainly `user_message` or `topic` if needed).
- Sets a `response` string tailored to that topic.
- Returns the state.

In the solutions notebook, a `shipping_handler_node` is added.

### 3.7 Building the Conditional LangGraph

- Creates a `StateGraph(SupportState)`.
- Adds all nodes.
- Sets `classify_topic_node` as the entry point.
- Uses `add_conditional_edges` to route based on `state["topic"]`:
  - `"billing"` → `billing_handler_node`
  - `"technical"` → `technical_handler_node`
  - `"account"` → `account_handler_node`
  - `"unknown"` → `fallback_handler_node`
- Each handler node is connected to `END`.

This demonstrates how **one classifier** can decide among multiple **specialized branches**.

### 3.8 Exercises

The notebook provides several exercises with `TODO` cells.

#### Exercise 1 – Add a “shipping” Topic

- Add a `"shipping"` topic.
- Keywords: `"shipping"`, `"delivery"`, `"tracking"`.
- Implement `shipping_handler_node` and update the routing table.
- Set a response that asks for order ID and shipping details.

#### Exercise 2 – Improve Classifier Priority

Some messages contain multiple kinds of keywords, e.g., both billing and technical.

You:

- Compute flags like `is_billing`, `is_technical`, etc.
- Define a clear priority (e.g., billing > technical > account > shipping).
- Implement logic that always follows this priority when multiple topics match.

#### Stretch Exercise 1 – Multi-topic Detection

- Extend the state with `topics: List[str]`.
- `classify_topic_node` collects **all** matched topics into `topics`, while still setting `topic` as the **primary topic**.
- Optionally, handlers can mention secondary topics in their responses.

#### Stretch Exercise 2 – Language Fallback

- Naively detect foreign-language hints (e.g., "bonjour", "hola").
- If such words appear, set `topic = "unknown"`.
- In `fallback_handler_node`, adjust the response to mention possible language limitations and human review.

---

## 4. How to Run the Notebook

From the root of your repo (`langgraph-agentic-intro/`):

1. **Create and activate a virtual environment (if needed):**

   ```bash
   python -m venv .venv
   source .venv/bin/activate        # on Windows: .venv\\Scripts\\activate
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

   `notebooks/04_conditional_customer_support_router.ipynb`

5. **Select the correct kernel:**

   Choose the env you created (e.g. `langgraph-intro`).

6. **Run the cells in order:**

   - Read the markdown explanations.
   - Execute the code cells.
   - Implement and test your solutions in the exercises.

---

## 5. How This Lesson Fits in the Course

This lesson extends the ideas from Lessons 1–3:

- **Lesson 1** – Single-node agents with simple state.
- **Lesson 2** – Single-node agents with multiple inputs.
- **Lesson 3** – Multi-node, **sequential pipelines**.
- **Lesson 4** – **Conditional routing**, enabling specialization per topic.

In **Lesson 5**, you'll combine these ideas with **looping** to build an iterative refinement agent.

---

## 6. Suggested Next Steps

After finishing Lesson 4:

1. Compare your routing logic and handlers with the solutions notebook.
2. Consider how this pattern would adapt to your own support or internal use cases.
3. Move on to **Lesson 5 – Looping Pattern: Social Post Draft Refinement Agent** to explore iterative improvement in an agentic graph.
