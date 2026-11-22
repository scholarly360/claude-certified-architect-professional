# **Claude Certified Architect – Professional (CCA-P)**

## **Domain 2: Claude Models, Prompting & Context Engineering**

**Exam weight:** 13%\
**Study scope:** July 31, 2026\
**Focus:** Model selection, prompt design, reasoning strategies, context optimization, and reusable prompt infrastructure.

⭐ **Central exam principle:** Choose the least complex, lowest-cost configuration that satisfies the application's measurable quality, latency, safety, and operational requirements.

The July 2026 examination blueprint contains five task statements. The sections below follow their exact wording.

# **Part 1: Per Sub-Domain Breakdown**

## **2.1 Select appropriate Claude models based on trade-offs**

### **Core Concept**

Model selection is an architectural optimization problem, not a ranking exercise.

The architect must balance:

| Dimension          | Architectural question                                 | Typical trade-off                                             |
| ------------------ | ------------------------------------------------------ | ------------------------------------------------------------- |
| Task quality       | Can the model meet acceptance criteria?                | Stronger models may cost more                                 |
| Latency            | Can requests meet response-time SLAs?                  | More reasoning can increase latency                           |
| Cost               | What is the total production cost?                     | Lower token price may not mean lower cost per successful task |
| Context capacity   | Can required input fit?                                | Larger windows may increase processing cost                   |
| Tool compatibility | Does the model support required tools and features?    | Support differs by model and platform                         |
| Reasoning effort   | How much reasoning is necessary?                       | Higher effort may improve quality but increase tokens         |
| Reliability        | How consistently does the model complete the workflow? | Cheap models may require more retries                         |

**⭐ Primary decision rule:** Select the cheapest model and configuration that meets every *binding requirement*, demonstrated through representative evaluations.

### **July 2026 model selection reference**

The following models were available by July 31, 2026. Pricing is standard Claude API pricing per million input/output tokens, not cloud-provider-specific pricing.

| Model             | Input / Output | Context window | Architectural fit                                                  |
| ----------------- | -------------- | -------------- | ------------------------------------------------------------------ |
| Claude Haiku 4.5  | $1 / $5        | 200K           | Classification, routing, simple extraction, high-volume operations |
| Claude Sonnet 4.6 | $3 / $15       | 1M             | Enterprise workflows, coding, analysis                             |
| Claude Sonnet 5   | $2 / $10       | 1M             | Strong general-purpose default candidate                           |
| Claude Opus 4.8   | $5 / $25       | 1M             | Complex reasoning, coding, long-running agents                     |
| Claude Opus 5     | $5 / $25       | 1M             | High-capability reasoning and agentic enterprise workloads         |

Sonnet 5 launched June 30, 2026, with introductory pricing. Opus 5 launched July 24, 2026. Specialized Fable 5 and restricted-access Mythos 5 also existed by July, but they are not necessary as default choices for ordinary enterprise tasks.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



**Important:** Capability tiers are starting hypotheses, not guaranteed accuracy rankings. Model availability, pricing, tool support, and output limits must be checked against the chosen deployment platform.

### **Canonical Skeleton**

```
Business Requirements
        |
        v
Define Binding Constraints
  - Minimum quality
  - P95 latency SLA
  - Cost ceiling
  - Context and tools
  - Security / deployment
        |
        v
Build Representative Eval Dataset
        |
        v
Benchmark Candidate Models
  Haiku / Sonnet / Opus
        |
        v
Test Prompt + Effort Configurations
        |
        v
Measure:
  Quality | P50/P95 | Cost | Failure rate
        |
        v
Eliminate Constraint Violations
        |
        v
Select Lowest-Cost Passing Configuration
        |
        v
Deploy → Monitor → Re-evaluate
```

**Example:** An enterprise processes 100,000 employee support requests daily. Haiku meets the intent-classification quality threshold on straightforward requests but struggles with ambiguous policy questions.

Best architecture:

- Route straightforward classification to Haiku.
- Route complex policy interpretation to Sonnet.
- Escalate rare, difficult cases to a more capable model or human reviewer.
- Evaluate routing errors and end-to-end costs, including retries.

This is **model routing** rather than forcing every request through one expensive model.

### **Best Practices**

- ⭐ **Evaluate before selecting:** Benchmark task accuracy, latency, token consumption, and safety using representative golden datasets.
- **Optimize for total cost:** Include reasoning tokens, repeated calls, retries, tool calls, and human-review costs.
- **Tune effort before changing models:** Where supported, a lower reasoning effort may preserve quality while reducing latency.
- **Use smaller models selectively:** Separate low-complexity operations from high-reasoning operations.
- **Validate deployment compatibility:** Features and availability can differ across Claude API, AWS, Azure, and Google Cloud.

Anthropic's model-selection guidance supports both efficiency-first and capability-first benchmarking, depending on the application's constraints.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Exam Traps**

| Trap                                                    | Why tempting                                       | Why wrong                                                                       |
| ------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------- |
| Always choose Opus because it is most capable.          | Maximizes apparent model intelligence.             | Ignores cost, latency, and whether simpler models already satisfy requirements. |
| Always choose Haiku for high-volume applications.       | Minimizes nominal token price.                     | Retries, misclassifications, and escalations may increase total costs.          |
| Select a model solely on benchmark rankings.            | Benchmarks appear objective.                       | Generic benchmarks may not represent enterprise-specific failure modes.         |
| Choose a larger context window to solve poor retrieval. | More context appears to solve missing information. | Irrelevant or stale content still produces poor responses.                      |

