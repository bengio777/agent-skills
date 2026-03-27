# Pattern Catalog Reference

Detailed reference for each of the 8 agentic patterns. Used during Stages 4 (trade-off discussion) and 5 (recommendation delivery). Look up a pattern by name, explain it conversationally, and cite the trade-offs and pitfalls that matter for the user's specific task.

---

### 1. Single LLM Call

**How it works:**
One prompt in, one response out. No tools, no retrieval, no chaining — just a well-crafted prompt and the model's response. This is the baseline every other pattern builds on, and you should only move past it when you've confirmed it can't do the job.

**Choose this when:**
- The task can be fully specified in a single prompt
- No external data, memory, or tool access is needed
- The input and output are both well-defined
- Prompt engineering alone can achieve the required quality
- You need the fastest, cheapest, most reliable option

**Real-world examples:**
- Summarize a meeting transcript into action items
- Classify a support ticket by priority and department
- Rewrite marketing copy for a different audience

**Key trade-offs:**

| Consideration | Impact |
|---------------|--------|
| Cost | Lowest — one API call, minimal tokens |
| Latency | Lowest — single round-trip |
| Complexity | Trivial to build, test, and debug |
| Error risk | Lowest — no compounding errors across steps |

**Common pitfall:** Dismissing this option too quickly. Most tasks that feel like they need orchestration actually just need a better prompt. Exhaust prompt engineering before escalating.

---

### 2. Augmented LLM

**How it works:**
A single LLM call enhanced with retrieval (RAG), tool use, and/or memory. The model still performs one logical step, but it actively pulls in external context or takes actions through tools to produce its response. This is the foundation for every multi-step pattern — if the augmented LLM can't use its tools well in a single step, it won't use them well across many steps.

**Choose this when:**
- The task needs real-time data or external context the model doesn't have
- A single prompt + tool call or retrieval step can produce the answer
- The model needs to take an action (write to a database, send a message, call an API)
- You don't need to decompose the task into multiple dependent steps

**Real-world examples:**
- Answer a question using RAG over a company knowledge base
- Look up a customer record, then draft a personalized response
- Query a weather API and summarize the forecast for a trip

**Key trade-offs:**

| Consideration | Impact |
|---------------|--------|
| Cost | Low — one LLM call plus tool/retrieval overhead |
| Latency | Low-to-moderate — adds retrieval or tool call time to the base call |
| Complexity | Moderate — requires well-designed tool definitions and retrieval pipelines |
| Error risk | Low — but poorly designed tools or vague descriptions degrade quality fast |

**Common pitfall:** Giving the model tools with vague descriptions, overlapping functionality, or no usage examples. The model can only be as good as the tool definitions you provide.

---

### 3. Prompt Chaining

**How it works:**
A fixed sequence of LLM calls where each step processes the output of the previous one. Between steps, validation gates check intermediate results — if a gate fails, the chain can retry or exit early. The key word is *fixed*: the sequence is determined at design time, not by the model at runtime.

**Choose this when:**
- The task decomposes naturally into ordered, dependent subtasks
- Each subtask has a clear input/output contract
- You want quality gates between steps to catch errors early
- The same sequence works reliably every time for this category of task
- A single prompt would be too complex or produce inconsistent results

**Real-world examples:**
- Generate marketing copy, then translate it into three languages
- Outline a document, validate the outline against requirements, then draft each section
- Extract structured data from a document, validate it against a schema, then write it to a database

**Key trade-offs:**

| Consideration | Impact |
|---------------|--------|
| Cost | Moderate — multiple LLM calls, but each is focused and efficient |
| Latency | Moderate — steps are sequential, so latency is additive |
| Complexity | Low-to-moderate — straightforward to build and debug because each step is isolated |
| Error risk | Lower than a monolithic prompt — gates catch problems early before they propagate |

**Common pitfall:** Making the chain too long. Every step adds latency and a potential failure point. If your chain has more than 4-5 steps, check whether some steps can be combined or run in parallel instead.

---

### 4. Routing

**How it works:**
An initial classifier examines the input and dispatches it to one of several specialized routes. Each route has its own prompt, tools, and potentially its own model — optimized for that specific category of input. The classifier can be an LLM call or a simpler function (keyword match, regex, embedding similarity).

