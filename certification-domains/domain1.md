# **Claude Certified Architect – Professional (CCA-P)**

## **Domain 1: Solution Design & Architecture (17%)**

**Exam scope:** July 2026 (Version 1.0)\
**Focus:** Architectural decision-making, pattern selection, orchestration, decomposition, and business outcomes.

⭐ **Core exam principle:** Choose the simplest architecture that meets the business requirements, quality thresholds, security constraints, and performance SLAs. Add agentic complexity only when evidence justifies it.

## **Domain 1 — Official Task Statements**

The July 2026 blueprint defines six task statements.

| #   | Official task statement                                                                                      | Priority for revision |
| --- | ------------------------------------------------------------------------------------------------------------ | --------------------- |
| 1.1 | Translate business problems into Claude-based AI solutions                                                   | ⭐ High                |
| 1.2 | Design end-to-end architectures (input → processing → output → feedback loops)                               | ⭐ High                |
| 1.3 | Select appropriate architectural patterns (workflow, agentic, augmented LLM)                                 | ⭐ Critical            |
| 1.4 | Design multi-agent systems and orchestration strategies                                                      | ⭐ Critical            |
| 1.5 | Apply decomposition techniques for complex problem solving                                                   | ⭐ High                |
| 1.6 | Align solutions to business value pillars (efficiency, transformation, productivity, cost, performance SLAs) | ⭐ High                |

The priority labels are study recommendations, not official per-task exam weights.

# **Part 1: Per Sub-Domain Breakdown**

## **1.1 Translate Business Problems into Claude-Based AI Solutions**

### **Core Concept**

Translate an ambiguous business requirement into a **measurable, technically feasible, and appropriately governed AI solution**.

The architect must determine:

- Whether the task actually requires an LLM.
- Which activities should use Claude versus deterministic software.
- What information, integrations, and human approvals are required.
- Whether the expected benefit justifies the operational cost.
- How success will be measured against the existing process.

**The solution should be driven by business requirements, not model capabilities.**

### **Decision Criteria**

| Business requirement                              | Preferred architectural decision             |
| ------------------------------------------------- | -------------------------------------------- |
| Fixed calculations and predictable business rules | Deterministic code or rules engine           |
| Semantic extraction from unstructured documents   | Claude with structured output and validation |
| Questions requiring enterprise knowledge          | Retrieval-augmented Claude                   |
| Known multistep business process                  | Predefined workflow                          |
| Unpredictable investigation requiring tools       | Agentic system                               |
| High-stakes or irreversible decisions             | Claude assistance with human approval        |
| Large volumes of independent, nonurgent tasks     | Asynchronous batch processing                |

**Example: Contract review automation**

Business request: "Reduce contract review turnaround from three days to six hours."

A suitable architecture could automate classification, clause extraction, deviation identification, and drafting of review summaries, while retaining human approval for consequential legal decisions.

### **Canonical Skeleton**

```
Business Problem
      |
      v
Define Business Objective and Baseline
      |
      v
Identify Tasks and Decision Points
      |
      v
Classify Tasks
  |          |          |
Rules       Claude     Human
  |          |          |
  +----------+----------+
             |
             v
Define Quality, Risk, Cost and SLA Constraints
             |
             v
Select Minimal Viable Architecture
             |
             v
Build Evaluation Dataset and Pilot
             |
             v
Measure Business Outcomes
             |
             v
Deploy / Improve / Reject
```

### **Best Practices**

1. ⭐ Define explicit, measurable success criteria before selecting a model or developing prompts.
2. Start with a simple Claude call; introduce retrieval, tools, workflows, or agents only when necessary.
3. Keep deterministic calculations, access enforcement, and reliable business rules in conventional software.
4. Place human review where errors could cause significant harm or irreversible consequences.
5. Evaluate against representative real-world cases, including failures and edge conditions.

