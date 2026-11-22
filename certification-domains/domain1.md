# **Claude Certified Architect – Professional (CCA-P)**

## **Domain 1: Solution Design & Architecture (17%)**

Exam-focused study guide | July 2026 scope | 6 task statements

**Central principle:** ⭐ Choose the simplest architecture that meets the business requirements, quality targets, security constraints, and performance SLAs. Add agentic complexity only when its measurable benefits justify the additional cost, latency, and operational risk.

## **Part 1: Per Sub-Domain Breakdown**

### **1.1 Translate business problems into Claude-based AI solutions**

#### **Core Concept**

An architect must translate a business problem into a solution with clearly defined capabilities, constraints, success criteria, and operational responsibilities.

The exam tests whether you can distinguish a genuine AI requirement from a problem better solved using deterministic software, traditional machine learning, or process redesign.

**⭐ The correct starting point is the business outcome, not the model or agent framework.**

| Business requirement                               | Preferred approach                               | Architectural reasoning                                     |
| -------------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| Calculate employee tax using fixed rules           | Deterministic rules engine                       | Exactness, reproducibility, auditability                    |
| Summarize complex legal agreements                 | Claude with document context                     | Handles unstructured language and contextual interpretation |
| Extract contract clauses into predefined fields    | Claude with structured outputs and validation    | Flexible extraction with schema-controlled results          |
| Answer questions using internal company policies   | Claude + RAG                                     | Grounds responses in controlled enterprise knowledge        |
| Process invoices through a fixed approval sequence | Workflow + targeted Claude steps                 | Predictable control flow and limited AI involvement         |
| Investigate an unfamiliar acquisition target       | Tool-using agent or orchestrated research agents | Requires iterative discovery and adaptive reasoning         |
| Approve high-value financial transactions          | Claude recommendation + human authorization      | High-impact decisions require appropriate external controls |

**Architectural decision criteria**

- **Task variability:** Fixed and predictable tasks favor deterministic workflows; open-ended tasks may justify agents.
- **Accuracy requirements:** Use retrieval, validation, and escalation for tasks requiring high factual reliability.
- **Operational constraints:** Define expected throughput, P95 latency, availability, cost per task, and data residency requirements.
- **Risk and reversibility:** Separate recommendations from actions that change records, transfer money, or create legal obligations.
- **Business value:** Measure outcomes such as processing time, cost per completed task, automation rate, or improved decision quality.

#### **Canonical Skeleton**

```
Business problem
      |
      v
Define desired outcome and baseline
      |
      v
Identify inputs, users, data, and constraints
      |
      v
Decompose AI versus deterministic responsibilities
      |
      v
Choose architecture
  ├── Rules / traditional software
  ├── Single Claude call
  ├── Claude + retrieval / tools
  ├── Deterministic LLM workflow
  └── Autonomous or multi-agent system
      |
      v
Define acceptance criteria and SLAs
      |
      v
Prototype → Evaluate against baseline
      |
      v
Deploy with controls → Measure business impact
```

#### **Best Practices**

- ⭐ **Begin with the simplest viable implementation.** Anthropic recommends optimizing single LLM calls before introducing complex agentic systems.
- Define measurable success criteria before selecting models, prompts, or orchestration frameworks.
- Separate probabilistic reasoning from deterministic calculations, authorization, validation, and business rules.
- Validate actual business outcomes, not merely whether Claude produces plausible or fluent answers.
- Design human review and escalation around consequences, uncertainty, and organizational policy—not an arbitrary confidence score.

**Exam heuristic:** If the scenario includes a measurable business problem but no evidence that autonomous planning is required, avoid selecting a multi-agent architecture simply because it is more sophisticated.

### **1.2 Design end-to-end architectures (input → processing → output → feedback loops)**

#### **Core Concept**

An end-to-end Claude architecture includes more than a model invocation. It must connect input ingestion, preprocessing, context retrieval, inference, tool execution, validation, delivery, evaluation, and continuous improvement.

A production architect must also account for identity, authorization, observability, reliability, and failure recovery.

**⭐ Feedback is part of the architecture, not a separate feature added after deployment.**

#### **Canonical Skeleton**

```
                User / Enterprise Application
                            |
                            v
                  API / Authentication
                            |
                            v
                Input Validation / Policy
                            |
                            v
                 Preprocessing / Routing
                            |
                            v
                    Context Assembly
                  /        |         \
              RAG      Tool Access    Memory
                  \        |         /
                            v
                     Claude Model
                            |
                  Tool-use loop, if needed
                            |
                            v
                 Output / Schema Validation
                            |
                  Safety / Business Rules
                            |
                            v
                 Human Approval (if needed)
                            |
                            v
                 Response / System Update
                            |
                            v
               Feedback + Traces + Metrics
                            |
                            v
                  Evaluation Dataset
                            |
                            v
                 Improve / Test / Release
```

**Critical architectural layers**

