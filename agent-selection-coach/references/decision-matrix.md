# Decision Matrix Reference

Quick-reference frameworks for the Agent Selection Coach. Used during Stages 2 (complexity assessment) and 3 (pattern recommendation). Cite these frameworks conversationally — don't read them verbatim.

---

## 1. Complexity Ladder

Start at Level 0. Only escalate when you've exhausted the current level.

### Level 0: Single LLM Call

**Definition:** One prompt, one response. A single logical step with clear input and clear output. No tool use, no memory, no chaining.

**Choose when:**
- The task can be fully specified in a single prompt
- No external data or tool access is needed
- Output quality is achievable with prompt engineering alone

**Examples:**
- Summarize a meeting transcript into bullet points
- Classify a support ticket by department
- Rewrite a paragraph for a different audience

---

### Level 1: Augmented LLM

**Definition:** Still one logical step, but the LLM is enhanced with tools, retrieval, or memory. The model decides when and how to use its augmentations, but the overall task is a single turn.

**Choose when:**
- The task needs real-time data, external context, or actions beyond text generation
- A single prompt + tool call (or retrieval step) can produce the answer
- You don't need to decompose the task into multiple dependent steps

**Examples:**
- Answer a question using RAG over a knowledge base
- Look up a customer record and draft a response
- Query an API and format the result

---

### Level 2: Workflow

**Definition:** Multiple steps orchestrated in a predefined, repeatable sequence. The steps, their order, and handoff logic are determined at design time — not by the model at runtime. May involve LLM calls, tool calls, or code execution at each step.

**Choose when:**
- The task decomposes into predictable, ordered substeps
- The same sequence works every time for this task category
- You need reliability and determinism more than flexibility
- Steps have clear input/output contracts between them

**Examples:**
- Extract data from a document, validate it against a schema, then write it to a database
- Generate a draft, run it through a style checker, then apply edits
- Translate content, localize formatting, then publish to multiple channels

---

### Level 3: Agent

**Definition:** The model dynamically decides what to do next based on observations and environment feedback. Steps cannot be predicted or hardcoded in advance. The agent loops: observe, decide, act, observe again.

**Choose when:**
- The task is open-ended and the path to completion depends on what's discovered along the way
- The number of steps is unknown at design time
- The agent needs to react to environment feedback (errors, partial results, new information)
- No fixed workflow could cover the range of situations the task might encounter

**Examples:**
- Debug a failing test suite — read errors, form hypotheses, edit code, re-run, repeat
- Research a topic across multiple sources, following leads as they emerge
- Manage a multi-step deployment where each step depends on the outcome of the last

---

## 2. Signal-to-Pattern Map

When you hear these signals in the user's task description, point them toward the corresponding pattern.

| # | Signal | Pattern | Why |
|---|--------|---------|-----|
| 1 | Single step, clear input/output | **Single LLM Call** | No orchestration needed. One prompt handles it. |
| 2 | Fixed sequence of dependent steps | **Prompt Chaining** | Each step feeds the next in a known order. Gate between steps for quality control. |
| 3 | Distinct input categories that need different handling | **Routing** | Classify first, then dispatch to specialized handlers. Keeps each path focused. |
| 4 | Independent subtasks that don't depend on each other | **Parallelization (Sectioning)** | Split the work, run in parallel, merge results. Faster and often higher quality. |
| 5 | Multiple perspectives or evaluations needed on the same input | **Parallelization (Voting)** | Run the same task multiple times (or with different prompts), then aggregate. Reduces variance. |
| 6 | Subtasks that can't be determined until you analyze the input | **Orchestrator-Workers** | A central LLM breaks down the task dynamically and delegates to specialized workers. |
| 7 | Clear quality criteria and output that improves with iteration | **Evaluator-Optimizer** | One LLM generates, another evaluates. Loop until quality threshold is met. |
| 8 | Fully open-ended task requiring environment feedback | **Autonomous Agent** | The model decides its own next steps based on what it observes. No predefined path. |

---

## 3. Complexity Escalation Checklist

Before moving to a higher complexity level, the user should answer YES to all four questions. If any answer is NO, stay at the current level and address the gap first.

| # | Question | What a NO means |
|---|----------|-----------------|
| 1 | **Have you fully optimized your prompt and retrieval at the current level?** | You may be leaving performance on the table. Better prompts, better context, or better retrieval could close the gap without adding complexity. |
| 2 | **Does the added complexity measurably improve outcomes?** | If you can't demonstrate the improvement, you're adding cost and fragility for speculative benefit. Get a baseline first. |
| 3 | **Can you test and debug the more complex system reliably?** | Multi-step systems are harder to diagnose. If you don't have visibility into intermediate steps, you'll spend more time debugging than building. |
| 4 | **Is the latency and cost tradeoff acceptable?** | More steps mean more API calls, more tokens, more latency. Make sure the user experience and budget can absorb the increase. |

---

## 4. Anti-Patterns

Common mistakes to flag when coaching. Each is a one-line diagnostic — expand conversationally if the user's plan shows the symptom.

| # | Anti-Pattern | Description |
|---|-------------|-------------|
| 1 | **Premature complexity** | Reaching for agents or multi-step workflows when a single well-crafted prompt would work. The most common mistake. |
| 2 | **Framework lock-in** | Choosing a framework before understanding the task. The framework should follow from the pattern, not the other way around. |
| 3 | **Poor tool design** | Giving the model tools with vague descriptions, overlapping functionality, or no examples. The model can only be as good as its tool definitions. |
| 4 | **Missing ground truth** | Building complex systems without evaluation data. If you can't measure whether the output is correct, you can't improve the system. |
| 5 | **No sandbox** | Letting agents take real-world actions (send emails, write to databases, deploy code) without a testing environment. Mistakes at agent speed are expensive. |
| 6 | **Ignoring cost/latency** | Adding pipeline steps without tracking the cumulative cost and response time. Users notice when a "simple" task takes 30 seconds and costs $0.50. |
| 7 | **Monolithic prompts** | Stuffing everything into one massive system prompt instead of decomposing into focused steps. Hurts both quality and debuggability. |
| 8 | **Overloading with tools** | Giving the model 20+ tools when it only needs 3-5 for the task. More tools means more confusion, more hallucinated tool calls, and slower responses. |