**Exam heuristic:** When choices include *upgrade model immediately*, *reduce requirements*, and *benchmark configurations against explicit acceptance criteria*, the evaluation-driven choice is generally strongest.

## **2.2 Design system prompts, templates, and guardrails**

### **Core Concept**

A production prompt is a structured behavioral specification rather than an informal instruction.

It should define:

- **Role:** What Claude is responsible for.
- **Task:** What Claude must accomplish.
- **Constraints:** What is permitted and prohibited.
- **Context:** What information Claude can use.
- **Output contract:** Required format and validation rules.
- **Failure behavior:** What to do when information is insufficient or a request is unsafe.

A **system prompt** establishes stable behavioral instructions through the top-level `system` parameter.

A **prompt template** separates reusable instructions from dynamic variables.

A **guardrail** is a control that restricts undesirable behavior. Some guardrails operate through prompts, while stronger controls operate through application logic, permissions, validation, and human approvals.

⭐ **Critical distinction:** Prompt instructions are behavioral controls, not security boundaries.

### **Canonical Skeleton**

```
SYSTEM LAYER
  Role and responsibilities
  Grounding rules
  Behavioral constraints
  Error/uncertainty handling
          |
          v
REUSABLE PROMPT TEMPLATE
  Task instructions
  Input definitions
  Output contract
  Few-shot examples (if required)
          |
          v
DYNAMIC INPUT
  User query
  Retrieved documents
  Relevant application state
          |
          v
CLAUDE
          |
          v
POST-PROCESSING
  JSON/schema validation
  Policy validation
  Authorization checks
  Human approval (if required)
          |
          v
APPROVED OUTPUT / ACTION
```

#### **Canonical prompt template**

```
<role>
You are an enterprise contract-analysis assistant.
</role>

<instructions>
Extract the requested contractual information.

Use only the supplied contract.
Do not invent missing clauses.
Distinguish explicit contractual obligations
from interpretations.
</instructions>

<constraints>
If evidence is insufficient, return
"not_found" for that field.

Never execute an external action.
</constraints>

<contract>
{{CONTRACT_TEXT}}
</contract>

<task>
{{EXTRACTION_REQUEST}}
</task>

<output_requirements>
Return valid JSON containing:
- field_name
- extracted_value
- evidence
- confidence_label
</output_requirements>
```

**Architecture note:** The example illustrates logical prompt sections. In the actual Messages API, stable system instructions belong in the `system` parameter, while variable content is normally supplied through `messages`. XML tags improve separation but do not establish independent authorization levels.

### **Best Practices**

- ⭐ **Write explicit, testable instructions:** Specify positive behaviors, failure handling, output formats, and edge cases.
- **Use XML tags:** Separate instructions, source documents, examples, and variable inputs with consistent descriptive tags.
- **Keep templates modular:** Version reusable policies, templates, output contracts, and examples independently.
- **Prefer constrained output mechanisms when needed:** Use supported structured-output configurations or strict tool schemas and validate results in application code.
- **Enforce high-risk controls outside Claude:** Apply authorization, least-privilege tool exposure, and human approval to sensitive actions.

Anthropic recommends clear instructions, role definition, relevant context, and well-structured examples rather than relying on increasingly forceful wording.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Exam Traps**

| Trap                                                                           | Why tempting                                         | Why wrong                                                                        |
| ------------------------------------------------------------------------------ | ---------------------------------------------------- | -------------------------------------------------------------------------------- |
| Put every policy inside the system prompt and assume compliance is guaranteed. | System instructions have higher behavioral priority. | They are not substitutes for authorization, validation, or external enforcement. |
| Repeat "CRITICAL" or "NEVER" dozens of times.                                  | Strong language appears to increase compliance.      | Clarity, specificity, and empirical testing matter more than repetition.         |
| Use XML tags as a security boundary.                                           | Tags provide clear structural separation.            | They organize text; they do not make untrusted content trustworthy.              |
| Trust syntactically valid JSON as verified business data.                      | The response passes a parser.                        | Schema conformance does not prove semantic correctness or policy compliance.     |

**Exam heuristic:** If a scenario involves an irreversible action, sensitive data, or regulatory obligations, prefer deterministic application controls over prompt-only restrictions.

## **2.3 Apply prompt engineering techniques (zero-shot, few-shot, chain-of-thought)**

### **Core Concept**

Prompt engineering selects the simplest technique that reliably produces the required behavior.

The three techniques explicitly identified in this exam objective are:

| Technique                  | Definition                       | Best use case                                      | Trade-off                                              |
| -------------------------- | -------------------------------- | -------------------------------------------------- | ------------------------------------------------------ |
| **Zero-shot**              | Instructions without examples    | Simple, well-defined tasks                         | Lowest prompt overhead; may struggle with ambiguity    |
| **Few-shot**               | Instructions plus demonstrations | Classification, formatting, extraction, edge cases | Improves consistency but adds input tokens             |
| **Chain-of-thought (CoT)** | Encouraging multi-step reasoning | Complex logic, planning, analysis                  | Can improve reasoning but increases latency and tokens |

⭐ **High-yield distinction:** Prompted chain-of-thought and Claude's native extended/adaptive thinking are not equivalent.

- **Prompted CoT:** A natural-language instruction requesting a reasoning process or explanation.
- **Extended thinking:** A model/API capability enabling internal reasoning, with model-specific configuration.
- **Adaptive thinking:** Claude dynamically determines the amount of reasoning required, influenced by the supported `effort` setting.