| Layer          | Responsibility                                          | Typical failure                                  |
| -------------- | ------------------------------------------------------- | ------------------------------------------------ |
| Input          | Authentication, validation, normalization               | Malformed or unauthorized requests               |
| Context        | Retrieve relevant, permitted information                | Stale data, irrelevant chunks, excessive tokens  |
| Reasoning      | Interpret and solve the task                            | Incorrect inference or unsupported conclusions   |
| Tool execution | Retrieve data or perform authorized actions             | Wrong tool, invalid arguments, permission misuse |
| Validation     | Check schema, facts, policy, and constraints            | Invalid or ungrounded output                     |
| Delivery       | Return response or update enterprise systems            | Duplicate writes, failed transactions            |
| Evaluation     | Measure accuracy, cost, latency, safety                 | Silent quality regressions                       |
| Feedback       | Convert failures and user corrections into improvements | Repeated production errors                       |

**Design trade-offs**

| Design choice                                     | Benefit                                   | Cost or risk                                            |
| ------------------------------------------------- | ----------------------------------------- | ------------------------------------------------------- |
| RAG instead of embedding all knowledge in prompts | Fresher, selective context                | Retrieval latency and indexing complexity               |
| Structured JSON output                            | Reliable downstream parsing               | Schema constraints do not guarantee factual correctness |
| Parallel independent calls                        | Lower overall latency                     | Higher concurrent API demand                            |
| Human approval                                    | Reduced risk for consequential actions    | Additional turnaround time                              |
| Prompt caching                                    | Lower repeated-prefix cost and latency    | Requires sufficiently reusable prompt prefixes          |
| Synchronous API calls                             | Immediate results                         | Constrained by interactive latency budgets              |
| Message Batches API                               | Lower cost for eligible asynchronous work | Not suitable for immediate responses                    |

#### **Best Practices**

- ⭐ Instrument every important stage with request IDs, tool-call traces, latency, token usage, errors, and outcome metrics.
- Validate outputs using schemas and deterministic business checks; use human review where consequences justify it.
- Handle API errors, timeouts, retries, and partial failures explicitly. Make side-effecting operations idempotent where possible.
- Build representative evaluation datasets from actual tasks, edge cases, and observed production failures.
- Keep retrieval, orchestration, model access, evaluation, and business logic sufficiently modular to change independently.

**Exam heuristic:** When asked for the best production design, prefer the option that closes the entire operational loop rather than one that only improves model accuracy.

### **1.3 Select appropriate architectural patterns (workflow, agentic, augmented LLM)**

#### **Core Concept**

This is one of Domain 1's highest-yield objectives.

Anthropic distinguishes between:

- **Augmented LLM:** A Claude model enhanced with retrieval, tools, or memory.
- **Workflow:** LLM calls and tools follow predefined code paths.
- **Agent:** Claude dynamically chooses actions, tools, and subsequent steps based on intermediate results.

⭐ **The determining factor is who controls the execution path: application code or Claude.**

#### **Canonical Skeleton**

```
                         Business Task
                              |
                 Can rules solve it reliably?
                        /            \
                      Yes             No
                       |               |
                 Rules engine    Is one LLM call
                                 sufficient?
                                   /     \
                                 Yes      No
                                  |        |
                            Claude /       |
                           Augmented LLM   |
                                            v
                              Are steps predictable?
                                  /          \
                                Yes           No
                                 |             |
                          Fixed Workflow   Does Claude need
                           /    |    \      to dynamically
                       Chain  Route Parallel plan and act?
                                             /     \
                                           No      Yes
                                            |       |
                                        Reassess  Agent
                                                   |
                                      Multiple independent
                                       workstreams needed?
                                              /    \
                                            No      Yes
                                             |       |
                                       Single agent
                                               Orchestrator
                                               + workers
```

**Architecture selection matrix**

| Pattern              | When it is BEST                                          | Trade-off                                                     |
| -------------------- | -------------------------------------------------------- | ------------------------------------------------------------- |
| Single LLM call      | One well-bounded generation or classification task       | Lowest complexity; limited external context                   |
| Augmented LLM        | One reasoning step requiring retrieval, tools, or memory | More capable, but additional context/tool overhead            |
| Prompt chaining      | Fixed, dependent processing stages                       | Better control and stage-level validation; cumulative latency |
| Routing              | Inputs belong to distinct, identifiable categories       | Specialization and cost efficiency; misrouting risk           |
| Parallelization      | Independent subtasks or multiple evaluation perspectives | Reduced wall-clock time; higher concurrent usage              |
| Orchestrator-workers | Claude must decide the decomposition dynamically         | Adaptive specialization; coordination overhead                |
| Evaluator-optimizer  | Quality improves through iterative critique              | Better results; additional inference cost and latency         |
| Autonomous agent     | Steps cannot be predicted and the system must adapt      | Maximum flexibility; less predictable cost and behavior       |

These patterns follow Anthropic's [Building effective agents](https://www.anthropic.com/research/building-effective-agents?utm_source=chatgpt.com).

#### **Five patterns to recognize immediately**

**1. Prompt chaining**

```
Document
   |
   v
Extract facts
   |
 Validate
   |
   v
Analyze risks
   |
 Validate
   |
   v
Generate report
```

Use when each stage depends on the previous stage. Prefer intermediate checks over allowing an early error to propagate silently.

**2. Routing**

```
Incoming request
       |
       v
    Classifier
    /   |    \
   /    |     \
Simple Complex  Sensitive
  |      |        |
Haiku  Sonnet/   Human /
       Opus     Controlled path
```