**Choose this when:**
- Inputs fall into distinct categories that need fundamentally different handling
- A one-size-fits-all prompt would be mediocre at everything instead of great at each category
- You want to optimize cost by sending simple inputs to smaller/cheaper models
- The classification boundary is clear and reliable

**Real-world examples:**
- Customer service triage: route billing questions to one handler, technical issues to another, account changes to a third
- Model routing by difficulty: send simple queries to a fast/cheap model, complex reasoning tasks to a frontier model
- Content moderation: classify content type first, then apply category-specific evaluation criteria

**Key trade-offs:**

| Consideration | Impact |
|---------------|--------|
| Cost | Can reduce cost overall by matching model size to task difficulty |
| Latency | Adds one classification step; routed paths run independently |
| Complexity | Moderate — you're maintaining multiple specialized paths plus the classifier |
| Error risk | Misclassification sends the input down the wrong path entirely; needs robust fallback handling |

**Common pitfall:** Making routes too granular. If you have more than 5-6 routes, the classifier becomes unreliable and the system becomes hard to maintain. Start with 2-3 routes and add only when data shows you need them.

---

### 5. Parallelization

Two sub-patterns under one umbrella — both run multiple LLM calls simultaneously, but for different reasons.

#### Sectioning

**How it works:**
Split a task into independent subtasks, run them simultaneously in parallel, then combine the results. Each subtask handles a different piece of the work. The key insight: LLMs perform better when each call focuses on a single concern rather than juggling many at once.

**Choose this when:**
- The task has clearly separable subtasks that don't depend on each other
- Processing speed matters and sequential execution is too slow
- Each subtask benefits from focused, dedicated attention
- Results can be meaningfully combined at the end

**Real-world examples:**
- Analyze a document for tone, factual accuracy, and grammar — three parallel calls, merged into one report
- Process multiple customer reviews simultaneously, each getting its own analysis
- Generate descriptions for 50 products by splitting into parallel batches

#### Voting

**How it works:**
Run the same task multiple times (with the same or different prompts/models), then aggregate the results to get a higher-confidence answer. Variations include majority voting, weighted scoring, or selecting the best response from the set.

**Choose this when:**
- The task has high variance and you need more consistent results
- Correctness matters more than speed or cost
- Multiple valid approaches exist and you want to consider several
- You're building a safety-critical pipeline where a single LLM judgment isn't enough

**Real-world examples:**
- Run a code review three times with different prompts, flag issues that appear in at least two reviews
- Have three models evaluate a content moderation case, take the majority decision
- Generate five candidate headlines, then score and select the best one

**Key trade-offs (both sub-patterns):**

| Consideration | Impact |
|---------------|--------|
| Cost | Higher — multiplied by the number of parallel calls |
| Latency | Lower than sequential for sectioning (wall-clock time = slowest subtask); same per-call for voting |
| Complexity | Moderate — requires managing parallel execution and result aggregation |
| Error risk | Sectioning: errors are isolated to one subtask. Voting: reduces individual call variance |

**Common pitfall:** Parallelizing tasks that actually have hidden dependencies. If subtask B needs context from subtask A's output to produce a good result, running them in parallel will degrade quality silently — you'll get an answer, but it won't be as good.

---

### 6. Orchestrator-Workers

**How it works:**
A central orchestrator LLM analyzes the task, breaks it into subtasks dynamically at runtime, delegates each subtask to specialized worker LLMs, then synthesizes their results into a final output. The critical difference from parallelization: the subtasks are *not* predetermined. The orchestrator decides what work needs to happen based on the specific input.

**Choose this when:**
- The subtasks can't be predicted or hardcoded in advance
- Different inputs require different decompositions
- Workers need specialized capabilities (different tools, models, or prompts)
- The task involves coordinating work across multiple domains or data sources

**Real-world examples:**
- Multi-file code refactoring: orchestrator analyzes the codebase, identifies which files need changes, assigns each edit to a worker, then verifies consistency
- Research synthesis: orchestrator breaks a question into sub-questions, workers research each one from different sources, orchestrator integrates the findings
- Content localization: orchestrator determines which sections need cultural adaptation vs. direct translation, assigns workers accordingly