By July 2026, adaptive thinking was the preferred approach on newer supporting models. Sonnet 5 and Opus 5 had thinking enabled by default. Manual `budget_tokens` remained usable but deprecated on Claude 4.6 models and was unsupported on Claude 4.7 and later.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Canonical Skeleton**

```
Define Task and Success Criteria
             |
             v
        Zero-Shot
             |
             v
     Evaluate Results
             |
       Quality met?
        /        \
      YES         NO
       |           |
       v           v
     Deploy   Diagnose Failure
                   |
           +-------+-------+
           |               |
     Format / Labels   Complex Reasoning
           |               |
           v               v
       Few-Shot       Native Thinking
           |           / Prompted CoT
           |               |
           +-------+-------+
                   |
                   v
          Re-run Evaluation
                   |
                   v
          Compare Cost, Quality,
               Latency
```

#### **Technique-selection examples**

**Zero-shot**

```
Classify the following request into one category:

- HR
- IT
- Finance
- Other

Request: "My salary reimbursement is delayed."

Return only the category.
```

**Few-shot**

```
<instructions>
Classify the request into HR, IT, Finance, or Other.
</instructions>

<examples>
  <example>
    Input: My laptop cannot connect to the VPN.
    Output: IT
  </example>

  <example>
    Input: My travel reimbursement is pending.
    Output: Finance
  </example>

  <example>
    Input: I want to update my leave balance.
    Output: HR
  </example>
</examples>

<input>
My expense claim was rejected.
</input>
```

**Reasoning-intensive task**

```
Analyze the contractual obligations and identify
conflicting clauses.

Consider the relevant dependencies and exceptions.
Ground the conclusion in the supplied text.

Return:
1. Conflicting clauses
2. Supporting evidence
3. Recommended interpretation
4. Unresolved ambiguity
```

For models with native thinking support, configure reasoning through the appropriate API mechanism rather than relying exclusively on instructions asking the model to "think step by step."

### **Best Practices**

- ⭐ **Start zero-shot:** Add examples or reasoning complexity only when evaluations show a specific need.
- **Use representative few-shot examples:** Include normal cases, difficult cases, and boundary conditions. Anthropic commonly recommends approximately 3–5 examples as a practical starting point, not a mandatory limit.
- **Structure examples consistently:** Use clear input/output pairs and descriptive XML tags.
- **Prefer native thinking for difficult reasoning:** On models supporting adaptive thinking, tune effort according to task difficulty, quality, and latency.
- **Evaluate the generated answer:** Judge correctness, completeness, grounding, and consistency rather than assuming longer reasoning is better.

Anthropic's prompting guidance distinguishes demonstration-based steering from native thinking and recommends avoiding unnecessary prescriptive reasoning instructions on capable models.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Exam Traps**

| Trap                                                                | Why tempting                                       | Why wrong                                                                                                        |
| ------------------------------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Few-shot is always superior to zero-shot.                           | Examples can improve consistency.                  | They introduce token overhead and can anchor the model to narrow patterns.                                       |
| More examples always improve quality.                               | More demonstrations appear more informative.       | Irrelevant or repetitive examples can cause overfitting to prompt patterns and increase cost.                    |
| Always use CoT for accuracy.                                        | Reasoning often improves complex-task performance. | Simple tasks may gain nothing and incur unnecessary latency.                                                     |
| Asking for a long reasoning trace is equivalent to native thinking. | Both appear to involve reasoning.                  | Native reasoning uses model-specific mechanisms; visible rationale is not the internal reasoning process.        |
| Increase reasoning effort before checking the evidence.             | More thinking sounds like greater intelligence.    | Missing facts, incorrect retrieval, or inconsistent task definitions cannot reliably be fixed by more reasoning. |

**Exam heuristic:** Use few-shot prompting when the model understands the task but applies the wrong classification boundary, structure, or style. Use stronger reasoning capability when the underlying task requires difficult inference.

## **2.4 Optimize context windows and manage token usage**

### **Core Concept**

**Context engineering** determines what information Claude receives, how it is ordered, and how unnecessary material is prevented from consuming the context budget.

A large context window provides capacity, not guaranteed relevance.

The architect must optimize:

1. Context relevance and grounding.
2. Input token consumption.
3. Output and reasoning token budgets.
4. Latency and request costs.
5. Multi-turn memory and state preservation.

### **Context window fundamentals**

For an API interaction, the context budget must accommodate the relevant inputs and generated output.

```
Context Budget
|
+-- System instructions
+-- Tool definitions
+-- Conversation history
+-- Retrieved documents
+-- Images and other supported content
+-- Tool results
+-- Generated output
    +-- Reasoning tokens, where applicable
```

⭐ **Critical rule:** A model's context-window capacity is not the same as its maximum output-token allowance.

For example, a model supporting 1M tokens of context does not necessarily support generating 1M tokens in a single response.

As of July 31, 2026, Sonnet 5, Opus 5, Sonnet 4.6, and Opus 4.8 supported 1M-token context windows, while Haiku 4.5 used a 200K-token window. These larger contexts should be treated as available capacity rather than an instruction to always fill them.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Canonical Skeleton**