Use when a reliable classifier can direct different request types to specialized prompts, models, or processes.

**3. Parallelization**

```
               Contract
                  |
          +-------+-------+
          |       |       |
       Privacy  Payment  Termination
        review   review    review
          |       |       |
          +-------+-------+
                  |
              Aggregate
```

Use when work is independent. The aggregation step must still reconcile contradictions.

**4. Orchestrator-workers**

```
             Research Objective
                    |
               Lead Claude
                    |
          Dynamically plans work
             /      |      \
          Agent A Agent B Agent C
             \      |      /
                Synthesis
```

Use when subtasks cannot be reliably predetermined.

**5. Evaluator-optimizer**

```
Generate draft
      |
      v
Evaluate against rubric
      |
      +--- Fail → Revise → Re-evaluate
      |
     Pass
      |
      v
   Deliver
```

Use when evaluation criteria are sufficiently clear and additional iterations produce measurable improvements. Always establish a termination condition.

#### **Best Practices**

- ⭐ Start with a single call or augmented LLM. Move to workflows or agents only when evaluation demonstrates a benefit.
- Use fixed workflows for predictable business processes; reserve autonomous agents for tasks that genuinely require dynamic planning.
- Parallelize only independent work. Dependent stages still require sequencing.
- Provide agents with clear tools, environmental feedback, stopping conditions, and controls over consequential actions.
- Avoid unnecessary orchestration-framework abstraction; understand the underlying model calls and tool execution.

**Exam heuristic:** When two architectures achieve equivalent quality, favor the one with fewer model calls, less orchestration, and a more predictable failure surface.

### **1.4 Design multi-agent systems and orchestration strategies**

#### **Core Concept**

A multi-agent architecture divides complex work among specialized Claude instances, typically under a coordinating agent.

The primary justification is not that multiple agents are inherently more intelligent. The benefits come from **parallel execution, focused contexts, specialized tools, and effective division of labor**.

Anthropic's multi-agent research architecture demonstrated these principles in production.

#### **Canonical Skeleton**

```
                   User Request
                        |
                        v
                 Lead Orchestrator
                 - Understand goal
                 - Create task plan
                 - Allocate budget
                 - Delegate work
                        |
              +---------+---------+
              |         |         |
              v         v         v
          Agent A    Agent B    Agent C
          Research   Financial Technical
              |         |         |
          Tools A    Tools B    Tools C
              |         |         |
              +---------+---------+
                        |
                 Structured Results
                        |
                        v
                 Lead Orchestrator
                 - Remove duplicates
                 - Resolve conflicts
                 - Check completeness
                 - Synthesize evidence
                        |
                        v
                  Final Validation
                        |
                        v
                       User
```

**Multi-agent orchestration choices**

| Strategy                         | Characteristics                               | Best use case                                           |
| -------------------------------- | --------------------------------------------- | ------------------------------------------------------- |
| Centralized orchestrator         | One lead agent delegates and synthesizes      | Enterprise research, complex analysis                   |
| Parallel specialists             | Independent workers execute concurrently      | Due diligence across unrelated dimensions               |
| Sequential handoffs              | One specialist passes its output to another   | Analysis → critique → final document                    |
| Hierarchical delegation          | Coordinators manage smaller groups of workers | Large, decomposable tasks with distinct workstreams     |
| Single agent with multiple tools | One context, no agent-to-agent coordination   | Tasks needing varied tools but limited independent work |

**⭐ Single-agent vs. multi-agent decision**

| Dimension        | Single agent                                           | Multi-agent                                     |
| ---------------- | ------------------------------------------------------ | ----------------------------------------------- |
| Coordination     | Simpler                                                | More complex                                    |
| Context          | Shared working context                                 | Separate worker contexts                        |
| Independent work | Usually serialized unless parallel tool calls are used | Parallel delegation possible                    |
| Cost             | Typically lower                                        | Often substantially higher                      |
| Debugging        | Easier to trace                                        | Requires cross-agent traces                     |
| Best fit         | Connected tasks with shared state                      | Independent research or analysis                |
| Failure risk     | Context overload and long tool loops                   | Duplication, missed tasks, conflicting findings |

Anthropic reported approximately **15× the tokens of ordinary chat interactions** for multi-agent research systems in its 2025 analysis. This is an observed workload comparison, not a universal cost multiplier or exam-mandated limit.

#### **Best Practices**

- ⭐ Give every worker a specific objective, scope, toolset, expected output format, and stopping criteria.
- Scale the number of workers to task complexity. Do not spawn agents simply because infrastructure permits it.
- Return concise, evidence-backed findings to the orchestrator instead of copying entire worker histories into the lead context.
- Design for partial failures using checkpoints, retries, resumability, and clear escalation.
- Evaluate final task success and coordination quality, including duplicate work, omissions, tool efficiency, and unsupported claims.

Anthropic's [multi-agent research system analysis](https://www.anthropic.com/research/multi-agent-research-system?utm_source=chatgpt.com) is particularly useful for this objective.

**Exam heuristic:** If a task requires every worker to continuously share the same mutable state, multi-agent decomposition may create more problems than it solves.

### **1.5 Apply decomposition techniques for complex problem solving**

#### **Core Concept**

Decomposition converts a difficult task into smaller, manageable units that can be solved, evaluated, and combined.