**Key trade-offs:**

| Consideration | Impact |
|---------------|--------|
| Cost | High — orchestrator calls plus all worker calls, often with substantial context in each |
| Latency | High — orchestrator planning step, then worker execution (parallel or sequential), then synthesis |
| Complexity | High — requires designing the orchestrator's planning logic, worker interfaces, and result integration |
| Error risk | Moderate-to-high — orchestrator can misdecompose the task; worker errors must be caught and handled |

**Common pitfall:** Using orchestrator-workers when the subtasks are actually predictable. If you know the decomposition in advance every time, use prompt chaining or parallelization instead — they're simpler, cheaper, and more reliable.

---

### 7. Evaluator-Optimizer

**How it works:**
A generator LLM produces output, then an evaluator LLM assesses it against defined criteria and provides specific feedback. The generator uses that feedback to produce an improved version. This loop repeats until the evaluator's criteria are met or a maximum iteration count is reached. The key insight: this works best when evaluation is substantially easier than generation.

**Choose this when:**
- Clear, measurable evaluation criteria exist (rubric, test suite, style guide)
- The output meaningfully improves with iterative refinement
- Evaluation is easier or more reliable than generation (easier to judge good writing than to write it)
- You need human-level quality and a single pass isn't enough
- You can define a concrete stopping condition

**Real-world examples:**
- Literary translation: generator translates, evaluator checks for meaning preservation, cultural nuance, and naturalness — loops until both accuracy and fluency score above threshold
- Code generation: generator writes code, evaluator runs it against a test suite and returns failing tests as feedback — loops until all tests pass
- Business proposal drafting: generator writes a section, evaluator scores it against a rubric (clarity, specificity, persuasiveness) — loops until each dimension meets the bar

**Key trade-offs:**

| Consideration | Impact |
|---------------|--------|
| Cost | High — at minimum 2 LLM calls per iteration, multiplied by number of loops |
| Latency | High — each iteration is sequential (generate, then evaluate), and you don't know the loop count in advance |
| Complexity | Moderate — straightforward loop structure, but requires well-defined evaluation criteria and feedback format |
| Error risk | Moderate — generator can get stuck in loops if feedback is vague; needs a max iteration cap as a safety valve |

**Common pitfall:** Using this pattern without clear evaluation criteria. If the evaluator can't articulate *specific, actionable* feedback, the generator just churns without improving — burning tokens and time without converging on better output.

---

### 8. Autonomous Agent

**How it works:**
An LLM operates in a loop — observe the environment, decide on the next action, execute it, observe the result, repeat. Unlike all previous patterns, the sequence of actions is fully open-ended and determined entirely at runtime by the model's judgment. The agent continues until it decides the task is complete or hits a resource limit.

**Choose this when:**
- The task is genuinely open-ended and the path to completion can't be predicted
- Steps depend on environment feedback (file contents, API responses, error messages, user input)
- No fixed workflow could cover the range of situations the task encounters
- You're willing to accept higher cost, latency, and error risk for the flexibility

**Real-world examples:**
- Debugging a failing test suite: read error output, form a hypothesis, edit code, re-run tests, observe new results, repeat until green
- Penetration testing: scan for vulnerabilities, attempt exploits based on findings, escalate based on results
- Open-ended research: search for information, follow leads, evaluate sources, synthesize findings — path depends entirely on what's discovered

**Key trade-offs:**

| Consideration | Impact |
|---------------|--------|
| Cost | Highest — unknown number of LLM calls, often with large context windows |
| Latency | Highest — sequential loop with no predetermined end point |
| Complexity | Moderate to build (it's often simpler than expected), but hard to test and predict |
| Error risk | Highest — errors compound across steps; agent can get stuck in loops or take wrong paths |

**Common pitfall:** Deploying an autonomous agent without a sandbox. Agents take real-world actions at machine speed — a misfire that sends 1,000 emails or drops a database table happens before anyone can intervene. Always sandbox first, then grant permissions incrementally.