```
User Query
     |
     v
Intent / Complexity Analysis
     |
     v
Relevant Context Selection
  - Conversation state
  - Retrieved documents
  - Tool outputs
  - Required policies
     |
     v
Context Filtering
  - Deduplicate
  - Remove irrelevant material
  - Prioritize important evidence
     |
     v
Token Counting
     |
     v
Within Budget?
   /       \
 YES        NO
  |          |
  |      Retrieve Less /
  |      Compact / Clear
  |          |
  +----------+
     |
     v
Construct Grounded Prompt
     |
     v
Claude Inference
     |
     v
Validate Answer and Evidence
     |
     v
Log Usage → Optimize
```

### **Context optimization techniques**

| Technique           | Mechanism                                                      | When to choose                                 |
| ------------------- | -------------------------------------------------------------- | ---------------------------------------------- |
| Selective retrieval | Include only relevant document chunks                          | Large enterprise knowledge bases               |
| Context pruning     | Remove redundant or irrelevant inputs                          | Bloated prompts                                |
| Summarization       | Replace detailed history with concise state                    | Long-running conversations                     |
| Context editing     | Clear eligible older tool results or thinking blocks           | Tool-heavy agent workflows                     |
| Compaction          | Condense conversation history while preserving essential state | Near context exhaustion                        |
| External memory     | Save persistent facts/state outside the prompt                 | Multi-session applications                     |
| Token counting      | Estimate input size before inference                           | Budget enforcement, routing, capacity planning |

**Version note:** Context editing and memory tooling were available before July 2026. Server-side compaction entered beta in February 2026 for supported models. Their configuration and availability must be checked by model and platform.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Long-document prompt design**

For long documents, Anthropic recommends placing the document material near the beginning of the relevant prompt content, separating sources with XML tags, and putting the specific query after the source material.

```
<documents>
  <document index="1">
    <source>policy_v3.pdf</source>
    <document_content>
      {{POLICY_DOCUMENT}}
    </document_content>
  </document>
</documents>

<question>
What is the escalation procedure for this exception?
</question>

<instructions>
Identify the relevant provisions.
Cite the supporting text.
State when the answer is not supported
by the document.
</instructions>
```

This document-ordering advice concerns the content of the task prompt; it does not mean moving stable application policies out of the system instruction layer.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs



### **Best Practices**

- ⭐ **Prefer relevant context over maximum context:** Use retrieval and filtering to reduce noise.
- **Count tokens programmatically:** Use the Messages token-counting API where supported, and use actual response `usage` for billing and monitoring.
- **Preserve critical information during compaction:** Keep decisions, user requirements, unresolved tasks, and evidence references.
- **Track full context consumption:** Include tool descriptions, documents, history, and reasoning-output allowances.
- **Measure context quality:** Evaluate grounded-answer accuracy, citation correctness, token costs, and P95 latency.

### **Exam Traps**

| Trap                                                   | Why tempting                                 | Why wrong                                                                    |
| ------------------------------------------------------ | -------------------------------------------- | ---------------------------------------------------------------------------- |
| Use the entire 1M-token window whenever possible.      | More context appears to mean better answers. | Irrelevant information can dilute evidence and increase processing costs.    |
| Summarize everything aggressively.                     | Maximizes token reduction.                   | Important exceptions, decisions, and evidence may be lost.                   |
| Prompt caching reduces context-window consumption.     | Cached content costs fewer input tokens.     | Cached tokens still occupy context capacity.                                 |
| `max_tokens` controls total context size.              | It is a token limit.                         | It primarily controls generated output, not the full request context window. |
| A larger model fixes an irrelevant retrieved document. | Stronger reasoning may appear sufficient.    | The source-selection failure remains unresolved.                             |

**Exam heuristic:** If a long-running agent is failing because of excessive conversation history, first determine whether relevant state can be preserved while stale tool outputs or redundant history are removed. Increasing the context window is not automatically the best answer.

## **2.5 Implement prompt reuse strategies (caching, modular prompts, Skills)**

### **Core Concept**

Prompt reuse reduces repeated work, improves consistency, and makes enterprise prompting easier to maintain.

The objective explicitly includes three mechanisms:

| Mechanism           | What it reuses                                           | Principal benefit                             |
| ------------------- | -------------------------------------------------------- | --------------------------------------------- |
| **Prompt caching**  | Previously processed identical prompt prefixes           | Lower cost and time-to-first-token            |
| **Modular prompts** | Reusable instruction and template components             | Maintainability, consistency, version control |
| **Skills**          | Reusable workflows, instructions, scripts, and resources | On-demand domain specialization               |

⭐ **Crucial distinction:** Caching, templates, and Skills solve different problems. They are complementary, not interchangeable.

### **A. Prompt caching**

Prompt caching allows Claude to reuse processing of identical prompt prefixes across requests.

Its main targets include:

- Repeated system instructions.
- Large policy documents.
- Shared background material.
- Tool definitions.
- Growing conversation prefixes.

#### **Canonical caching architecture**

```
REQUEST
|
+-- Stable tool definitions
+-- Stable system instructions
+-- Shared policies
+-- Shared examples
|         |
|         v
|    CACHE BREAKPOINT
|         |
+-- Dynamic user input
+-- Dynamic retrieved context
          |
          v
     Claude API
          |
       Cache hit?
      /         \
    YES          NO
     |            |
Cached read   Cache creation
     |            |
     +------+-----+
            |
            v
       Generate Response
            |
            v
      Inspect Usage Fields
```

Anthropic's cache behavior processes the prompt in the order **tools → system → messages**. Cache hits depend on matching the relevant prefix, including applicable configuration and model/platform constraints.

#### **Example: explicit cache breakpoint**