The exam tests whether you can determine **how to split work, which dependencies matter, and when decomposition improves reliability rather than merely adding complexity**.

⭐ Decompose along meaningful task boundaries, not arbitrary divisions.

**Primary decomposition strategies**

| Technique                      | Description                                             | Example                                                             |
| ------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------------- |
| Sequential decomposition       | Each step depends on earlier results                    | Extract facts → analyze → summarize                                 |
| Parallel decomposition         | Independent subtasks execute concurrently               | Analyze privacy, payment, and liability clauses                     |
| Hierarchical decomposition     | Large objectives split into nested subproblems          | Market analysis → regions → companies                               |
| Dynamic decomposition          | Claude decides which subtasks are needed                | Open-ended research or unfamiliar codebase investigation            |
| Context-based decomposition    | Separate work to prevent context overload               | Different agents analyze different document collections             |
| Complexity-based decomposition | Route simple and difficult subtasks to different models | Basic extraction with Haiku; difficult analysis with Sonnet or Opus |

#### **Canonical Skeleton**

```
                  Complex Objective
                         |
                         v
                Identify subtasks
                         |
                         v
                Map dependencies
                         |
             +-----------+-----------+
             |                       |
     Independent tasks        Dependent tasks
             |                       |
      Run in parallel          Run sequentially
             |                       |
             +-----------+-----------+
                         |
                         v
               Validate task outputs
                         |
                         v
                 Aggregate results
                         |
                         v
               Cross-check consistency
                         |
                         v
                  Final response
```

**Decomposition example: Contract due diligence**

```
                     Contract Package
                            |
                      Extract Text
                            |
                       Normalize
                            |
              +-------------+-------------+
              |             |             |
         Payment Risk   Liability Risk   Privacy Risk
              |             |             |
              +-------------+-------------+
                            |
                     Consolidate Risks
                            |
                     Resolve Conflicts
                            |
                     Generate Report
```

The extraction and normalization stages are shared prerequisites. The three risk-analysis stages are largely independent and can run in parallel.

**Critical distinction**

- **Parallelization:** The application already knows the subtasks.
- **Orchestrator-workers:** Claude determines the subtasks dynamically.
- **Autonomous agent:** Claude also determines its subsequent actions and when additional work is needed.

#### **Best Practices**

- ⭐ Identify dependencies before choosing sequential or parallel execution.
- Give subtasks clear inputs, output contracts, acceptance criteria, and ownership boundaries.
- Use context isolation when specialized workers need only a subset of the available information.
- Set budgets for tokens, tool calls, execution time, retries, and refinement cycles.
- Validate intermediate outputs and final synthesis, especially when agents may produce inconsistent or overlapping findings.

Anthropic's [context engineering guidance](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents?utm_source=chatgpt.com) explains why small, relevant working contexts and structured memory often outperform indiscriminate context accumulation.

**Exam heuristic:** If a scenario mentions excessive context, repetitive work, poor parallel execution, or long sequential processing, inspect the decomposition strategy before recommending a stronger model.

### **1.6 Align solutions to business value pillars (efficiency, transformation, productivity, cost, performance SLAs)**

#### **Core Concept**

An architect must connect architectural decisions to measurable business results.

A technically successful agent can still be a poor business solution if it costs too much, misses latency targets, increases review burden, or delivers little incremental value.

⭐ **Optimize for successful business outcomes under binding constraints—not maximum model capability.**

#### **Business value pillars**

| Value pillar     | What it means                                     | Example KPIs                                         |
| ---------------- | ------------------------------------------------- | ---------------------------------------------------- |
| Efficiency       | Reduce time, manual effort, or operational waste  | Cycle time, automation rate, turnaround time         |
| Transformation   | Enable materially different business capabilities | New services, new revenue, expanded process coverage |
| Productivity     | Increase useful output per employee               | Cases per analyst, tasks completed, hours saved      |
| Cost             | Improve unit economics                            | Cost per task, cost per successful outcome, ROI      |
| Performance SLAs | Meet agreed service commitments                   | P95 latency, availability, success rate, throughput  |

#### **Canonical Skeleton**

```
                   Business Objective
                          |
                          v
               Establish Current Baseline
                          |
                          v
                Define Target Outcomes
                          |
                          v
                 Identify Constraints
                /         |         \
             Quality     Cost      Latency
                \         |         /
                          v
                 Compare Architectures
                 /         |         \
              Simple    Workflow    Agentic
                          |
                          v
              Evaluate Quality + Economics
                          |
                          v
                 Check Binding SLAs
                          |
                  Pass          Fail
                    |             |
                    |       Revise architecture
                    v
                Controlled Rollout
                          |
                          v
                  Measure Actual ROI
```

#### **Key architectural economics**

**1. Cost per completed task**

\\[ C\_{\text{task}} = C\_{\text{model}} + C\_{\text{tools}} + C\_{\text{infrastructure}} + C\_{\text{review}} \\]

**2. Cost per successful task**

\\[ C\_{\text{success}} = \frac{\text{Total operating cost}}{\text{Number of successful tasks}} \\]

This is often more meaningful than API cost per request. An inexpensive model that repeatedly fails may be more expensive per successful outcome.

**3. End-to-end latency**

