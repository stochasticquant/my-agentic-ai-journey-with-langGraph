# Lesson 5 – Looping Pattern: Social Post Draft Refinement Agent

This README explains the notebook:

`05_looping_social_post_refinement.ipynb`

It is the fifth lesson in the **LangGraph Agentic AI – Intro Course** and introduces the **looping pattern**, where a graph repeatedly evaluates and improves a draft until a stop condition is met.

---

## 1. Learning Goals

After working through this notebook, you should be able to:

- Design a state that tracks a **draft**, **quality score**, and **iteration counters**.
- Implement an **evaluation node** that scores the draft based on simple heuristics.
- Implement an **improvement node** that updates the draft and increments the iteration.
- Use conditional edges in a LangGraph to **loop** until a stop condition is satisfied.
- Extend the loop with additional fields like `stop_reason`, `feedback`, and `platform`.

---

## 2. Conceptual Overview

### 2.1 Why looping?

Many agentic workflows are inherently iterative:

- Refine a draft until it looks good.
- Optimize a plan until constraints are satisfied.
- Re-run a tool until an external signal indicates success.

The looping pattern lets your graph revisit the same nodes multiple times while updating state on each iteration.

### 2.2 Use case: Social Post Draft Refinement

In this lesson, you build a small agent that:

1. Starts from a short social media post (e.g., a LinkedIn or Twitter post).
2. Repeatedly:
   - **Evaluates** the draft.
   - **Improves** it using simple, rule-based heuristics.
3. Stops when:
   - The quality has reached a certain threshold, or
   - The maximum number of iterations has been reached.

Example initial draft:

> "Learning AI. Any tips?"

The agent might iteratively turn it into something like:

> "I'm learning AI and applying it to small projects at work. I'm exploring this topic in depth and will share what I learn. Let me know your thoughts in the comments."

---

## 3. Notebook Structure

### 3.1 Intro & Objectives

- Explains how looping builds on the ideas from the previous four patterns.
- Sets the goal of implementing a simple but realistic refinement loop.

### 3.2 Environment Setup

- Reminds you to:
  - Create and activate a virtual environment.
  - Install dependencies from `env/requirements.txt`.
  - Use the correct Jupyter kernel.

### 3.3 Concept Warm-Up

- Introduces the idea of "evaluate → improve → evaluate → …".
- Provides a scratch cell for quick experiments.

This section frames the mental model of a **feedback loop** on top of a graph.

### 3.4 State Definition

Defines the base `PostState` (in the teaching notebook):

```python
class PostState(TypedDict):
    draft: str
    quality_score: int
    iteration: int
    max_iterations: int
    history: List[str]
```

- `draft` – current version of the post.
- `quality_score` – numeric score (0–100).
- `iteration` – which iteration we are on.
- `max_iterations` – hard cap.
- `history` – list of previous drafts for introspection.

In the solutions notebook, `stop_reason`, `feedback`, and `platform` are added.

### 3.5 Evaluation Node – `evaluate_post_node`

- Computes a score starting from 50.
- Applies penalties/bonuses:
  - Short length penalty.
  - Bonus for calls to action ("comment", "let me know").
  - Bonus for mentioning learning/sharing.
- Clamps the score to [0, 100].

In the exercises/solutions, you extend this with:

- Bonus for including a question mark.
- Bonus for time horizon phrases like "today", "this week", "this year", "over the next".

### 3.6 Improvement Node – `improve_post_node`

- Saves the current `draft` in `history`.
- Adds more context if the draft is short.
- Adds a call to action if missing.
- Increments the `iteration` counter.

In the stretch exercises/solutions, it also:

- Adapts behavior based on `feedback` (e.g., "shorter" or "more examples").
- Adapts style based on `platform` (e.g., shorter for Twitter, more detailed for LinkedIn).

### 3.7 Loop Logic & Graph Wiring

The looping behavior is implemented by:

1. Creating a `StateGraph(PostState)`.
2. Adding the `evaluate_post_node` and `improve_post_node`.
3. Setting `evaluate_post_node` as the entry point.
4. Adding conditional edges from `evaluate_post_node` using a helper `should_continue` function.
5. Adding an edge from `improve_post_node` back to `evaluate_post_node`.

The conditional helper returns either `"continue"` or `"stop"` depending on:

- `quality_score` vs threshold (e.g., 70).
- `iteration` vs `max_iterations`.

Mapping:

- `"continue"` → `improve_post_node`
- `"stop"` → `END`

In the solution, the helper also sets `stop_reason` in the state.

### 3.8 Exercises

The notebook provides exercises with `TODO` cells.

#### Exercise 1 – Smarter Scoring

- Extend `evaluate_post_node` to:
  - Add bonus points if the draft contains a question mark.
  - Add bonus points if it mentions a time horizon.
- Keep scores clamped between 0 and 100.

#### Exercise 2 – Stop Reason

- Add `stop_reason: str` to the state.
- Set it to `"good_quality"` when quality-based stopping happens.
- Set it to `"max_iterations_reached"` when the iteration limit is reached.

#### Stretch Exercise 1 – User Feedback Loop

- Add `feedback: str` to the state.
- Change `improve_post_node` behavior based on feedback:
  - `"shorter"` → avoid excessive length.
  - `"more examples"` → append example sentences.

This simulates a user-in-the-loop refinement process.

#### Stretch Exercise 2 – Multi-platform Style

- Add `platform: str` to the state (`"linkedin"`, `"twitter"`, …).
- Adjust `improve_post_node` so that:
  - Twitter posts stay more compact.
  - LinkedIn posts can be longer and more formal.

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

   `notebooks/05_looping_social_post_refinement.ipynb`

5. **Select the correct kernel:**

   Use the environment you created (e.g. `langgraph-intro`).

6. **Run cells top-to-bottom:**

   - Read the markdown explanations.
   - Execute and inspect the code cells.
   - Implement your own versions of the exercises before checking the solutions notebook.

---

## 5. How This Lesson Fits in the Course

By now, you have covered the five core patterns:

1. **Single Node** – minimal agent (Lesson 1).
2. **Multiple Inputs** – richer state (Lesson 2).
3. **Sequential** – multi-step pipelines (Lesson 3).
4. **Conditional** – router-style branching (Lesson 4).
5. **Looping** – iterative refinement (Lesson 5).

These patterns can be combined to design more complex, real-world LangGraph agents.

---

## 6. Suggested Next Steps

After finishing Lesson 5:

1. Compare your loop implementation with the solutions notebook.
2. Think about other looping scenarios (e.g., planning with retries, tool calls with backoff, multi-step reasoning loops).
3. Start designing a **capstone use case** that combines multiple patterns – for example, a document processing agent or a customer support workflow using routing + sequential + looping behavior.