Anthropic guidance: [Building effective agents](https://www.anthropic.com/research/building-effective-agents?utm_source=chatgpt.com) and [Define success criteria and build evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests?utm_source=chatgpt.com).

### **Exam Traps**

| Trap                                            | Why it is tempting                                        | Why it is wrong                                                               |
| ----------------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Use Claude for every business-process step      | Creates an apparently unified AI solution                 | Deterministic tasks may be cheaper, faster, and more reliable in code         |
| Select the most capable model first             | Assumes model capability is the primary design constraint | Business requirements, risk, workload, and SLA must determine model selection |
| Fully automate all decisions                    | Maximizes apparent efficiency                             | High-stakes decisions may require accountable human oversight                 |
| Optimize prompt quality before defining success | Prompt improvements are easy to demonstrate               | A technically impressive response may deliver no measurable business value    |

**⭐ Exam heuristic:** When a scenario begins with a vague business request, choose the option that clarifies measurable outcomes, task boundaries, and constraints before committing to an architecture.

## **1.2 Design End-to-End Architectures (Input → Processing → Output → Feedback Loops)**

### **Core Concept**

An end-to-end Claude architecture includes more than an LLM endpoint. It encompasses the complete lifecycle of a request, from ingestion to validated output, operational monitoring, and iterative improvement.

A production architecture must consider:

- Input ingestion and validation.
- Authentication, authorization, and data access controls.
- Retrieval and context preparation.
- Model inference and tool execution.
- Output validation and downstream integration.
- Feedback, evaluation, observability, and recovery.

**Architectural quality is determined by the reliability of the entire pipeline, not solely by model accuracy.**

### **Canonical Skeleton**

```
                  USER / ENTERPRISE SYSTEM
                           |
                           v
                API Gateway / Authentication
                           |
                           v
                 Input Validation / Policy
                           |
                           v
                  Context Preparation
                  /                 \
         Retrieval / RAG       Approved Tools
                  \                 /
                   v               v
                     Claude Model
                           |
                           v
                  Output Validation
                  /                \
             Valid                Invalid
               |                     |
               v                     v
        Business Workflow       Retry / Repair /
               |                 Human Escalation
               v
       Human Approval (if needed)
               |
               v
        Final Output / Action
               |
               v
        Feedback and Monitoring
               |
               v
        Evaluation and Iteration
```

Security, authorization checks, tracing, and cost monitoring should apply across the relevant stages rather than only at the beginning or end.

### **Architecture Layer Responsibilities**

| Layer     | Responsibility                       | Typical controls                                |
| --------- | ------------------------------------ | ----------------------------------------------- |
| Input     | Accept and normalize inputs          | Schema validation, size limits, input filtering |
| Context   | Supply relevant enterprise knowledge | Access-controlled retrieval, freshness checks   |
| Reasoning | Perform semantic analysis            | Model selection, prompts, token budgets         |
| Tools     | Access external systems              | Least privilege, allowlists, timeouts           |
| Output    | Produce consumable results           | Schema validation, evidence verification        |
| Execution | Trigger downstream actions           | Authorization, idempotency, approval            |
| Feedback  | Measure and improve results          | Evals, logs, traces, human corrections          |

### **Best Practices**

1. ⭐ Separate model reasoning from deterministic application logic and authorization enforcement.
2. Give Claude only the relevant, authorized context required for the task.
3. Validate tool arguments and outputs using application code; treat retrieved content and tool responses as untrusted.
4. Capture end-to-end traces, including retrieval results, tool calls, errors, latency, tokens, and costs, subject to privacy controls.
5. Design explicit retry, fallback, timeout, and escalation paths.

A Claude `tool_use` request is not itself a completed external action. For client tools, the application executes the requested operation and returns a corresponding `tool_result`.

Reference: [Tool use with Claude](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview?utm_source=chatgpt.com).

### **Exam Traps**

| Trap                                 | Why tempting                                    | Why wrong                                                                                 |
| ------------------------------------ | ----------------------------------------------- | ----------------------------------------------------------------------------------------- |
| RAG guarantees factual correctness   | Relevant documents appear to ground the answer  | Retrieval can return stale or irrelevant content, and the model may still misinterpret it |
| Prompt instructions enforce security | System prompts can describe access restrictions | Authorization must also be enforced outside the model                                     |
| Logging final outputs is sufficient  | User-visible errors are easy to identify        | Root-cause analysis often requires intermediate retrieval and tool traces                 |
| Retry every failed operation         | Retries often fix transient problems            | Retrying non-idempotent operations may duplicate side effects                             |

**⭐ Exam heuristic:** For production-architecture questions, prefer the answer with explicit input controls, intermediate validation, authorization, recovery, observability, and feedback—not merely a more capable model.

## **1.3 Select Appropriate Architectural Patterns (Workflow, Agentic, Augmented LLM)**

### **Core Concept**

This is one of the most important architectural distinctions in Domain 1.

Anthropic differentiates between:

- **Augmented LLM:** A Claude call enhanced with retrieval, tools, or memory.
- **Workflow:** LLM calls and tools follow predefined, application-controlled paths.
- **Agent:** Claude dynamically decides its next steps and tool usage to accomplish a goal.

An augmented LLM is a building block, not a mutually exclusive alternative to workflows and agents. Both workflows and agents may use retrieval, tools, and memory.

### **Canonical Skeleton: Architectural Selection**

```
Is a conventional deterministic solution sufficient?
                  |
           +------+------+
           |             |
          Yes            No
           |             |
        Use code     Can one Claude call
                     solve the task?
                           |
                     +-----+-----+
                     |           |
                    Yes          No
                     |           |
               Augmented LLM   Are steps known?
                                  |
                            +-----+-----+
                            |           |
                           Yes          No
                            |           |
                     Fixed Workflow   Agentic Loop
                            |           |
                            +-----+-----+
                                  |
                       Need independent exploration
                       beyond a single agent?
                                  |
                            +-----+-----+
                            |           |
                           No          Yes
                            |           |
                      Keep simple   Consider
                                    Multi-Agent
```

### **Pattern Selection Matrix**

| Pattern              | When to use                                               | Main benefit                   | Main cost or limitation              |
| -------------------- | --------------------------------------------------------- | ------------------------------ | ------------------------------------ |
| Single Claude call   | Simple, bounded task                                      | Low orchestration overhead     | Limited decomposition                |
| Augmented LLM        | Knowledge-intensive task requiring context or tools       | Capability extension           | Retrieval/tool failure risks         |
| Prompt chaining      | Known sequential stages                                   | Better stage-level control     | Increased latency                    |
| Routing              | Clearly different task categories                         | Specialized prompts and models | Classification errors                |
| Parallelization      | Independent subtasks                                      | Reduced wall-clock time        | Additional concurrent resource usage |
| Orchestrator-workers | Dynamic subtasks determined by input                      | Flexible delegation            | Coordination and token overhead      |
| Evaluator-optimizer  | Output can be iteratively improved against clear criteria | Higher quality                 | Additional calls and possible loops  |
| Autonomous agent     | Unknown sequence of actions                               | Adaptive problem-solving       | Less predictable cost and behavior   |

### **Canonical Workflow Patterns**

**1. Prompt chaining**

```
Input -> Step A -> Validate -> Step B -> Validate -> Output
```

Use when intermediate results must be available before the next operation.

**2. Routing**

```
Input -> Classifier -> Specialist A / B / C -> Output
```

Use when categories need materially different handling.

**3. Parallelization**

```
                -> Worker A -
Input -> Split  -> Worker B -> Aggregate -> Output
                -> Worker C -
```

Use for independent subtasks or independent perspectives.

**4. Orchestrator-workers**

```
Input -> Orchestrator -> Dynamic Worker Creation
                              |
                         Worker Results
                              |
                         Synthesis -> Output
```

Use when the work cannot be fully decomposed beforehand.

**5. Evaluator-optimizer**

```
Generate -> Evaluate -> Pass?
                        |
                  No -> Revise
                        |
                  Yes -> Output
```

Use when clear grading criteria support useful iteration.

**6. Agentic loop**

```
Goal -> Claude -> Select Tool -> Execute
           ^                      |
           |------ Observation ---|
           |
       Stop when goal met
       or budget exhausted
```

### **Decision Trade-Offs**

| Condition                                      | Prefer                         |
| ---------------------------------------------- | ------------------------------ |
| Strict latency and predictable processing      | Single call or simple workflow |
| Independent analysis tasks                     | Parallelization                |
| Highly variable tool exploration               | Agent                          |
| Distinct request categories                    | Routing                        |
| Verifiable intermediate stages                 | Prompt chaining with gates     |
| Repeated refinement against an explicit rubric | Evaluator-optimizer            |
| Multiple independent research directions       | Orchestrator-workers           |

### **Best Practices**

1. ⭐ Begin with the simplest composable pattern that achieves the required outcome.
2. Distinguish **application-controlled workflow** from **model-controlled agent behavior**.
3. Use agents when flexibility is essential, not merely because tools are involved.
4. Test whether additional stages or agents improve task success enough to justify latency and cost.
5. Define termination criteria for iterative workflows and agents.

These patterns follow Anthropic's December 2024 architecture guidance: [Building effective agents](https://www.anthropic.com/research/building-effective-agents?utm_source=chatgpt.com).

### **Exam Traps**

| Trap                                        | Why tempting                                    | Why wrong                                                                                |
| ------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Every tool-using system is an agent         | Tool use looks agentic                          | A fixed workflow can call tools without delegating control flow to Claude                |
| Multi-agent is always superior              | More agents appear to provide more intelligence | Coordination, token usage, and failure surfaces can increase                             |
| RAG is itself an autonomous agent           | Retrieval extends model capabilities            | Static retrieval does not necessarily involve dynamic planning                           |
| Routing and orchestration are equivalent    | Both assign tasks to specialized processors     | Routing selects among predefined branches; orchestration may dynamically create subtasks |
| Evaluator-optimizer always improves outputs | Revisions sound inherently valuable             | Weak grading criteria can produce unnecessary or harmful iterations                      |

**⭐ Exam heuristic:** Known steps → workflow. Unknown steps → agent. Independent work → parallelize. Dynamic decomposition → orchestrator-workers. Clear iterative grading → evaluator-optimizer.

## **1.4 Design Multi-Agent Systems and Orchestration Strategies**

### **Core Concept**

Multi-agent systems employ multiple Claude instances or agent processes with distinct objectives, tool access, or context.

The objective is not simply to increase the number of agents. It is to improve task performance by dividing complex work into **meaningfully independent or specialized activities**.

A central orchestrator may handle planning, task assignment, coordination, synthesis, and escalation.

Multi-agent architectures are most useful when:

- A problem requires broad exploration.
- Independent research directions can execute concurrently.
- Specialists require different tools or instructions.
- A single context window becomes operationally inefficient.
- The quality improvement justifies the extra compute and coordination.

### **Canonical Skeleton: Orchestrator–Worker Architecture**

```
                     User Request
                          |
                          v
                    Lead Agent
                    /        \
             Task Planning   Budget and Policy
                    |
          +---------+---------+
          |         |         |
          v         v         v
      Worker A   Worker B   Worker C
      Legal      Finance    Technical
          |         |         |
       Tools A   Tools B   Tools C
          |         |         |
          +---------+---------+
                    |
                    v
             Evidence Aggregation
                    |
                    v
               Lead Agent
                    |
                    v
            Verification / Review
                    |
                    v
                Final Result
```

### **Orchestration Strategy Comparison**

| Strategy                   | Control mechanism                               | Best fit                     |
| -------------------------- | ----------------------------------------------- | ---------------------------- |
| Sequential delegation      | Each worker completes before the next begins    | Dependent stages             |
| Parallel delegation        | Workers execute independently                   | Research breadth and speed   |
| Hierarchical orchestration | Lead agent delegates to specialized workers     | Complex, dynamic tasks       |
| Fixed specialist workflow  | Application code invokes predefined specialists | Predictable processes        |
| Evaluator-worker loop      | Critic reviews and requests revisions           | Quality-sensitive generation |

### **Agent Contract**

Every delegated task should specify:

```
objective: "Identify material financial risks"
scope: "Financial statements for the specified period"
tools: ["approved_financial_search"]
inputs: ["company_id", "reporting_period"]
outputs:
  findings: "structured list"
  evidence: "source references"
  unresolved_questions: "list"
constraints:
  max_steps: "configured workload limit"
  max_budget: "configured cost limit"
  write_access: false
completion: "Return findings and uncertainties to orchestrator"
```

The contract is an illustrative design convention, not a mandatory Anthropic API schema.

### **Best Practices**

1. ⭐ Delegate bounded objectives with clear input/output contracts, evidence requirements, and task boundaries.
2. Give each worker only the tools and context it needs.
3. Parallelize independent exploration but preserve sequential dependencies.
4. Apply budgets, timeouts, concurrency limits, and stopping conditions.
5. Have the orchestrator verify, reconcile, and synthesize worker results instead of merely concatenating them.

Anthropic's multi-agent research architecture uses a lead agent, parallel subagents, and a final synthesis process. Its published internal experiment showed substantial gains on broad research tasks, but those results are **not universal performance guarantees**.

Reference: [How we built our multi-agent research system](https://www.anthropic.com/research/multi-agent-research-system?utm_source=chatgpt.com).

### **Exam Traps**

| Trap                                                   | Why tempting                         | Why wrong                                                                    |
| ------------------------------------------------------ | ------------------------------------ | ---------------------------------------------------------------------------- |
| Spawn as many agents as possible                       | Maximizes apparent parallelism       | Duplicated work and coordination overhead can overwhelm the benefit          |
| Give all workers full conversation history             | Appears to preserve shared knowledge | Creates unnecessary context growth and weakens isolation                     |
| Let every agent invoke production write tools          | Provides maximum autonomy            | Expands risk and makes side effects difficult to control                     |
| Assume parallel agents automatically reduce total cost | Parallel completion is faster        | Total token usage and tool costs may rise significantly                      |
| Combine worker answers without verification            | Minimizes orchestration logic        | Results can conflict, duplicate evidence, or contain unsupported conclusions |

**⭐ Exam heuristic:** Select multi-agent architecture when tasks are sufficiently independent, the problem benefits from specialized context or tools, and the value of improved performance exceeds coordination costs.

## **1.5 Apply Decomposition Techniques for Complex Problem Solving**

### **Core Concept**

Decomposition breaks a complex objective into smaller, verifiable units of work.

The primary design challenge is determining **how to split the work while preserving necessary dependencies and avoiding unnecessary communication**.

Decomposition is broader than multi-agent design. It can be implemented using ordinary code, sequential Claude calls, parallel workers, or dynamic agents.

### **Canonical Skeleton**

```
Complex Objective
       |
       v
Identify Required Outputs
       |
       v
Break into Subtasks
       |
       v
Map Dependencies
       |
       v
Choose Execution Strategy
       |
       +---- Dependent ------> Sequential
       |
       +---- Independent ----> Parallel
       |
       +---- Unpredictable --> Dynamic Agent
       |
       v
Define Intermediate Output Contracts
       |
       v
Execute and Validate
       |
       v
Aggregate Results
       |
       v
Final Verification
```

### **Decomposition Methods**

| Technique                  | Description                                 | Best use                                 |
| -------------------------- | ------------------------------------------- | ---------------------------------------- |
| Sequential decomposition   | Output of one stage feeds the next          | Extraction → validation → synthesis      |
| Functional decomposition   | Split by distinct capabilities              | Legal, technical, and financial analysis |
| Data decomposition         | Partition independent documents or records  | Large document collections               |
| Hierarchical decomposition | Break major tasks into subtasks recursively | Complex research and planning            |
| Dynamic decomposition      | Model decides subtasks at runtime           | Open-ended problem-solving               |

### **Example: Contract Due Diligence**

```
Contract Portfolio
        |
        v
Document Classification
        |
        v
Parallel Clause Extraction
  |          |          |
Liability   Payment   Termination
  |          |          |
  +----------+----------+
             |
             v
     Normalize Findings
             |
             v
    Cross-Clause Analysis
             |
             v
    Validate Against Evidence
             |
             v
       Risk Summary
```

The clause extractions are parallelizable because they can often be evaluated independently. Cross-clause analysis must wait until the relevant extractions are available.

### **Best Practices**

1. ⭐ Use task dependency, not arbitrary task size, to determine sequencing and parallelization.
2. Define each subtask's input, output, completion criteria, and validation checks.
3. Minimize unnecessary context transfer between tasks.
4. Use specialized prompts and tools only where specialization improves outcomes.
5. Verify intermediate artifacts before using them in downstream reasoning.

Anthropic's context-engineering guidance favors high-signal context, bounded tools, and focused subagent work.

Reference: [Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents?utm_source=chatgpt.com).

### **Exam Traps**

| Trap                                                                | Why tempting                                  | Why wrong                                                                                               |
| ------------------------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Every complex problem needs multiple agents                         | Agents appear to be natural subtask owners    | A deterministic decomposition may be simpler and more reliable                                          |
| Run all subtasks concurrently                                       | Maximizes apparent throughput                 | Dependent stages need upstream results                                                                  |
| Ask Claude to reason step by step instead of decomposing the system | Appears to solve complexity through prompting | Prompt-level reasoning does not provide independent validation, state management, or execution controls |
| Pass all intermediate outputs to every stage                        | Avoids losing information                     | Excess context increases cost and can reduce relevance                                                  |
| Skip intermediate validation                                        | Reduces latency                               | Upstream errors can propagate and compound                                                              |

**⭐ Exam heuristic:** Determine the dependency graph first. Parallelize independent work; sequence dependent work; use dynamic planning only where task structure cannot be predetermined.

## **1.6 Align Solutions to Business Value Pillars (Efficiency, Transformation, Productivity, Cost, Performance SLAs)**

### **Core Concept**

AI architecture must justify its complexity through measurable business outcomes.

Technical performance is necessary but insufficient. A solution can achieve high benchmark accuracy yet fail commercially because of latency, operating costs, poor adoption, or excessive human rework.

An architect must connect design decisions to the business-value pillars defined in the task statement.

### **Business Value Matrix**

| Pillar           | Architectural objective                                     | Useful metrics                                   |
| ---------------- | ----------------------------------------------------------- | ------------------------------------------------ |
| Efficiency       | Reduce process time and unnecessary steps                   | Turnaround time, automation rate                 |
| Transformation   | Enable new business capabilities                            | New workflow coverage, adoption, task completion |
| Productivity     | Increase effective output per worker                        | Accepted cases per hour, rework rate             |
| Cost             | Reduce total cost of achieving an accepted outcome          | Cost per successful task, cost per document      |
| Performance SLAs | Meet production reliability and responsiveness requirements | p50/p95/p99 latency, availability, error rate    |

### **Canonical Skeleton**

```
Business Objective
       |
       v
Measure Existing Baseline
       |
       v
Define Business KPIs and Guardrails
       |
       v
Identify Feasible Architectures
       |
       v
Estimate Quality / Latency / Cost / Risk
       |
       v
Benchmark Using Representative Workloads
       |
       v
Evaluate Trade-Offs
       |
       v
Select Architecture
       |
       v
Pilot and Measure Real Outcomes
       |
       v
Optimize / Scale / Reconsider
```

### **Cost–Quality–Latency Trade-Offs**

| Design decision        | Potential benefit                                   | Trade-off                                               |
| ---------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| Smaller suitable model | Lower cost and often lower latency                  | Possible quality reduction                              |
| More capable model     | Better performance on difficult tasks               | Potentially higher cost or latency                      |
| Prompt caching         | Reduced repeated-prefix processing cost and latency | Requires reusable prefixes and suitable cache lifetime  |
| Message Batches API    | Lower cost for asynchronous workloads               | Not suitable for immediate interactive responses        |
| Parallel processing    | Reduced elapsed time for independent work           | Higher concurrency and possibly higher spend            |
| Multi-agent design     | Better exploration of difficult tasks               | Additional calls, coordination, and failure modes       |
| Extended thinking      | May improve difficult reasoning                     | Additional reasoning budget and possible latency impact |
| Human review           | Reduced risk of consequential mistakes              | Review cost and longer turnaround time                  |

### **Essential Financial and Operational Metrics**

**Cost per accepted outcome**

\\[ \text{Cost per accepted outcome} = \frac{\text{Total processing + infrastructure + review cost}} {\text{Number of accepted outcomes}} \\]

**Task success rate**

\\[ \text{Task success rate} = \frac{\text{Successfully completed tasks}} {\text{Attempted tasks}} \\]

**Automation rate**

\\[ \text{Automation rate} = \frac{\text{Tasks completed without human intervention}} {\text{Eligible tasks}} \\]

**Approximate net benefit**

\\[ \text{Net benefit} = \text{Value of realized savings and benefits} - \text{Total solution cost} \\]

Track quality and risk guardrails alongside these metrics. A high automation rate is not desirable if it comes with unacceptable errors.

### **Evaluation Strategy**

| Evaluation type          | Best use                                         | Trade-off                                                  |
| ------------------------ | ------------------------------------------------ | ---------------------------------------------------------- |
| Deterministic/code-based | JSON validity, exact labels, business rules      | Fast and reproducible, but limited semantic nuance         |
| LLM-as-judge             | Relevance, completeness, qualitative comparisons | Scalable but requires rubric calibration                   |
| Human expert evaluation  | Legal, business, and safety-sensitive judgments  | More expensive but necessary for some decisions            |
| A/B testing              | Comparing production-ready alternatives          | Requires controlled conditions and sufficient observations |

### **Best Practices**

1. ⭐ Optimize for **cost per successful business outcome**, not merely cost per API call.
2. Use realistic workload evaluations to compare architectures and model configurations.
3. Measure tail latency, especially p95 and p99, where user experience or SLAs depend on it.
4. Apply prompt caching to stable, repeatedly reused context and batching to eligible asynchronous workloads.
5. Include failure handling, human intervention, and infrastructure expenditure in total cost estimates.

References: [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching?utm_source=chatgpt.com), [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing?utm_source=chatgpt.com), and [Evaluation guidance](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests?utm_source=chatgpt.com).

### **Exam Traps**

| Trap                                               | Why tempting                  | Why wrong                                                              |
| -------------------------------------------------- | ----------------------------- | ---------------------------------------------------------------------- |
| Lowest token cost is the best architecture         | Easy to compare financially   | Poor quality may cause retries and human rework                        |
| Average latency proves an SLA is met               | Simple aggregate metric       | Tail latency may violate customer requirements                         |
| More accuracy always justifies extra cost          | Prioritizes model performance | Marginal quality improvements may not justify business cost            |
| Batch processing should be used for every workload | Offers significant discounts  | Asynchronous execution is unsuitable for strict real-time requirements |
| Human review should always be removed              | Increases apparent automation | Review may be necessary for risk, legal, or operational accountability |

**⭐ Exam heuristic:** Choose the architecture that meets all mandatory quality, security, and latency requirements at the lowest justified total cost. Do not optimize a single metric in isolation.

# **Part 2: Consolidated Sections**

## **Key Facts to Memorize**

### **A. Architecture Definitions**

| Term                 | Definition or rule to remember                             |
| -------------------- | ---------------------------------------------------------- |
| ⭐ Augmented LLM      | Claude enhanced with retrieval, tools, or memory           |
| ⭐ Workflow           | Predefined control flow managed by application code        |
| ⭐ Agent              | Model dynamically determines actions and tool usage        |
| Prompt chaining      | Sequential dependent LLM tasks                             |
| Routing              | Classify input and select an appropriate predefined branch |
| Parallelization      | Concurrent execution of independent tasks                  |
| Orchestrator-workers | Dynamically delegated subtasks and centralized synthesis   |
| Evaluator-optimizer  | Generate, evaluate, and iteratively refine                 |
| RAG                  | Retrieve external information to ground generation         |
| Human-in-the-loop    | Human participation or approval at defined decision points |

### **B. Pattern Decision Rules**

| Exam scenario                                    | Best starting choice                                 |
| ------------------------------------------------ | ---------------------------------------------------- |
| Known deterministic logic                        | Conventional software                                |
| Simple unstructured extraction                   | Single Claude call                                   |
| Questions over internal enterprise documents     | Augmented LLM with retrieval                         |
| Predictable, ordered processing                  | Prompt chaining                                      |
| Different predefined categories                  | Routing                                              |
| Independent analysis                             | Parallelization                                      |
| Unpredictable, dynamic task sequence             | Agent                                                |
| Dynamic independent subtasks                     | Orchestrator-workers                                 |
| Repeated improvement with clear scoring criteria | Evaluator-optimizer                                  |
| Irreversible or consequential action             | Approval and authorization controls before execution |

**⭐ Always remember:** These are starting choices, not absolute rules. The correct architecture depends on demonstrated requirements and evaluations.

### **C. Claude API and Optimization Features**

| Feature                              | High-yield fact as of July 2026                                                          |
| ------------------------------------ | ---------------------------------------------------------------------------------------- |
| Messages API                         | Primary API interface for Claude message-based interactions                              |
| `tool_use`                           | Claude's request to invoke a tool                                                        |
| `tool_result`                        | Result supplied back from tool execution                                                 |
| Client tools                         | Application is responsible for execution and security enforcement                        |
| MCP                                  | Standardized mechanism for connecting models to tools and external systems               |
| Prompt caching                       | Reuses eligible repeated prompt prefixes                                                 |
| Default prompt-cache TTL             | 5 minutes                                                                                |
| Extended prompt-cache TTL            | 1 hour where supported                                                                   |
| 5-minute cache write                 | Typically 1.25× base input-token price                                                   |
| 1-hour cache write                   | Typically 2× base input-token price                                                      |
| Cache read                           | Typically 0.1× base input-token price                                                    |
| Message Batches API                  | Asynchronous processing at approximately 50% of standard API token pricing               |
| Batch processing window              | Up to 24 hours; not an interactive latency guarantee                                     |
| Extended thinking                    | Additional reasoning capability with budget, cost, and latency implications              |
| `max_tokens`                         | Configures the maximum generated output tokens, subject to model and API limits          |
| Structured outputs / JSON validation | Useful for integration contracts; structural validity does not prove factual correctness |

**Version caution:** Model availability, maximum context windows, output-token ceilings, supported thinking configurations, cacheable minimum lengths, rate limits, and deployment-platform support vary. Do not memorize a single universal limit for all models or deployments.

### **D. Evaluation and SLA Metrics**

| Metric                   | What it measures                            | Typical use                       |
| ------------------------ | ------------------------------------------- | --------------------------------- |
| Accuracy                 | Proportion of correct predictions           | Classification                    |
| Precision                | Correct positives among predicted positives | False-positive-sensitive tasks    |
| Recall                   | Correct positives among actual positives    | Miss-sensitive extraction         |
| F1                       | Harmonic mean of precision and recall       | Extraction and classification     |
| Task success rate        | Completed goals / attempted goals           | Agent evaluation                  |
| Groundedness             | Whether claims are supported by evidence    | RAG                               |
| Citation correctness     | Whether references substantiate claims      | Research workflows                |
| p50 latency              | Median request latency                      | Typical responsiveness            |
| p95 latency              | 95th-percentile latency                     | SLA and tail-performance analysis |
| p99 latency              | 99th-percentile latency                     | Severe tail-performance analysis  |
| Availability             | Successful service availability over time   | Reliability SLAs                  |
| Cost per successful task | Cost normalized by useful outcomes          | Economic efficiency               |
| Tool error rate          | Failed tool invocations / total invocations | Integration reliability           |
| Human escalation rate    | Requests requiring human intervention       | Automation effectiveness          |

**Threshold rule:** There is no universal architectural requirement that every Claude solution achieve a particular F1, p95, cost, or success-rate threshold. These targets must be derived from business and risk requirements.

### **E. Non-Negotiable Architectural Rules**

1. ⭐ **Start simple; justify complexity with evaluations.**
2. Never treat model-generated tool arguments as independently authorized operations.
3. Never assume a retrieved document or tool response is trustworthy simply because it appears in context.
4. Do not rely solely on prompts to enforce access control.
5. Parallelize only when dependencies permit.
6. Apply stopping conditions and resource budgets to iterative agents.
7. Validate intermediate outputs when downstream steps depend on them.
8. Choose grading methods based on the required fidelity: code-based where objective, calibrated LLM grading where nuanced, and human experts where necessary.
9. Treat human review cost and rework as part of the solution's total cost.
10. Evaluate end-to-end task success rather than assuming improved individual model responses guarantee business success.

# **5 Exam-Style Questions**

These are original scenario-based practice questions, not questions from the live certification exam. Each has one best answer.

## **Question 1 — Business Problem Translation**

**Task statement: 1.1**

A legal-services organization wants to automate contract review. Its current review process takes approximately three days per contract. Some contracts contain unusual clauses that require legal interpretation. Leadership requests a fully autonomous Claude solution to minimize human involvement.

Which architecture should the solutions architect recommend FIRST?

**A.** Deploy a multi-agent architecture where separate agents negotiate, interpret, and approve every contract.

**B.** Deploy the most capable Claude model available and allow it to make all contractual decisions without human review.

**C.** Identify automatable tasks, define success criteria, use Claude for extraction and analysis, retain appropriate human approval, and measure results against the existing process.

**D.** Fine-tune a model on historical contracts before establishing evaluation criteria or workflow requirements.

**Correct answer: C**

**Why C is correct:** It begins with task decomposition, measurable success, and risk-aware allocation of work between Claude and humans. It allows the architecture to be validated against real business outcomes before increasing autonomy.

**Why the others are wrong:**

- **A:** Introduces complex orchestration and autonomous authority without evidence of need.
- **B:** Confuses stronger model capability with permission to make consequential decisions.
- **D:** Assumes model customization is necessary before identifying failure modes, evaluation criteria, or simpler alternatives.

⭐ **Exam takeaway:** Business objectives and operational accountability come before model or agent selection.

## **Question 2 — Architectural Pattern Selection**

**Task statements: 1.2 and 1.3**

A financial-services company must process thousands of standardized loan application documents. The required process is fixed: extract information, validate fields against known rules, generate an application summary, and send flagged cases to a human reviewer.

Management prioritizes predictability, traceability, and operational consistency.

Which architecture is BEST?

**A.** An autonomous agent that dynamically determines the steps and tools for each application.

**B.** A predefined workflow with Claude-based extraction and summarization, deterministic validation gates, and human escalation.

**C.** A large orchestrator-workers system with independently planning subagents for each application.

**D.** A single Claude prompt that extracts, validates, and approves applications without intermediate checks.

**Correct answer: B**

**Why B is correct:** The process steps are predictable. A predefined workflow enables traceability, controlled execution, deterministic field validation, and risk-based review.

**Why the others are wrong:**

- **A:** Unnecessary autonomous planning increases unpredictability for a fixed process.
- **C:** Adds delegation and coordination overhead without clear benefit.
- **D:** Combines multiple responsibilities without adequate validation or approval controls.

⭐ **Exam takeaway:** A fixed business process generally favors a workflow over autonomous orchestration.

## **Question 3 — Multi-Agent Orchestration**

**Task statement: 1.4**

An enterprise intelligence platform must produce an extensive market-entry report covering financial performance, competitors, regulations, and technology trends.

The research directions are largely independent, the appropriate sources cannot be predicted beforehand, and information gathering requires substantial tool usage.

What is the BEST architecture?

**A.** One large prompt containing every available enterprise document.

**B.** A sequential workflow in which one model examines every topic in a predetermined order.

**C.** A lead Claude agent that dynamically delegates bounded research tasks to parallel specialist agents and synthesizes evidence-backed results.

**D.** Four independent specialist agents that publish their findings directly, without shared synthesis or reconciliation.

**Correct answer: C**

**Why C is correct:** It combines dynamic planning, specialization, independent exploration, and centralized synthesis. Subagents can operate with focused contexts while the lead agent coordinates the overall research objective.

**Why the others are wrong:**

- **A:** Produces unnecessary context volume and cannot adaptively investigate missing information.
- **B:** May work but loses opportunities for parallel exploration and flexible delegation.
- **D:** Lacks a mechanism to resolve contradictions, detect gaps, and produce a coherent final result.

⭐ **Exam takeaway:** Complex, open-ended, parallelizable research is a strong candidate for orchestrator-workers.

## **Question 4 — Decomposition and Dependencies**

**Task statement: 1.5**

An AI-powered due-diligence workflow performs document classification, clause extraction, cross-document risk analysis, and report generation.

The architect wants to reduce total processing time while retaining verifiable intermediate outputs.

Which decomposition strategy is BEST?

**A.** Execute every stage simultaneously, even when later stages require earlier outputs.

**B.** Use independent parallel workers for clause extraction, then perform dependent risk analysis and report generation sequentially, with validation between stages.

**C.** Put all documents in a single prompt and ask Claude to complete the entire process without intermediate artifacts.

**D.** Create separate autonomous agents for every processing step regardless of task complexity.

**Correct answer: B**

**Why B is correct:** It respects the dependency graph while exploiting safe parallelism. Intermediate validation reduces the risk of errors propagating into downstream analysis.

**Why the others are wrong:**

- **A:** Violates data dependencies and can cause incomplete or inconsistent outputs.
- **C:** Reduces observability and opportunities to detect intermediate failures.
- **D:** Adds autonomy without demonstrating that dynamic control is needed.

⭐ **Exam takeaway:** Parallelize independent work, sequence dependent operations, and validate stage boundaries.

## **Question 5 — Business Value and Performance SLAs**

**Task statement: 1.6**

A customer-support platform must respond to users within a defined p95 latency target and maintain an agreed minimum quality score.

Three architectures have been evaluated:

| Architecture           | Quality | p95 latency | Cost per successful task |
| ---------------------- | ------- | ----------- | ------------------------ |
| Single Claude workflow | 94%     | 2.1 seconds | $0.04                    |
| Multi-agent workflow   | 97%     | 8.3 seconds | $0.15                    |
| Smaller-model workflow | 88%     | 1.2 seconds | $0.02                    |

The business requires at least **93% quality** and **p95 latency below 4 seconds**.

Which option is BEST?

**A.** Multi-agent workflow, because it has the highest quality.

**B.** Smaller-model workflow, because it has the lowest latency and cost.

**C.** Single Claude workflow, because it satisfies both mandatory constraints at the lowest cost among qualifying options.

**D.** Multi-agent workflow using batch processing, because batching guarantees faster interactive responses.

**Correct answer: C**

**Why C is correct:** It meets both binding constraints. The multi-agent architecture violates latency requirements, and the smaller-model workflow fails the quality requirement.

**Why the others are wrong:**

- **A:** Quality alone does not override a mandatory SLA.
- **B:** Cost and speed cannot compensate for failure to meet the minimum quality threshold.
- **D:** Batch processing is asynchronous and does not guarantee low interactive latency.

⭐ **Exam takeaway:** Identify binding constraints first, eliminate infeasible designs, and then optimize cost and business value among the feasible alternatives.

# **Last-Minute Revision Checklist**

- ⭐ Distinguish augmented LLM, workflow, agent, and multi-agent architectures.
- Choose deterministic code when an LLM is unnecessary.
- Memorize the five core workflow patterns: chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer.
- Know when parallel delegation is appropriate and when dependencies require sequencing.
- Define clear responsibilities, context boundaries, output contracts, and stopping criteria for agents.
- Include validation, authorization, observability, feedback, and failure handling in production architectures.
- Remember prompt caching for repeated context and batch processing for nonurgent asynchronous workloads.
- Know the differences between task accuracy, task success rate, cost per successful task, and p95 latency.
- Apply human approval to consequential actions where required by risk and accountability.
- ⭐ Select the simplest architecture that meets every binding business, quality, risk, and SLA requirement.