For strictly sequential stages:

\\[ L\_{\text{total}} = \sum\_{i=1}^{n} L_i \\]

For parallel stages:

\\[ L\_{\text{parallel}} \approx \max(L_1,L_2,\ldots,L_n) + L\_{\text{aggregation}} \\]

These are simplified execution-path models. Retries, queuing, network overhead, and tail-latency effects must also be measured.

**4. Value realization**

\\[ ROI = \frac{\text{Benefits}-\text{Total Costs}}{\text{Total Costs}}\times100\\% \\]

Benefits must account for actual adoption, quality, human supervision, and failure recovery rather than assuming every automated action creates equivalent savings.

#### **Architecture comparison example**

An enterprise needs an internal research assistant with at least 92% acceptable-answer quality, P95 latency below 12 seconds, and an average processing cost below $0.15 per request.

Illustrative evaluation results, not Anthropic benchmarks.

| Architecture             | Quality | P95 latency | Cost/request | Decision                          |
| ------------------------ | ------- | ----------- | ------------ | --------------------------------- |
| Single Claude call + RAG | 93%     | 6s          | $0.04        | Meets all targets                 |
| Three-stage workflow     | 95%     | 11s         | $0.10        | Meets all targets                 |
| Multi-agent research     | 98%     | 35s         | $0.42        | Violates latency and cost targets |

The single-call architecture is the strongest default if its 93% quality is sufficient and no other constraints favor the workflow. The three-stage workflow may be justified if the additional quality creates material business value.

The multi-agent architecture has the highest quality but fails two binding constraints.

#### **Best Practices**

- ⭐ Define objective acceptance criteria and service-level targets before architectural selection.
- Compare architectures using representative tasks, including failures and difficult cases—not isolated demonstrations.
- Route requests according to complexity and use a higher-cost model only where its measured benefit is justified.
- Use prompt caching for repeated context and Message Batches for eligible asynchronous workloads.
- Combine automated evaluations with targeted human review to capture factual, operational, and business-quality failures.

Anthropic's [agent evaluation guidance](https://www.anthropic.com/research/demystifying-evals-for-ai-agents?utm_source=chatgpt.com) emphasizes task outcomes, graders, traces, and representative evaluations rather than subjective impressions.

**Exam heuristic:** When an option produces better answers but violates a mandatory SLA, it is not the best architecture unless the scenario explicitly permits changing that SLA.

## **Part 2: Consolidated Sections**

### **2. Exam Traps**

| Trap                                             | Why it is tempting                                 | Why it is wrong                                                                                   |
| ------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| ⭐ Choose multi-agent for every complex problem   | More agents appear to provide greater intelligence | Complex tasks may still follow predictable workflows; extra agents increase coordination and cost |
| Use the largest model for all requests           | Stronger reasoning may improve accuracy            | Model capability alone does not optimize cost per successful task or latency                      |
| Equate tool use with autonomy                    | Both workflows and agents call tools               | Tool access does not determine who controls execution flow                                        |
| Treat augmented LLM and agent as synonyms        | Both may use retrieval, memory, and tools          | An augmented LLM is a building block; an agent dynamically controls a process                     |
| Parallelize all processing stages                | Parallelization reduces latency                    | Data dependencies can make concurrent stages invalid                                              |
| Use orchestrator-workers for predefined tasks    | The pattern looks similar to parallelization       | Dynamic orchestration is unnecessary when subtasks are known in advance                           |
| Spawn more workers to improve completeness       | Wider exploration may increase coverage            | Duplicate searches, higher costs, and coordination failures may reduce efficiency                 |
| Pass complete worker histories to the lead agent | More context appears safer                         | Excessive context increases token costs and can bury relevant information                         |
| Let an agent retry indefinitely                  | Persistence may eventually solve difficult tasks   | Unbounded loops create unpredictable cost, latency, and operational risk                          |
| Assume structured output means correctness       | JSON validates successfully                        | Schema compliance does not establish factual accuracy or authorization                            |
| Use average latency to prove SLA compliance      | Averages are easy to calculate                     | A P95 SLA concerns tail latency, not average performance                                          |
| Assume prompt caching speeds up generation       | Cached prefixes reduce processing work             | Caching does not eliminate output-token generation time                                           |
| Use Message Batches for interactive requests     | Batch processing offers lower token costs          | Results are asynchronous and unsuitable for strict interactive response requirements              |
| Optimize API cost alone                          | Token spending is visible                          | Failed work, tool usage, human correction, and infrastructure may dominate total cost             |
| Rely exclusively on LLM-as-judge                 | Automated grading scales well                      | Judges can be biased, miss edge cases, or reward convincing but incorrect responses               |
| Require agents to follow one exact trajectory    | Deterministic tests are easier to create           | Different valid tool sequences can reach the same correct outcome                                 |

**⭐ Fast elimination technique:** Reject choices that introduce unnecessary autonomy, ignore a binding business constraint, or attempt to solve a deterministic control problem using a more powerful model.

### **3. Key Facts to Memorize**

#### **A. Architecture definitions**

