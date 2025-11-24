# Lesson 3 – Sequential Pattern: Bug Report Cleaning & Triage Pipeline

This README explains the notebook:

`03_sequential_bug_report_triage.ipynb`

It is the third lesson in the **LangGraph Agentic AI – Intro Course** and introduces the **sequential pattern**, where multiple nodes are chained together to process data step by step.

---

## 1. Learning Goals

After working through this notebook, you should be able to:

- Design a `TypedDict` state for a multi-step processing pipeline.
- Implement several node functions, each responsible for one clear transformation.
- Wire nodes together in a **linear LangGraph pipeline** (A → B → C).
- Inspect state at different points in the pipeline to understand intermediate results.
- Extend the pipeline with extra logic and extra nodes.

---

## 2. Conceptual Overview

### 2.1 Why sequential pipelines?

Many real workflows are naturally **stepwise**:

1. Clean or normalize incoming data.
2. Extract structured information.
3. Make decisions or recommendations.

The sequential pattern in LangGraph captures these scenarios: each node adds or refines information in the state, and passes it on to the next.

### 2.2 Use case: Bug Report Cleaning & Triage

In this lesson, we simulate a simple bug triage system. Given a raw bug report like:

> "App keeps freezing when I click Export with CSV on Mac. Urgent!!!"

the agent should:

1. Normalize the text (`clean_report`).
2. Infer:
   - `severity` (low/medium/high),
   - `platform` (web/mac/windows/unknown),
   - `feature` (export/login/unknown).
3. Produce a `recommendation` string describing how to handle the bug.

This mirrors how an internal tool might assist support engineers or developers triaging issues.

---

## 3. Notebook Structure

### 3.1 Intro & Objectives

- States the goals of the lesson.
- Connects Lesson 3 to the previous ones (we move from single-node agents to multi-node pipelines).

### 3.2 Environment Setup

- Reminds you to:
  - Create/activate a virtual environment.
  - Install dependencies from `env/requirements.txt`.
  - Use the correct Jupyter kernel.

### 3.3 Concept Warm-Up

- Explains why multi-step flows are common in real systems.
- Presents a concrete bug report example.
- Provides a scratch cell for simple string processing tests.

### 3.4 State Definition

Defines the base `BugState`:

```python
class BugState(TypedDict):
    raw_report: str
    clean_report: str
    severity: str
    platform: str
    feature: str
    recommendation: str
```

- `raw_report` – original text.
- `clean_report` – normalized version used for detection.
- `severity`, `platform`, `feature` – extracted metadata.
- `recommendation` – summary decision.

### 3.5 Node 1 – Cleaning the Report

`clean_report_node`:

- Reads `raw_report`.
- Collapses multiple spaces into one.
- Strips leading/trailing whitespace.
- Writes `clean_report`.

This sets up a clean input for subsequent nodes.

### 3.6 Node 2 – Metadata Extraction

`extract_metadata_node`:

- Reads `clean_report` (lowercased).
- Determines `severity` based on keywords:
  - e.g. "urgent", "crash", "freez" → high.
- Determines `platform` based on "mac", "windows", "web", "browser".
- Determines `feature` based on "export" or "login".
- Writes these values into the state.

This step illustrates how to turn unstructured text into simple structured fields.

### 3.7 Node 3 – Recommendation Generation

`make_recommendation_node`:

- Reads `severity`, `platform`, `feature`.
- Builds a human-readable `recommendation` string such as:

  > "High severity bug on mac in export. Assign to on-call engineer immediately."

- Uses basic rules for high vs non-high severity.

This demonstrates a decision layer built on top of extracted metadata.

### 3.8 Building the Sequential LangGraph

The notebook then:

- Imports `StateGraph`.
- Adds the three nodes.
- Sets:
  - Entry point: `clean_report`.
  - Edges: `clean_report` → `extract_metadata` → `make_recommendation`.
  - Finish point: `make_recommendation`.
- Compiles the graph and invokes it with an example `BugState`.
- Encourages you to tweak `raw_report` to see how outputs change.

### 3.9 Exercises

The notebook includes several exercises with `TODO` cells.

#### Exercise 1 – More Severity Rules

Extend severity logic to:

- Set `severity = "high"` if `"data loss"` appears in the text.
- Set `severity = "low"` when `"typo"` appears and no high-severity keywords are present.
- Keep existing behavior otherwise.

#### Exercise 2 – Unknown Handling

Modify `make_recommendation_node` so that if `platform == "unknown"` or `feature == "unknown"`, it appends:

> " Need more info on platform/feature."

to the `recommendation` string.

#### Stretch Exercise 1 – Third-party Flag

Add `is_third_party: bool` to the state and set it to `True` when the text mentions "plugin" or "extension". Then update the recommendation to append:

> " Route to integrations team."

when `is_third_party` is `True`.

#### Stretch Exercise 2 – Assign Team Node

Add a new node `assign_team_node` that:

- Reads `feature`.
- Sets `assigned_team: str` depending on the feature (e.g., "Data Export Team", "Auth Team", "General Backend Team").
- Is wired **after** `make_recommendation_node` in the pipeline.
- Becomes the new finish node.

---

## 4. How to Run the Notebook

From the repo root (`langgraph-agentic-intro/`):

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

   `notebooks/03_sequential_bug_report_triage.ipynb`

5. **Select the correct kernel:**

   Use the virtual environment you just created (e.g. `langgraph-intro`).

6. **Run the cells in order:**

   - Read the markdown explanations.
   - Execute the code cells.
   - Implement and test your solutions in the exercise sections.

---

## 5. How This Lesson Fits in the Course

This lesson extends the ideas from Lessons 1–2:

- **Lesson 1** – Single-node agents with simple state.
- **Lesson 2** – Single-node agents with richer, multiple inputs.
- **Lesson 3** – Multi-node, **sequential pipelines** built on that foundation.

Upcoming lessons:

- **Lesson 4** – Conditional routing (different paths based on state).
- **Lesson 5** – Looping (iterative refinement).

By completing this lesson, you’ll understand how to compose multiple nodes into a meaningful pipeline, which is a key building block for more advanced agentic architectures.

---

## 6. Suggested Next Steps

After finishing Lesson 3:

1. Compare your implementation with the solutions notebook.
2. Think about how to adapt this pipeline to other domains (e.g., support ticket triage, log processing, email preprocessing).
3. Move on to **Lesson 4 – Conditional Pattern: Customer Support Router Agent**, where you’ll learn how to route different inputs to different nodes based on the state.
