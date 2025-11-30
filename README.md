# Algorithm Mentor – Multi-Agent AI Tutor for Algorithms

**Algorithm Mentor** is a multi-agent AI tutor for algorithms and data structures, built for stressed undergrads, returning learners, and ESL students.

It runs as a single **Kaggle notebook** and uses Google’s **Gemini 2.5 Flash Lite** and **Agent Development Kit (ADK)** to:

- Explain concepts with structured lessons and synthetic examples  
- Generate and auto-grade practice problems  
- Visualize algorithms step-by-step  
- Diagnose student needs and orchestrate specialist agents  
- Evaluate itself with a Judge Agent and basic metrics  

> This repository accompanies my submission to the **Kaggle Agents Intensive – Agents for Good** track (education / tutoring).

---

## 1. Problem & Motivation — why algorithms tutoring is hard

Intro algorithms and data structures are **gatekeeper courses** for CS. Many students (especially stressed undergrads, returning learners, and ESL students) experience them as a blur of proofs, pseudocode, and asymptotic notation. They can memorize patterns, but struggle to build real intuition or transfer knowledge to new problems.

Traditional resources (slides, textbooks, static problem sets) are **one-size-fits-all**. Generic chatbots can answer isolated questions, but they don’t:

- remember what a student already tried,  
- intentionally switch between *explain → visualize → practice*, or  
- track mastery over time.  

The problem: we lack a **structured, personalized AI tutor for algorithms** that behaves like a patient TA with a plan, rather than a one-shot “answer generator.”  
Algorithm Mentor is my attempt to fill that gap.

---

## 2. Why Agentic AI? — why agents fit this problem

This tutoring scenario is naturally **multi-step and multi-role**:

- Sometimes the student needs a **deep explanation** and intuition.  
- Sometimes they need **practice problems + feedback**.  
- Sometimes a **visual step-by-step** is what finally makes it click.  
- And something has to **decide what to do next** based on their state.

Instead of one huge prompt, Algorithm Mentor uses **specialized agents**, each with a focused system prompt and responsibility:

- A **Diagnostic + Personalization Agent** reads the session state + student message and plans the next actions (mode, topic, difficulty, and which specialists to call).  
- A **Concept Explainer Agent** focuses purely on clear, structured teaching.  
- A **ProblemGen + Auto-Grader Agent** focuses on high-quality synthetic questions and rubrics.  
- A **Visualization Agent** focuses on stepwise Markdown visuals.  
- A **Judge Agent** focuses on evaluation and scoring.

This cleanly demonstrates core Agentic AI ideas: **multi-agent orchestration, sessions & memory, tooling, observability, and agent-based evaluation**.

---

## 3. What I Built — Algorithm Mentor at a glance

Algorithm Mentor is a **multi-agent tutoring system** implemented inside a single Kaggle notebook, powered by **Gemini 2.5 Flash Lite** and **Google’s Agent Development Kit (ADK)**.

**Top: Student & Kaggle Notebook UI**  
- The notebook is the UI: students run demo cells and type natural-language questions.

**UX Helpers (agent call layer)**  
- Helper functions hide ADK plumbing and connect UI ↔ agents:  
  - `run_diagnostic_turn(...)`  
  - `call_concept_explainer(...)`  
  - `call_problem_generator(...)`  
  - `call_visualization_agent(...)`

**Core Agents & Runners (ADK)**  
Each agent is an ADK `Agent` with its own system prompt and `InMemoryRunner`:

- **Diagnostic + Personalization (orchestrator)**  
  - Reads a compact JSON view of `SessionState` + the latest student message.  
  - Chooses intent, mode (`tutor` / `practice` / `exam` / `review`), topic, difficulty.  
  - Emits an **OrchestratorTurn JSON plan** with actions like `CALL_CONCEPT_EXPLAINER`, `CALL_PROBLEM_GEN_AUTOGRADER`, `CALL_VISUALIZATION`.  

- **Concept Explainer**  
  - Returns structured Markdown lessons with sections: Overview, Intuition, Why it matters, Trace, Pseudocode, Complexity, Pitfalls, Check-your-understanding.  

- **ProblemGen + Auto-Grader**  
  - Generates **synthetic** practice problems (no real course content) and concise rubrics.  

- **Visualization**  
  - Produces step-by-step Markdown visualizations for arrays, graphs, DP tables, recursion trees, heaps, hashing, etc.  

- **Judge (Evaluation)**  
  - Scores other agents’ outputs using a small JSON rubric (score, pass/fail, notes).

**Session & Memory Layer**  
- Dataclasses: `StudentProfile`, `SessionState`, `MasteryEntry`, `EvalTestCase`, `EvalResult`, `EvalSummary`.  
- **Short-term memory**: `chat_history` + `rolling_summary`, compacted via `compact_history_if_needed(...)`.  
- **Lightweight long-term memory**: `algorithm_mentor_memory.json` storing profile, mastery, and notes across runs (a simple stand-in for a Memory Bank).

**Observability & Evaluation**  
- `METRICS` dict + `print_metrics()` track agent usage and eval runs.  
- `EVAL_TESTS` + `run_eval_suite()` automatically call target agents (e.g., Dijkstra explainer, binary search problems) and send their outputs to the Judge Agent for scoring.

At the bottom, all agents run on **Gemini + ADK Runtime** (`Agent`, `InMemoryRunner`, `tools(google_search)`), so the same design could later be deployed to Agent Engine or A2A.