| Term                | Definition to remember                                                            |
| ------------------- | --------------------------------------------------------------------------------- |
| Workflow            | Application-defined sequence of model and tool operations                         |
| Agent               | Model-directed loop in which Claude dynamically selects actions                   |
| Augmented LLM       | LLM enhanced with retrieval, tools, or memory                                     |
| RAG                 | Retrieval-augmented generation; grounds generation in retrieved information       |
| Orchestrator        | Coordinator that plans, delegates, and synthesizes work                           |
| Worker/subagent     | Specialized agent with a bounded delegated objective                              |
| ACI                 | Agent-computer interface; the tools and interfaces through which an agent acts    |
| HITL                | Human-in-the-loop; human validation or authorization at specified decision points |
| Context engineering | Selection and management of information placed in the model's working context     |
| Compaction          | Summarizing older context while preserving relevant state for continuation        |
| Agent harness       | Runtime that handles model calls, tools, state, and agent execution               |

#### **B. Pattern selection — memorize these triggers**

| Scenario phrase                                           | Likely preferred pattern                    |
| --------------------------------------------------------- | ------------------------------------------- |
| "Simple, predictable task"                                | Single Claude call                          |
| "Needs current enterprise knowledge"                      | Augmented LLM + RAG                         |
| "Known sequence of dependent tasks"                       | Prompt chaining                             |
| "Different request types require different processing"    | Routing                                     |
| "Independent analyses"                                    | Parallelization                             |
| "Subtasks cannot be predicted beforehand"                 | Orchestrator-workers                        |
| "Repeated critique demonstrably improves quality"         | Evaluator-optimizer                         |
| "Open-ended environment requiring exploration and action" | Autonomous agent                            |
| "Human judgment required before irreversible action"      | Human approval within a controlled workflow |
| "Existing code or rules solve it exactly"                 | Deterministic software                      |

#### **C. Metrics and evaluation terminology**

| Metric                      | What it measures                                    | Architectural relevance                    |
| --------------------------- | --------------------------------------------------- | ------------------------------------------ |
| Task success rate           | Percentage of tasks completed correctly             | Primary outcome measure                    |
| Precision                   | Correct positive results / all positive predictions | False-positive sensitivity                 |
| Recall                      | Correct positive results / all actual positives     | Missed-information sensitivity             |
| F1                          | Harmonic mean of precision and recall               | Balanced classification/extraction measure |
| Groundedness / faithfulness | Whether conclusions are supported by evidence       | RAG and research reliability               |
| P50 latency                 | Median response time                                | Typical experience                         |
| P95 latency                 | 95th-percentile response time                       | Tail-latency SLAs                          |
| TTFT                        | Time to first token                                 | Streaming responsiveness                   |
| Cost per success            | Total cost divided by successful tasks              | End-to-end economic efficiency             |
| Tool success rate           | Percentage of tool invocations completing correctly | Integration reliability                    |
| Escalation rate             | Percentage requiring human intervention             | Automation effectiveness                   |
| Cache hit rate              | Share of eligible reuse served from cache           | Repeated-context efficiency                |
| Throughput                  | Requests or completed tasks per unit time           | Scalability                                |

**Evaluation terminology**

- **Task:** One evaluation case with inputs and success criteria.
- **Trial:** One attempt to execute a task; repeat trials to assess variability.
- **Grader:** Programmatic, model-based, or human evaluation of an output or outcome.
- **Trace/trajectory:** Record of model interactions, tool calls, and intermediate results.
- **Outcome:** The actual end state, which may differ from what the agent claims to have done.

#### **D. Claude APIs, capabilities, and quantitative facts**

| Feature or fact               | July 2026 exam-relevant point                                                                        |
| ----------------------------- | ---------------------------------------------------------------------------------------------------- |
| Domain weight                 | **17%** of the exam blueprint                                                                        |
| Anthropic Messages API        | Primary API interface for Claude interactions                                                        |
| Client tool use               | Claude returns `tool_use`; the application executes the tool and supplies `tool_result`              |
| `stop_reason: "tool_use"`     | Indicates Claude is yielding control for tool execution                                              |
| Structured outputs            | JSON-schema-constrained output formatting; factual validation still required                         |
| `max_tokens`                  | Limits generated output tokens; not the total workflow's spending budget                             |
| Prompt caching                | Reuses eligible repeated prefixes of tools, system messages, and message content                     |
| Cache TTLs                    | 5 minutes and optional 1 hour, subject to model/platform support                                     |
| Message Batches API           | Approximately 50% token-price discount for eligible asynchronous processing                          |
| Batch processing window       | Designed for asynchronous execution; processing can take up to 24 hours                              |
| Extended/adaptive thinking    | Model reasoning capabilities with model-specific configuration and cost/latency trade-offs           |
| Claude Agent SDK              | Provides building blocks for tool-using agent applications                                           |
| MCP                           | Model Context Protocol for interoperable tool and context integrations                               |
| Multi-agent token consumption | Anthropic observed approximately 15× chat token usage in its research workload—not a universal limit |

**Model examples available by July 31, 2026**

| Model            | Exam-relevant positioning                                            |
| ---------------- | -------------------------------------------------------------------- |
| Claude Haiku 4.5 | Cost-efficient, lower-latency tasks                                  |
| Claude Sonnet 5  | Strong general-purpose and agentic workloads with balanced economics |
| Claude Opus 5    | High-capability reasoning and difficult agentic work                 |

