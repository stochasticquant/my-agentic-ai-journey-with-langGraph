# Lesson 1 – Single Node Pattern: Email Subject Booster Agent

This README explains the notebook:

`01_single_node_email_subject_booster.ipynb`

It is the first lesson in the **LangGraph Agentic AI – Intro Course** and introduces the **single node pattern** using a simple, practical use case: improving email subject lines.

---

## 1. Learning Goals

After working through this notebook, you should be able to:

- Explain the core LangGraph concepts of **state**, **node**, and **graph**.
- Define a simple `TypedDict` state for your agent.
- Implement a **single-node LangGraph app** that:
  - Reads data from the state,
  - Applies transformation logic,
  - Writes updated data back into the state.
- Run the compiled graph and inspect the resulting state.
- Start extending the single-node pattern with small variations (modes, categories, emojis, validation).

---

## 2. Conceptual Overview

### 2.1 Agentic mental model

The notebook uses a very minimal, but important, mental model for agentic AI:

- **State**  
  A Python object that carries all the information your agent knows and updates as it runs.  
  In this lesson, the state tracks:
  - `original_subject`: the raw email subject line.
  - `improved_subject`: the enhanced version produced by the agent.

- **Node**  
  A pure-ish function that:
  - Takes the current state as input,
  - Reads some fields,
  - Writes some fields,
  - Returns the modified state.

  Here, we use one node:
  - `improve_subject(state: SubjectState) -> SubjectState`

- **Graph**  
  A directed graph of nodes connected via edges. The graph describes how data flows through the agent.  
  In this lesson:
  - We create the smallest possible graph:
    - One node.
    - No branching.
    - The same node is both entry and finish.

### 2.2 Use case: Email Subject Booster

The notebook wraps these concepts into a tangible use case:

- Input: a possibly boring or messy subject like:
  - `"meeting"`
  - `" invoice 1234 "`
  - `""` (empty string)

- Output: a slightly nicer subject, for example:
  - `"Update: Meeting"`
  - `"Invoice 1234"` (trimmed and capitalized)
  - `"No subject provided"` (for empty input)

This “booster” logic is intentionally simple and rule-based so that the focus stays on **LangGraph patterns**, not on AI models.

---

## 3. Notebook Structure

The notebook is structured into clear, incremental sections:

### 3.1 Intro & Objectives

- Short description of the lesson.
- Explicit learning goals and prerequisites (basic Python, simple understanding of graphs).

### 3.2 Environment Setup

- Reminds you to:
  - Create and activate a virtual environment.
  - Install dependencies from `env/requirements.txt`.
- Confirms you should run the notebook using the course’s Python kernel (e.g. `langgraph-intro`).

### 3.3 Concept Warm-Up

- Explains the mental model:
  - State → Node → Graph.
- Presents the email subject example so you understand *why* you’re building this agent.
- Includes an optional “scratch” code cell for quick Python experiments.

### 3.4 State Definition

- Introduces `TypedDict` from `typing`.
- Defines the `SubjectState` type:

  ```python
  class SubjectState(TypedDict):
      original_subject: str
      improved_subject: str
  ```

- Shows an example `initial_state` for testing:
  - `{"original_subject": "meeting", "improved_subject": ""}`

This section emphasizes the **shape of the state** and how everything the agent knows should live inside this structure.

### 3.5 Node Implementation

- Defines the `improve_subject` function:

  ```python
  def improve_subject(state: SubjectState) -> SubjectState:
      """Simple subject booster node.

      - Trims whitespace
      - If very short, prepends 'Update: '
      - Ensures the first letter is capitalized
      """
      original = state["original_subject"].strip()

      if not original:
          improved = "No subject provided"
      elif len(original) < 10:
          improved = f"Update: {original.capitalize()}"
      else:
          improved = original[0].upper() + original[1:]

      state["improved_subject"] = improved
      return state
  ```

- Demonstrates how to test the function **without LangGraph** by calling it on a `SubjectState` dictionary.

Key ideas:

- Nodes are ordinary Python functions.
- They operate on well-defined state types.
- They can be tested independently from the graph.