```
import anthropicclient = anthropic.Anthropic()response = client.messages.create(    model="claude-sonnet-5",    max_tokens=1024,    system=[        {            "type": "text",            "text": SHARED_POLICY_TEXT,            "cache_control": {                "type": "ephemeral"            }        }    ],    messages=[        {            "role": "user",            "content": USER_QUERY        }    ])print(response.usage)
```

*Illustrative July 2026 API pattern. `SHARED_POLICY_TEXT` and `USER_QUERY` represent application-provided values.*

For growing conversations, top-level automatic caching was also available by July 2026:

```
cache_control={"type": "ephemeral"}
```

Automatic caching manages the moving breakpoint. Explicit placement is especially useful when independent requests reuse a long static prefix but each has different dynamic content.

### **Cache economics**

| Operation               | Default pricing multiplier |
| ----------------------- | -------------------------- |
| Normal input            | 1.0×                       |
| Five-minute cache write | 1.25×                      |
| One-hour cache write    | 2.0×                       |
| Cache read              | 0.1×                       |

These were the relevant multipliers for ordinary July 2026 models. The read multiplier is model-dependent across the wider product lineup.

A 5-minute write pays back after one subsequent cache read compared with repeatedly processing the same prefix normally. A 1-hour write pays back after two reads.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Cache limits to memorize**

| Setting                       | July 2026 behavior                                                 |
| ----------------------------- | ------------------------------------------------------------------ |
| Default TTL                   | 5 minutes                                                          |
| Extended TTL                  | 1 hour                                                             |
| Maximum explicit breakpoints  | 4                                                                  |
| Cache type                    | `ephemeral`                                                        |
| First cache write             | More expensive than normal input                                   |
| Subsequent cache read         | Discounted input processing                                        |
| Prompt requirement            | Prefix must be identical and meet the model's minimum cache length |
| Cache hit verification        | Inspect `cache_read_input_tokens`                                  |
| Cache miss/write verification | Inspect `cache_creation_input_tokens`                              |

**Minimum cacheable prefix lengths — model-specific:**

| Model             | Minimum tokens |
| ----------------- | -------------- |
| Claude Haiku 4.5  | 4,096          |
| Claude Sonnet 4.6 | 1,024          |
| Claude Sonnet 5   | 1,024          |
| Claude Opus 4.6   | 4,096          |
| Claude Opus 4.7   | 2,048          |
| Claude Opus 4.8   | 1,024          |
| Claude Opus 5     | 512            |

These thresholds apply to supported Claude API deployments; availability and constraints should be checked for specific cloud-provider integrations.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs



### **B. Modular prompts**

Modular prompts separate shared components from task-specific instructions.

```
Enterprise Prompt Library
|
+-- Shared Policy Module
+-- Shared Output Rules
+-- Task Module
|    +-- Classification
|    +-- Extraction
|    +-- Summarization
|
+-- Optional Examples
|
+-- Dynamic User Data
         |
         v
    Prompt Assembly
         |
         v
      Claude API
```

**Benefits:** Centralized updates, versioning, testing, easier debugging, and consistent behavior.

**Limitation:** Modular prompts do not automatically reduce token costs. If all modules are expanded into every request, they may increase context consumption. Combining modularity with caching or selective assembly is usually preferable.

### **C. Agent Skills**

Skills package reusable instructions and resources in directories centered around `SKILL.md`.

A typical structure:

```
contract-review/
|
+-- SKILL.md
+-- references/
|    +-- review-policy.md
|    +-- clause-taxonomy.md
|
+-- scripts/
     +-- validate_output.py
```

The Skill describes what Claude can do and how to execute the workflow.

#### **⭐ Progressive disclosure model**

```
Level 1: Skill metadata
         name + description
               |
               v
       Skill is relevant?
               |
               v
Level 2: Load SKILL.md
         Workflow instructions
               |
               v
Level 3: Load resources/scripts
         Only when required
```

This avoids injecting the entire workflow library into every request.

**July 2026 status:** Agent Skills and the Skills API were available in beta through the `skills-2025-10-02` mechanism. API-based Skills required an appropriate code-execution container. Do not apply later generally available API behavior to a July 2026 exam question.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Best Practices**

- ⭐ **Place stable content before variable content:** Maximize reusable prompt prefixes and cache hits.
- **Choose TTL by reuse frequency:** Five-minute caching for frequently repeated prefixes; one-hour caching when reuse intervals justify the higher write price.
- **Version shared prompt components:** Maintain testing and rollback procedures for policies, examples, and templates.
- **Use Skills for reusable workflows:** Prefer on-demand loading instead of embedding large procedural instructions into every prompt.
- **Monitor actual cache behavior:** Validate hit rates, token billing, and context usage instead of assuming caching works because it was configured.

### **Exam Traps**

| Trap                                                 | Why tempting                                | Why wrong                                                                                             |
| ---------------------------------------------------- | ------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Place dynamic user content before the cached policy. | Mirrors natural conversational ordering.    | Changes invalidate the reusable prefix.                                                               |
| Enable caching for any short prompt.                 | Caching sounds universally beneficial.      | Minimum token thresholds may prevent caching entirely.                                                |
| Assume caching means saving Claude's final answer.   | The word "cache" suggests response reuse.   | Prompt caching reuses processing of prefixes, not completed answers.                                  |
| Use a Skill for every reusable sentence.             | Skills provide convenient packaging.        | Small stable instructions are generally better handled through prompt modules or system instructions. |
| Load every Skill file into the system prompt.        | Guarantees the model has every instruction. | Defeats progressive disclosure and increases context consumption.                                     |