Sonnet 5 was introduced June 30, 2026, and Opus 5 on July 24, 2026. Availability and feature support varied by platform; no particular model should be assumed to support every feature in every deployment.

**Important:** Architecture selection does not require memorizing a universal context-window size, token budget, maximum number of subagents, or SLA threshold. Such values are model-specific, platform-specific, or established by business requirements.

#### **E. Critical architecture rules**

| Rule                              | Remember                                                                                        |
| --------------------------------- | ----------------------------------------------------------------------------------------------- |
| ⭐ Simplicity first                | Start with the least complex architecture that meets requirements                               |
| Explicit success criteria         | Define what correct task completion means before building                                       |
| Deterministic controls            | Enforce permissions, business rules, and consequential-action safeguards outside model judgment |
| Evidence over plausibility        | Verify facts and actual outcomes, not just fluent responses                                     |
| Parallelism requires independence | Do not parallelize stages with unresolved dependencies                                          |
| Dynamic work requires boundaries  | Give agents tool limits, timeouts, budgets, and exit criteria                                   |
| Separate contexts when useful     | Workers should receive only information relevant to their tasks                                 |
| Test the entire system            | Evaluate model, prompts, retrieval, tools, orchestration, and environment together              |
| Tail latency matters              | Use the latency percentile specified by the SLA                                                 |
| Reversible vs. irreversible       | Increase validation and approval requirements as consequences increase                          |

### **4. Five Exam-Style Questions**

These are original practice scenarios designed to test architectural judgment rather than recall of product terminology.

#### **Question 1 — Business problem translation and end-to-end design**

An enterprise processes 40,000 purchase requests monthly. Most requests follow fixed approval rules. A small percentage contain ambiguous justifications requiring interpretation.

Management wants a scalable solution with strong auditability, low operating cost, and no unauthorized financial approvals.

Which approach is BEST?

**A.** Deploy a multi-agent architecture where separate Claude agents evaluate policy, budget, and approval eligibility.

**B.** Use a deterministic approval workflow, invoke Claude for ambiguous justifications, validate structured outputs, and require appropriate authorization for consequential approvals.

**C.** Send every purchase request to the highest-capability Claude model and ask it to decide whether to approve.

**D.** Train a classification model and automatically approve all requests receiving a sufficiently high confidence score.

**Correct answer: B**

**Why B is correct:** The known approval logic remains deterministic. Claude is used only where language interpretation adds value, and authorization remains under controlled business processes. This balances reliability, cost, and auditability.

**Why the others are wrong:**

- **A:** Introduces unnecessary orchestration and autonomy for largely predictable tasks.
- **C:** Makes probabilistic model judgment responsible for fixed rules and financial authorization.
- **D:** A confidence score does not replace enforceable policy or approval authority.

**Tested objectives:** 1.1, 1.2, 1.6.

#### **Question 2 — Architectural pattern selection**

A customer-support system receives billing questions, technical problems, and account-access requests.

The organization already has well-defined procedures for each category. The majority of requests are simple, but technical questions may require deeper reasoning. Latency and cost are major constraints.

Which architecture is BEST?

**A.** A single highly capable Claude model with every available support tool and one universal prompt.

**B.** A centralized orchestrator that creates specialized agents dynamically for every request.

**C.** A routing workflow that classifies requests, directs them to specialized handlers, chooses an appropriate model tier, and escalates sensitive cases.

**D.** Multiple autonomous agents processing every request simultaneously and selecting a response by majority vote.

**Correct answer: C**

**Why C is correct:** Categories and procedures are already known. Routing provides specialization, bounded tool access, and model-tier optimization without requiring autonomous task decomposition.

**Why the others are wrong:**

- **A:** Increases unnecessary cost and gives the model a broad tool surface.
- **B:** Dynamic decomposition adds little value when processing categories are predefined.
- **D:** Runs unnecessary parallel work; majority agreement does not guarantee a correct or authorized action.

**Tested objectives:** 1.3, 1.6.

#### **Question 3 — Multi-agent orchestration**

An investment research organization needs to investigate acquisition targets across financial filings, market trends, technology assets, and competitive positioning.

Research directions differ by target, the relevant information is spread across many sources, and much of the investigation can happen independently.

Accuracy and breadth are more important than immediate interactive latency.

Which approach is BEST?

**A.** Use a single Claude agent with all documents placed into its initial context.

**B.** Build a fixed four-stage sequential workflow that always investigates each category in the same order.

**C.** Deploy a lead orchestrator that dynamically delegates bounded research tasks to specialized workers, gathers evidence-backed summaries, resolves gaps, and synthesizes findings.

**D.** Spawn as many research agents as available compute capacity allows, with each agent independently investigating the full target.

**Correct answer: C**

**Why C is correct:** The task benefits from dynamic planning, independent research, context isolation, and parallel exploration. A lead orchestrator can control duplication, budget, completeness, and final synthesis.

**Why the others are wrong:**

- **A:** Risks context overload and serial exploration; it may be adequate for a small case but is poorly matched to the stated breadth.
- **B:** Unnecessarily serializes independent work and cannot adapt well when research directions differ.
- **D:** Duplicates work and creates unbounded coordination and token costs.

**Tested objectives:** 1.3, 1.4, 1.5.