---

## 4. Demo Walkthrough — what using it looks like

The notebook includes demo cells that show the full tutoring loop: **explain → practice → diagnose → visualize → evaluate.**

**Concept explanation**

Returns a structured lesson in Markdown, with intuition, a small synthetic graph, pseudocode, complexity, and self-quiz questions.

```python
await call_concept_explainer(
    topic="Dijkstra's algorithm",
    level="standard",
    persona_hint="Sara, overloaded CS undergrad, prefers C++-style pseudocode."
)
```
**Practice problems**
```python
await call_problem_generator(
    topic="binary search",
    difficulty="easy",
    num_questions=2,
)
```
Generates two easy, self-contained binary search questions plus brief rubrics.

**Diagnostic + orchestration**
```python
await run_diagnostic_turn(
    "I'm confused about dynamic programming, especially 0/1 knapsack tables."
)
```
- **The Diagnostic Agent:**

    - explains in a few sentences what the student likely needs next, then

    - emits an OrchestratorTurn JSON plan, e.g.:

    - explain 0/1 knapsack at an easy level,

    - visualize the DP table,

    - then create 2 practice questions.

- **Visualization** 

```python
await call_visualization_agent(
    viz_request="Visualize merge sort on [4, 1, 3, 9, 7].",
)
```
Produces a step-by-step Markdown visualization of the recursive splits/merges.


**Evaluation + metrics**
```python
eval_summary = await run_eval_suite()
print_metrics()
```

Runs a small eval set, uses the **Judge Agent** for scoring, and prints how many tests passed plus counts of agent calls.

Together, these show Algorithm Mentor working as a coherent multi-agent tutor:
**diagnose → explain → visualize → practice → evaluate.**
## 5. Repository Structure
```text
.
├── README.md
├── notebooks/
│   └── algorithm_mentor_kaggle_demo.ipynb
├── images/
│   ├── thumbnail_dashboard.jpg
│   └── architecture.jpg
├── requirements.txt
└── LICENSE
```
- **notebooks/** – main Kaggle demo notebook (multi-agent tutor).

- **images/** – thumbnail + architecture diagram used in the writeup.

- **requirements.txt** – minimal dependencies if you want to run locally.

- **LICENSE** – MIT License.
## 6. How to Run
**Option A — Kaggle (recommended)**

 1. Open the Kaggle notebook (link from the competition submission).

2. Add your GOOGLE_API_KEY as a Kaggle Secret.

3. Run all cells from top to bottom.

4. Use the demo cells to:

    - Get concept explanations

    - Generate practice problems

    - Run diagnostic turns

    - Produce visualizations

    - Run the evaluation suite

**Option B — Local (experimental)**

1. Create a virtual environment and install requirements:
```python
pip install -r requirements.txt
```
2. Set your Google API key:
```python
export GOOGLE_API_KEY="your-key-here"
```

3. Open the notebook in Jupyter / VS Code and run all cells.

> Note: The project is optimized for Kaggle (e.g., kaggle_secrets, Kaggle paths). Some small tweaks may be needed locally.

## 7. Key Design Decisions

- **Specialized agents instead of one mega-prompt**
Each responsibility (explain, quiz, visualize, judge, orchestrate) is isolated, easier to test, and reusable.

- **Explicit sessions & context engineering**
`SessionState.to_dict()` exposes a compact JSON view (tail of `chat_history` + `rolling_summary`) to the Diagnostic Agent.
`compact_history_if_needed(...)` keeps recent turns verbatim and compresses older ones.

- **Lightweight long-term memory**
`algorithm_mentor_memory.json` stores profile + mastery across runs, mimicking Memory Bank behavior in a Kaggle-friendly way.

- **Built-in evaluation**
`EvalTestCase` / `EvalResult` dataclasses + a dedicated Judge Agent move from “it feels good” to **quantifiable** behavior.

- **Safety & pedagogy**
All agents are instructed to use **synthetic content** only (no real course slides or exams).
The ProblemGen + Auto-Grader Agent is positioned as a **practice and feedback** tool, not a cheating engine.
The overall tone is that of a **teaching assistant**: scaffold understanding rather than just dumping answers.

## 8. Next Steps — what I’d do with more time

If I extend Algorithm Mentor beyond this capstone, I would:

- **Upgrade memory and personalization**

    - Replace the JSON file with a proper Memory Bank / vector store.

    - Track misconceptions and automatically surface weak topics for spaced review.

- **Adopt long-running, deployed agents**

    - Move from InMemoryRunner to a deployed Agent Engine setup with pause/resume, so the tutor can run full structured study sessions.

- **Broaden evaluation & curriculum**

    - Expand EVAL_TESTS to cover more topics (sorting, graphs, trees, DP, complexity).

    - Log Judge metrics over time to monitor quality drift and improvements.

- **Build a dedicated front-end**

    - Wrap the same agents in a simple web UI so students don’t have to use Kaggle to benefit from the tutor.

- **Add safe code execution**

    - Introduce a tool to run small snippets of student code on test cases, closing the loop between explanation → visualization → implementation practice.

These steps would turn Algorithm Mentor from a Kaggle-based multi-agent demo into a more complete, production-ready AI tutor for algorithms and data structures.

## 9. License

This project is licensed under the MIT License – see the LICENSE file for details.


---