**Exam heuristic:** A repeated large prefix points to caching. Shared instructions requiring controlled maintenance point to modular prompts. Reusable multi-step workflows requiring optional resources point to Skills.

# **Part 2: Consolidated Sections**

## **Key Facts to Memorize**

### **1. Model selection**

| ⭐ Fact                     | Memorize                                                                                      |
| -------------------------- | --------------------------------------------------------------------------------------------- |
| Selection criterion        | Lowest-cost configuration satisfying all mandatory constraints                                |
| Quality evaluation         | Representative, task-specific golden datasets                                                 |
| Important trade-offs       | Accuracy, cost, latency, context, safety, feature support                                     |
| Operational latency        | P50, P95, and sometimes P99                                                                   |
| Model efficiency           | Cost per successful task, not just cost per token                                             |
| Common starting candidates | Haiku for simple/high-volume work, Sonnet for broad enterprise tasks, Opus for demanding work |
| Model routing              | Direct requests to different models according to task complexity                              |
| Versioning                 | Re-evaluate changes in model ID, model behavior, or prompting configuration                   |

### **2. Prompt engineering and guardrails**

| ⭐ Fact                 | Memorize                                                                     |
| ---------------------- | ---------------------------------------------------------------------------- |
| System prompt          | Stable behavior and application-level instructions                           |
| Prompt template        | Reusable instructions with dynamic variables                                 |
| Zero-shot              | Instructions without demonstrations                                          |
| Few-shot               | Instructions plus representative examples                                    |
| CoT prompting          | Natural-language reasoning guidance                                          |
| Native thinking        | API/model-supported reasoning capability                                     |
| `output_config.effort` | Controls reasoning/work effort on supporting models                          |
| XML tags               | Structural organization, not a security boundary                             |
| Structured outputs     | Schema-conforming output where supported; still requires semantic validation |
| Guardrails             | Prompt controls plus deterministic application controls                      |
| Prompt injection       | Untrusted content attempting to alter intended model behavior                |

### **3. Reasoning modes — July 2026**

| Model family                                     | Important distinction                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| Claude 4.5 and earlier supported thinking models | Manual extended thinking with `budget_tokens`                                               |
| Claude 4.6                                       | Adaptive thinking supported; manual thinking deprecated but still functional                |
| Claude 4.7 / 4.8                                 | Adaptive thinking; manual `budget_tokens` unsupported                                       |
| Claude Sonnet 5                                  | Adaptive thinking enabled by default                                                        |
| Claude Opus 5                                    | Adaptive thinking enabled by default; effort-dependent restrictions when disabling thinking |

⭐ **Do not memorize&#x20;**`budget_tokens`**&#x20;as the universal reasoning configuration.** Always consider the model generation and its supported API interface.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **4. Context and token optimization**

| Concept                 | High-yield detail                                                              |
| ----------------------- | ------------------------------------------------------------------------------ |
| Context window          | Budget covering relevant prompt content and generated output                   |
| 1M-token models         | Sonnet 4.6, Sonnet 5, Opus 4.8, Opus 5, among others available by July         |
| 200K-token model        | Haiku 4.5                                                                      |
| Token counting endpoint | `/v1/messages/count_tokens`                                                    |
| Actual token usage      | Response `usage` object                                                        |
| Input usage fields      | `input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`       |
| Output usage            | `output_tokens`                                                                |
| Thinking usage          | `usage.output_tokens_details.thinking_tokens` on supported models              |
| Compaction              | Condense history while preserving critical task state                          |
| Context editing         | Remove eligible obsolete context content                                       |
| RAG                     | Retrieve relevant evidence instead of always including entire knowledge stores |

**⭐ Subtle but frequently tested:** When prompt caching is used, `input_tokens` alone does not represent all input tokens. Cached reads and writes must also be included when accounting for context and costs.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **5. Caching and prompt reuse**

| Concept                    | Key fact                                      |
| -------------------------- | --------------------------------------------- |
| Prompt caching             | Reuses identical prompt prefixes              |
| Default TTL                | 5 minutes                                     |
| Longer TTL                 | 1 hour                                        |
| Five-minute write          | 1.25× input price                             |
| One-hour write             | 2× input price                                |
| Typical cache read         | 0.1× input price                              |
| Maximum cache breakpoints  | 4                                             |
| Explicit caching parameter | `cache_control`                               |
| Preferred ordering         | Stable content first, changing content later  |
| Modular prompts            | Reusable, independently maintained components |
| Skills                     | Reusable domain workflows, loaded on demand   |
| Skill entry file           | `SKILL.md`                                    |
| API Skill execution        | Requires supported code execution environment |

### **6. Batch API — important adjacent optimization concept**

Although evaluation and cost optimization are principally covered under Domain 4, the Message Batches API is relevant to Domain 2 when selecting cost-efficient execution patterns.

| Property                   | July 2026 behavior                             |
| -------------------------- | ---------------------------------------------- |
| API                        | Message Batches API                            |
| Processing style           | Asynchronous                                   |
| Standard discount          | 50% on eligible input and output usage         |
| Maximum requests per batch | 100,000                                        |
| Maximum batch size         | 256 MB                                         |
| Processing window          | Up to 24 hours; individual requests may expire |
| Real-time responses        | Not its intended use case                      |
| Prompt caching combination | Supported, but cache hits are best-effort      |