#### **Question 4 — Decomposition and dependencies**

A legal AI application must process 2,000 contracts. For every contract, it must extract normalized clauses, assess privacy risks, assess payment risks, and generate a consolidated report.

Privacy and payment analysis are independent once normalized clauses exist.

The current system runs every operation sequentially, creating unacceptable latency.

What is the BEST improvement?

**A.** Run all stages simultaneously, including extraction and risk analysis, regardless of dependencies.

**B.** Replace the existing model with the highest-capability Claude model and keep the workflow unchanged.

**C.** Build a fully autonomous agent that independently decides the processing order of every contract.

**D.** Preserve extraction and normalization as prerequisites, execute independent risk analyses in parallel, then consolidate and validate results.

**Correct answer: D**

**Why D is correct:** It respects the dependency graph while parallelizing independent operations. This reduces unnecessary wall-clock latency without losing predictable execution or validation.

**Why the others are wrong:**

- **A:** Risk analysis may execute before its required input exists.
- **B:** A stronger model does not remove sequential processing dependencies.
- **C:** Introduces agentic overhead when the processing stages and dependencies are already known.

**Tested objectives:** 1.2, 1.3, 1.5.

#### **Question 5 — Business value, evaluation, and SLAs**

A company is selecting the production architecture for an employee knowledge assistant.

The mandatory requirements are:

- At least 94% acceptable-answer quality.
- P95 latency no higher than 10 seconds.
- Average processing cost no higher than $0.12 per request.
- All answers must use authorized enterprise information.

The evaluation results are:

| Architecture                       | Quality | P95 latency | Cost/request |
| ---------------------------------- | ------- | ----------- | ------------ |
| A. Single Claude + RAG             | 95%     | 7s          | $0.05        |
| B. Prompt chaining                 | 96%     | 9s          | $0.09        |
| C. Multi-agent system              | 99%     | 22s         | $0.30        |
| D. Single Claude without retrieval | 97%     | 4s          | $0.04        |

Which approach is BEST?

**A.** Select Architecture A because it meets all quantified constraints while retaining authorized retrieval and the lowest cost among compliant options.

**B.** Select Architecture B because it has higher accuracy than A, regardless of the incremental cost.

**C.** Select Architecture C because architectural quality should always outweigh cost and latency.

**D.** Select Architecture D because it has the lowest latency and cost.

**Correct answer: A**

**Why A is correct:** It meets quality, cost, and latency targets while using authorized retrieval. Architecture B is also feasible, but its marginal quality benefit has not been shown to justify the higher expense and latency.

**Why the others are wrong:**

- **B:** The higher quality is attractive but does not automatically justify additional cost; business value must establish that benefit.
- **C:** Violates two mandatory numerical constraints.
- **D:** Omits the required enterprise retrieval mechanism; its reported quality does not establish reliable use of authorized knowledge.

**Tested objectives:** 1.1, 1.2, 1.6.

### **5. Official Anthropic Reading — July 2026 Scope**

These resources directly support the architectural concepts above.

| Resource                                                                                                                                             | Highest-yield topics                                                        |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [Building effective agents](https://www.anthropic.com/research/building-effective-agents?utm_source=chatgpt.com)                                     | Workflow vs. agent, augmented LLM, pattern selection, complexity trade-offs |
| [How we built our multi-agent research system](https://www.anthropic.com/research/multi-agent-research-system?utm_source=chatgpt.com)                | Orchestrator-workers, delegation, parallel execution, cost and coordination |
| [Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents?utm_source=chatgpt.com) | Context isolation, compaction, persistent notes, agent decomposition        |
| [Demystifying evals for AI agents](https://www.anthropic.com/research/demystifying-evals-for-ai-agents?utm_source=chatgpt.com)                       | Task success, trial, graders, traces, human and automated evaluation        |
| [Tool use with Claude](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview?utm_source=chatgpt.com)                                | Tool execution, tool schemas, client and server tools                       |
| [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching?utm_source=chatgpt.com)                                        | Reusable prefixes, TTLs, caching economics                                  |
| [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing?utm_source=chatgpt.com)                                    | Asynchronous workload optimization and batch economics                      |

The architectural principles are grounded in material available by July 2026. Live API documentation can change, so model-specific parameters, feature support, and limits should be interpreted according to their July 2026 availability rather than later updates.

### **Last-Minute Revision Checklist**

- ⭐ Distinguish workflows (code-controlled) from agents (model-controlled).
- Know when a single augmented LLM is sufficient.
- Memorize prompt chaining, routing, parallelization, orchestrator-workers, and evaluator-optimizer.
- Distinguish static parallelization from dynamic task decomposition.
- Know when separate worker contexts and parallel execution justify multi-agent overhead.
- Define task boundaries, tool permissions, stopping criteria, and failure recovery.
- Keep deterministic rules and authorization outside probabilistic model decisions.
- Evaluate actual outcomes using quality, latency, cost, reliability, and human review.
- Recognize prompt caching and Message Batches as workload-specific optimizations.
- ⭐ Reject architectures that violate binding SLAs, even when they offer higher model quality.

**Final exam rule:** The best architecture is the least complex system that reliably satisfies the complete set of business, technical, operational, and governance requirements.