### 3.6 Building the LangGraph App

- Imports the core LangGraph class:

  ```python
  from langgraph.graph import StateGraph
  ```

- Performs the standard sequence:

  1. Create a graph:  
     ```python
     graph = StateGraph(SubjectState)
     ```
  2. Add the node:  
     ```python
     graph.add_node("improve_subject", improve_subject)
     ```
  3. Set the entry and finish points:  
     ```python
     graph.set_entry_point("improve_subject")
     graph.set_finish_point("improve_subject")
     ```
  4. Compile the graph into an app:  
     ```python
     app = graph.compile()
     ```
  5. Invoke the app with an initial state:  
     ```python
     result = app.invoke({"original_subject": "meeting", "improved_subject": ""})
     ```

- Prints/inspects `result` so you can see the updated `improved_subject` field.

This section demonstrates how to turn a plain Python function into a **LangGraph-powered agent**.

### 3.7 Playing with Inputs

- Encourages you to modify `original_subject` and rerun the graph:
  - `"deadline tomorrow"`
  - `" invoice 1234 "`
  - `""`

- Shows how different inputs lead to different outputs based on the internal rules.

This is where you build intuition about how the state and node logic interact.

### 3.8 Exercises

The last part of the notebook contains exercise sections with `TODO` cells. They guide you to extend the basic pattern.

#### Exercise 1 – Shortener vs Booster

- Add a `mode: str` field to `SubjectState` (e.g. `"boost"` or `"shorten"`).
- Extend `improve_subject` to:
  - When `mode == "shorten"` and the subject is longer than 40 chars:
    - Truncate and add `"..."`.
- This explores **conditional behavior controlled by the state**, even with a single node.

#### Exercise 2 – Prefix by Category

- Add `category: str` (e.g. `"project"`, `"invoice"`).
- If `category == "invoice"`, prefix the subject with `"Invoice – "`.
- This encourages you to think about **domain-specific rules** and how they can live inside a single node.

#### Stretch Exercise 1 – Emoji Enhancer

- Add emoji rules such as:
  - `"deadline"` → ⏰
  - `"invoice"` → 💸
- Extend the node to add emojis at the beginning or end of the subject.
- This is a fun way to practice detecting keywords and post-processing text.

#### Stretch Exercise 2 – Validation

- Improve the handling of empty or whitespace-only subjects:
  - If `original_subject` is empty, set `improved_subject = "No subject provided"`.
- Ensures you think about **input validation** and robust behavior, even in a tiny agent.

---

## 4. How to Run the Notebook

From the root of your repo (`langgraph-agentic-intro/`):

1. **Create and activate a virtual environment (if not already done):**

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

   `notebooks/01_single_node_email_subject_booster.ipynb`

5. **Select the correct kernel (if needed):**

   - Choose the virtual environment (e.g. `langgraph-intro`) you just created.

6. **Run each cell from top to bottom:**

   - Read the markdown explanations.
   - Execute the code cells.
   - Modify inputs and logic as instructed in the exercises.

---

## 5. How This Lesson Fits in the Course

This first lesson is deliberately small and focused:

- It establishes the **foundational mental model** (“state + node + graph”) that will be reused throughout the course.
- It demonstrates how even a **single-node graph** can be useful and structured.
- It prepares you for the next lessons, where:
  - Lesson 2 introduces **multiple inputs and richer state**.
  - Lesson 3 adds **sequential multi-node pipelines**.
  - Lesson 4 introduces **conditional routing**.
  - Lesson 5 demonstrates **looping / iterative refinement**.

By the time you complete this notebook (including exercises), you’ll be comfortable with the core mechanics needed to build more complex LangGraph agents.

---

## 6. Suggested Next Steps

After finishing this notebook:

1. Commit your changes (including any exercise solutions) to your Git repo.
2. Reflect on how the single-node pattern could be reused for:
   - Other small text transformations,
   - Data cleaning tasks,
   - Or simple rule-based decisions.
3. Move on to **Lesson 2 – Multiple Inputs Pattern: Daily Expense Summary Agent** and see how the ideas scale to richer state.

Happy hacking with LangGraph 🚀