**⭐ Decision rule:** Use Batch API for bulk operations that tolerate asynchronous completion. Use the synchronous Messages API for interactive workloads with strict response-time SLAs.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs



### **7. Evaluation metrics to remember**

| Metric                | Measures                                 | Relevant objective         |
| --------------------- | ---------------------------------------- | -------------------------- |
| Accuracy              | Correct predictions overall              | Model selection            |
| Precision             | Correctness of positive predictions      | Classification/extraction  |
| Recall                | Coverage of relevant positives           | Extraction/retrieval       |
| F1 score              | Harmonic mean of precision and recall    | Balanced classification    |
| Exact-match rate      | Agreement with expected answer           | Structured tasks           |
| JSON/schema validity  | Output format conformance                | Prompt design              |
| Groundedness          | Whether claims are supported by evidence | Context engineering        |
| Citation accuracy     | Whether cited evidence supports claims   | Long-document processing   |
| P50/P95 latency       | Typical and tail response time           | Model optimization         |
| TTFT                  | Time to first generated token            | Latency/caching            |
| Token usage           | Input/output consumption                 | Cost optimization          |
| Cache hit rate        | Frequency of reusable-prefix hits        | Prompt caching             |
| Task success rate     | End-to-end successful completion         | Architecture effectiveness |
| Human escalation rate | Frequency of required human intervention | Workflow reliability       |

**Evaluation method selection:**

- **Code-based grading:** Prefer for deterministic properties such as exact matches, parseable JSON, schema constraints, and calculated values.
- **LLM-as-judge:** Appropriate for nuanced assessments using explicit rubrics; calibrate against trusted labels.
- **Human evaluation:** Appropriate for high-stakes judgment, ambiguous cases, and calibration of automated graders.

No universal passing threshold exists for these application metrics. Thresholds should be derived from business requirements, regulatory obligations, and representative evaluation results.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs



### **8. Always / Never — exam decision rules**

| Rule                                                              | Architect-level justification                                  |
| ----------------------------------------------------------------- | -------------------------------------------------------------- |
| ⭐ Always define success criteria before model optimization.       | Otherwise there is no objective way to choose a configuration. |
| ⭐ Always validate model-dependent features.                       | API capabilities and parameter combinations differ by version. |
| Always test prompts against representative edge cases.            | Happy-path success is not production reliability.              |
| Always preserve important state during context reduction.         | Aggressive compaction can destroy correctness.                 |
| Always enforce sensitive permissions in application logic.        | Prompt instructions are not authorization mechanisms.          |
| Never assume more reasoning is inherently better.                 | It can increase latency without improving task quality.        |
| Never assume a larger context window improves evidence relevance. | Context selection remains necessary.                           |
| Never assume prompt caching reduces context-window occupancy.     | Cached tokens still count toward context.                      |
| Never assume Skills eliminate prompt token overhead.              | Metadata and loaded instructions still consume context.        |
| Never select a model based solely on token pricing.               | Total successful-task cost matters.                            |

# **Part 3: Five Exam-Style Questions**

*These are original practice questions designed around architect-level decision-making, not questions from the live certification examination.*

## **Question 1 — Model selection and routing**

**Objective: 2.1 Select appropriate Claude models based on trade-offs**

A multinational organization is building an internal assistant that handles 250,000 employee requests per day.

Initial benchmarking shows:

- Haiku meets the accuracy target for 88% of requests.
- Sonnet meets the target for 97% of requests.
- Opus improves the most complex cases but increases cost.
- The system must satisfy a strict P95 latency SLA.
- Approximately 12% of requests require complex reasoning.

Which architecture is the BEST choice?

**A.** Use Opus for every request because it delivers the strongest overall reasoning.

**B.** Use Haiku for simple requests and route ambiguous or complex requests to Sonnet or Opus, validating overall quality and latency.

**C.** Use Haiku for every request and increase the system prompt length until complex-case accuracy improves.

**D.** Randomly distribute requests across all three models to balance usage.

### **Correct Answer: B**

**Why B is correct:** It aligns model capability with request complexity, reduces unnecessary spending, and allows end-to-end performance to be measured against quality and P95 latency constraints. The routing logic itself must be evaluated to ensure complex cases are not misclassified.

**Why the others are wrong:**

- **A:** Optimizes intelligence without respecting cost efficiency.
- **C:** Assumes prompt length can compensate for reasoning limitations.
- **D:** Distributes workload without considering suitability or correctness.

⭐ **Exam takeaway:** Optimize cost per successful task using measured routing policies, not arbitrary model selection.

## **Question 2 — System prompts and guardrails**

**Objective: 2.2 Design system prompts, templates, and guardrails**

A financial-services organization deploys Claude to summarize customer refund requests.

The agent has access to a refund-processing API. A malicious customer submits a document containing instructions telling the agent to bypass approval rules.

The system prompt already says that refunds above $5,000 require manager approval.

Which change BEST protects the application?

**A.** Repeat the approval instruction ten times in the system prompt.

**B.** Use XML tags around the customer's document and assume the contents cannot influence Claude.

**C.** Enforce approval and authorization requirements in the application layer before invoking the refund API, while treating customer documents as untrusted data.

**D.** Switch to Opus because a stronger model can always distinguish malicious instructions from legitimate requests.

### **Correct Answer: C**

**Why C is correct:** It establishes a deterministic security boundary. Untrusted documents may be analyzed, but they cannot authorize privileged actions. The application independently checks policy, user permissions, and required approvals.

**Why the others are wrong:**

- **A:** Stronger prompt wording is not an authorization mechanism.
- **B:** XML tags help separate content but do not prevent injection.
- **D:** Model intelligence does not replace application security controls.

⭐ **Exam takeaway:** Prompt guardrails are defense-in-depth controls, not substitutes for least privilege and deterministic enforcement.

## **Question 3 — Choosing the prompting technique**

**Objective: 2.3 Apply prompt engineering techniques (zero-shot, few-shot, chain-of-thought)**

A legal-operations team extracts renewal conditions from supplier contracts.

Claude correctly identifies the clauses but inconsistently labels automatic-renewal exceptions. The team has 20 representative annotated examples covering the relevant clause variants.

The system already meets latency requirements.

What is the BEST next step?

**A.** Add a small, diverse selection of examples that demonstrate correct labeling of normal, exceptional, and ambiguous renewal clauses, then evaluate against a held-out dataset.

**B.** Enable maximum reasoning effort for every contract, regardless of clause complexity.

**C.** Increase the model's maximum context window without changing the prompt.

**D.** Replace the entire extraction workflow with a multi-agent architecture.

### **Correct Answer: A**

**Why A is correct:** The model already understands the extraction task. The failure concerns inconsistent application of classification boundaries. Few-shot examples directly address this problem and can be evaluated without introducing unnecessary architectural complexity.

**Why the others are wrong:**

- **B:** Adds reasoning cost without directly addressing labeling consistency.
- **C:** More context does not explain the desired classification boundary.
- **D:** Introduces orchestration complexity for a problem that may be solved through examples.

⭐ **Exam takeaway:** Match prompt techniques to the observed failure mode. Do not escalate complexity prematurely.

## **Question 4 — Context engineering and token optimization**

**Objective: 2.4 Optimize context windows and manage token usage**

An enterprise research assistant answers questions over thousands of internal policy documents.

The engineering team proposes sending all relevant departments' policy documents into every request because the selected model supports a 1M-token context window.

Testing shows increased latency, higher costs, and occasional answers citing superseded policies.

Which approach is BEST?

**A.** Continue sending all policies and increase reasoning effort.

**B.** Implement retrieval that filters documents by authorization, currency, and relevance; assemble a concise grounded context and evaluate citation correctness.

**C.** Summarize the entire policy corpus into one permanent system prompt.

**D.** Switch to the model with the largest available output token limit.

### **Correct Answer: B**

**Why B is correct:** It addresses both context relevance and cost. Retrieval should filter obsolete or unauthorized documents before inference, while the answer should be validated against the selected evidence.

**Why the others are wrong:**

- **A:** More reasoning cannot reliably repair irrelevant or outdated evidence.
- **C:** Static summarization risks losing exceptions, source fidelity, and updateability.
- **D:** Output capacity does not solve retrieval quality.

⭐ **Exam takeaway:** Context engineering prioritizes the right information, not the maximum amount of information.

## **Question 5 — Prompt reuse and caching**

**Objective: 2.5 Implement prompt reuse strategies (caching, modular prompts, Skills)**

A company runs an internal compliance assistant handling thousands of requests daily.

Every request includes:

- An identical 8,000-token policy document.
- A standard system instruction.
- A short, changing user question.

Most requests arrive within a few minutes of one another. The policy must remain available in full because important exceptions may appear anywhere in the document.

The architect must reduce latency and token costs without compromising completeness.

Which approach is BEST?

**A.** Truncate the policy to 1,000 tokens.

**B.** Store the policy inside a Skill and assume that this automatically eliminates its token cost.

**C.** Use the Message Batches API for all interactive questions.

**D.** Keep stable instructions and the full policy in a reusable prefix, enable prompt caching, and place the changing user question after the cached content.

### **Correct Answer: D**

**Why D is correct:** Prompt caching is directly suited to repeated large prefixes. It preserves the complete policy while reducing repeated input processing costs and time-to-first-token on cache hits.

**Why the others are wrong:**

- **A:** Can remove important compliance exceptions.
- **B:** Skills provide reusable instructions and on-demand resources; they do not inherently create cache hits or eliminate input costs.
- **C:** Batch processing is asynchronous and unsuitable when interactive latency is a requirement.

⭐ **Exam takeaway:** A long, frequently reused static prefix is the classic prompt-caching scenario.

# **Last-Minute Revision Checklist**

- ⭐ Can I choose Haiku, Sonnet, or Opus based on binding quality, cost, and latency requirements?
- Can I distinguish system prompts, prompt templates, XML structure, and application-enforced guardrails?
- Can I choose between zero-shot, few-shot, prompted CoT, and native thinking?
- Do I understand model-specific adaptive thinking, `effort`, and deprecated `budget_tokens` behavior?
- Can I explain context-window size versus output limits and token consumption?
- Can I choose retrieval, pruning, context editing, or compaction without losing essential state?
- ⭐ Can I explain prompt caching, identical prefixes, TTLs, minimum lengths, and usage fields?
- Can I distinguish modular prompts from Skills and explain progressive disclosure?
- Can I distinguish synchronous requests, caching, and asynchronous Batch API optimization?
- ⭐ Can I select the best architecture using measurable acceptance criteria instead of model size or prompt complexity alone?

**Final exam decision principle:** For Domain 2, the strongest answer typically preserves required quality and controls while minimizing unnecessary model capability, reasoning, context, repetition, and cost.
