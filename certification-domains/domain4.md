# **Claude Certified Architect – Professional (CCA-P)**

# **Domain 4: Evaluation, Testing & Optimization (16%)**

**Exam scope:** July 2026\
**Difficulty:** Architect / Professional\
**Primary focus:** Evaluation-driven development, mixed evaluation methodologies, systematic failure diagnosis, performance optimization, and production observability.

⭐ indicates a high-yield concept worth prioritizing during exam preparation.

The six task statements below follow the July 2026 exam structure, in their original order. Section numbering and priority ratings are study aids, not official individual task weightings. The supplied link could not be rendered directly, so the task statements were verified against an accessible matching version of the exam outline.

| Sub-domain | Primary competency                                                | Study priority |
| ---------- | ----------------------------------------------------------------- | -------------- |
| 4.1        | Defining evaluation metrics                                       | Very high      |
| 4.2        | Designing evaluation datasets and mixed-method testing frameworks | Very high      |
| 4.3        | A/B testing and iterative improvement                             | High           |
| 4.4        | Diagnosing prompts, hallucinations and model mismatches           | Very high      |
| 4.5        | Optimizing tokens, latency and cost-performance                   | Very high      |
| 4.6        | Monitoring performance through logging and observability          | High           |

## **Central mental model**

The strongest architectural decision is rarely to immediately change the model, rewrite the prompt, or increase the context window.

Instead, establish measurable requirements, identify the failing component, apply a targeted intervention, and demonstrate improvement through controlled evaluation.

```
Business Requirements
        │
        ▼
Measurable Success Criteria
        │
        ▼
Representative Evaluation Dataset
        │
        ▼
Baseline Evaluation
        │
        ▼
Failure Classification
        │
        ▼
Targeted Optimization
        │
        ▼
Regression Evaluation
        │
        ▼
Controlled Production Rollout
        │
        ▼
Continuous Monitoring
        │
        └────── Feedback into Evaluation Dataset
```

**Domain 4 is fundamentally about evidence-based architectural decisions rather than trial-and-error development.**

# **Part 1: Per Sub-Domain Breakdown**

## **4.1 Define evaluation metrics: accuracy, latency, cost, safety and security**

### **Core Concept**

An enterprise Claude application must be evaluated against multiple dimensions of success, not just response accuracy.

A technically accurate application can still fail its business requirements because it is too expensive, too slow, unreliable, or unsafe.

⭐ **Define success criteria before optimizing the application.**

Evaluation metrics should be:

- Specific: Clearly define what constitutes success.
- Measurable: Use quantitative metrics or explicit scoring rubrics.
- Relevant: Directly connected to the business objective.
- Multidimensional: Cover quality, operational performance, safety and security.
- Risk-sensitive: Reflect the consequences of different failure types.

### **Essential evaluation metrics**

| Dimension      | Metrics                                          | What they measure                                |
| -------------- | ------------------------------------------------ | ------------------------------------------------ |
| Quality        | Accuracy, precision, recall, F1                  | Correctness of predictions or extractions        |
| Groundedness   | Faithfulness, unsupported claim rate             | Whether answers are supported by evidence        |
| Retrieval      | Recall\@K, Precision\@K, MRR, nDCG               | Relevance and ranking of retrieved information   |
| Latency        | TTFT, P50, P95, P99                              | Responsiveness and tail latency                  |
| Cost           | Cost per request, cost per successful task       | Economic efficiency                              |
| Reliability    | Task completion rate, error rate                 | Consistency of system operation                  |
| Safety         | Harmful output rate, false refusal rate          | Appropriate handling of safe and unsafe requests |
| Security       | Injection success rate, unauthorized action rate | Resistance to adversarial activity               |
| Business value | Resolution rate, time saved, escalation rate     | Actual organizational benefit                    |

### **⭐ Precision vs. recall**

| Metric    | Formula                                       | Prioritize when                                                     |
| --------- | --------------------------------------------- | ------------------------------------------------------------------- |
| Precision | TP / (TP + FP)                                | False positives are expensive                                       |
| Recall    | TP / (TP + FN)                                | Missing a relevant result is expensive                              |
| F1        | 2 × Precision × Recall / (Precision + Recall) | Both error types matter                                             |
| Accuracy  | (TP + TN) / Total                             | Overall correctness is meaningful and class imbalance is manageable |

For example, a contract analysis application identifying high-risk contractual clauses may prioritize recall to minimize missed risks, while a downstream automated remediation system may prioritize precision to avoid inappropriate actions.

### **⭐ Hard constraints vs. optimization objectives**

Distinguish requirements that must be satisfied from metrics that should be improved.

Consider the following illustrative requirements for an enterprise document intelligence application:

| Metric                       | Target             | Requirement type       |
| ---------------------------- | ------------------ | ---------------------- |
| Extraction F1                | ≥ 95%              | Minimum quality gate   |
| Critical clause recall       | ≥ 99%              | Minimum quality gate   |
| P95 latency                  | ≤ 3 seconds        | Operational constraint |
| Unauthorized tool actions    | 0 observed         | Security gate          |
| Cost per successful document | As low as possible | Optimization objective |

These thresholds are examples, not universal Anthropic requirements.

A configuration that reduces costs by 50% but violates a mandatory quality or security threshold is not an acceptable optimization.

### **Canonical Skeleton**

```
Business Objective
        ↓
Identify Failure Consequences
        ↓
Define Quality and Operational Metrics
        ↓
Establish Acceptance Thresholds
        ↓
Measure Baseline
        ↓
Compare Candidate Configurations
        ↓
Reject Configurations Failing Hard Constraints
        ↓
Select Best Remaining Configuration
```

### **Best Practices**

- Establish explicit, task-specific success criteria before prompt development.
- Use multiple metrics rather than relying on a single aggregate score.
- Evaluate representative scenarios, including edge cases and adversarial inputs.
- Measure latency distributions instead of relying exclusively on averages.
- Track business outcomes alongside technical model performance.

Anthropic's [evaluation guidance](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests?utm_source=chatgpt.com) emphasizes measurable, use-case-specific success criteria and systematic testing.

## **4.2 Design evaluation datasets and test frameworks using mixed methodologies**

### **Core Concept**

LLM applications are probabilistic systems. Traditional unit testing remains important, but exact-output assertions alone cannot adequately evaluate open-ended generation, reasoning, research or agentic behavior.

A robust evaluation framework combines three grading approaches:

1. Code-based evaluation.
2. Model-based evaluation.
3. Human evaluation.

⭐ **Use the least expensive grading method that can reliably evaluate the required behavior.**

### **Three evaluation methodologies**

| Method      | Examples                                            | Strengths                             | Limitations                                      |
| ----------- | --------------------------------------------------- | ------------------------------------- | ------------------------------------------------ |
| Code-based  | Exact match, regex, JSON validation, unit tests     | Fast, inexpensive, repeatable         | Limited semantic understanding                   |
| Model-based | LLM-as-a-judge, rubric scoring, pairwise comparison | Scalable semantic evaluation          | Judge bias, variability, additional cost         |
| Human-based | Expert review, annotation, blind comparison         | Domain expertise and nuanced judgment | Expensive, slow, potential reviewer disagreement |

### **Choosing the right grader**

| Evaluation requirement        | Best initial methodology                |
| ----------------------------- | --------------------------------------- |
| Valid JSON output             | Schema validator                        |
| Correct numerical calculation | Deterministic code                      |
| Correct classification label  | Exact match                             |
| Code functionality            | Unit and integration tests              |
| Summarization completeness    | Rubric-based LLM judge                  |
| Factual groundedness          | Source-aware LLM judge and verification |
| Legal interpretation          | Subject-matter expert                   |
| Writing quality               | Model judge calibrated against humans   |
| Unauthorized tool execution   | Tool audit and state assertions         |

### **⭐ Golden evaluation dataset**

A golden dataset is a curated set of test cases with trusted expected outcomes or explicit grading criteria.

It should contain more than successful, straightforward examples.

| Dataset category     | Purpose                                                    |
| -------------------- | ---------------------------------------------------------- |
| Representative cases | Measure typical production performance                     |
| Edge cases           | Evaluate unusual but legitimate inputs                     |
| Negative cases       | Ensure the application does not act when it should abstain |
| Adversarial cases    | Test security and guardrail effectiveness                  |
| Historical failures  | Prevent recurrence of known defects                        |
| Ambiguous cases      | Evaluate clarification and escalation behavior             |
| Long-context cases   | Test context management and evidence retrieval             |

### **Example evaluation record**

```
{
  "test_id": "CONTRACT_042",
  "category": "missing_information",
  "input": {
    "question": "What is the liability cap?",
    "document": "Contract with no liability cap clause"
  },
  "expected_behavior": {
    "must_identify_missing_information": true,
    "must_not_invent_amount": true
  },
  "graders": [
    "schema_validation",
    "groundedness_judge"
  ],
  "severity": "high"
}
```

For open-ended tasks, expected behavior and grading rubrics are often more appropriate than a single exact reference answer.

### **⭐ LLM-as-a-judge**

An evaluator model assesses the output of another model against defined criteria.

A useful judge prompt includes:

- The original task.
- The candidate response.
- Reference material or expected outcomes.
- Explicit scoring criteria.
- A structured evaluation result.

Example:

```
Evaluate the candidate answer against the supplied contract.

Score each dimension independently:

1. Factual correctness
2. Completeness
3. Evidence support
4. Compliance with instructions

Return:
- Pass or fail for each criterion
- Supporting evidence
- Identified errors
```

Avoid vague instructions such as "Rate this answer from 1 to 10."

### **LLM judge biases**

| Bias                 | Description                                | Mitigation                               |
| -------------------- | ------------------------------------------ | ---------------------------------------- |
| Position bias        | Prefers the first or second response       | Randomize answer ordering                |
| Verbosity bias       | Prefers longer responses                   | Explicitly score relevance and concision |
| Self-preference bias | Favors responses from its own model family | Independent judge and human calibration  |
| Style bias           | Rewards polished language over correctness | Separate factual and stylistic criteria  |
| Inconsistent scoring | Produces varying judgments                 | Clear rubrics and repeated evaluation    |

### **⭐ Evaluating agents vs. evaluating ordinary LLM calls**

Agent evaluation must consider both the final response and what actually happened in the external environment.

| Component             | What to evaluate                                      |
| --------------------- | ----------------------------------------------------- |
| Final response        | Correctness and completeness                          |
| Tool selection        | Whether the appropriate tool was chosen               |
| Tool arguments        | Whether parameters were valid                         |
| Execution trajectory  | Whether actions followed required constraints         |
| Environmental outcome | Whether the intended operation actually succeeded     |
| Efficiency            | Number of calls, tokens, time and retries             |
| Security              | Whether prohibited actions were attempted or executed |

An agent claiming that it successfully created a support ticket is not proof that the ticket exists. Verify the actual system state.

Do not require an exact tool-call sequence unless the sequence itself is a business, safety or compliance requirement.

### **⭐ Capability vs. regression evaluations**

| Evaluation type       | Purpose                                                           |
| --------------------- | ----------------------------------------------------------------- |
| Capability evaluation | Discover and improve behaviors the system cannot reliably perform |
| Regression evaluation | Ensure previously working behaviors remain functional             |
| End-to-end evaluation | Verify the complete application achieves its intended outcome     |
| Component evaluation  | Identify failures in specific system stages                       |

As capabilities mature, successful capability tests can become regression tests.

### **Agent reliability:&#x20;**`pass@k`**&#x20;vs.&#x20;**`pass^k`

| Metric   | Meaning                                        | Appropriate use                   |
| -------- | ---------------------------------------------- | --------------------------------- |
| `pass@1` | Success on the first attempt                   | Single-attempt tasks              |
| `pass@k` | At least one successful attempt among k trials | Solution exploration              |
| `pass^k` | All k attempts succeed                         | Consistency-critical applications |

For independent trials with an individual success probability of 80%:

- `pass@3` = 99.2%.
- `pass^3` = 51.2%.

A high `pass@k` can conceal poor single-attempt reliability. This distinction is particularly important for customer-facing agents.

### **Canonical Skeleton**

```
Production Examples + Expert Cases + Known Failures
                         ↓
                Curated Golden Dataset
                         ↓
            Development / Held-out Test Split
                         ↓
                  Evaluation Harness
                         ↓
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
    Code Grader      LLM Grader     Human Reviewer
         │               │               │
         └───────────────┼───────────────┘
                         ▼
              Metrics and Failure Analysis
                         ↓
               Regression Test Repository
```

### **Best Practices**

- Start with approximately 20–50 realistic agent evaluation tasks rather than delaying evaluation until a large dataset is available.
- Prefer deterministic checks where correctness is objectively verifiable.
- Calibrate model-based graders periodically using expert judgment.
- Keep final test cases separate from prompt-tuning examples.
- Run repeated trials for nondeterministic agents and verify actual outcomes.

Anthropic discusses these principles in [Demystifying evals for AI agents](https://www.anthropic.com/research/demystifying-evals-for-ai-agents?utm_source=chatgpt.com), published January 2026.

## **4.3 Conduct A/B testing and iterative improvements**

### **Core Concept**

A/B testing evaluates whether a candidate configuration produces a measurable improvement over an existing configuration under comparable conditions.

In Claude applications, experiments may compare:

- Model families or versions.
- System prompts and few-shot examples.
- Retrieval and reranking configurations.
- Thinking effort or reasoning budgets.
- Tool definitions and agent orchestration strategies.

⭐ **Change one independent variable at a time when the objective is to establish causality.**

Changing the model, prompt and retrieval strategy simultaneously makes it difficult to determine which modification caused the improvement or regression.

### **Offline vs. online evaluation**

| Method             | Description                                                                       | Best use                                    |
| ------------------ | --------------------------------------------------------------------------------- | ------------------------------------------- |
| Offline evaluation | Run configurations against a fixed test dataset                                   | Prompt development and regression testing   |
| A/B testing        | Randomly assign production users or requests to variants                          | Measuring real-world impact                 |
| Shadow testing     | Process production inputs using a candidate without exposing its outputs to users | Low-risk production validation              |
| Canary deployment  | Release a candidate to a small percentage of production traffic                   | Limiting deployment risk                    |
| Human review       | Experts assess sampled outputs                                                    | Validating subjective or high-risk outcomes |

Shadow testing must suppress external side effects. A shadow agent must not independently issue refunds, modify records or send emails.

### **⭐ Controlled experiment design**

| Experiment component | Example                                       |
| -------------------- | --------------------------------------------- |
| Hypothesis           | A revised prompt improves extraction accuracy |
| Control              | Existing production prompt                    |
| Treatment            | Revised prompt                                |
| Controlled variables | Model version, retrieval strategy, tools      |
| Primary metric       | Extraction F1                                 |
| Guardrail metrics    | P95 latency, cost, security violations        |
| Success criteria     | F1 improves without violating constraints     |
| Rollback criteria    | Critical regression or security failure       |

### **Interpreting experiment results**

Consider the following illustrative experiment:

| Metric                             | Configuration A | Configuration B |
| ---------------------------------- | --------------- | --------------- |
| Accuracy                           | 92%             | 96%             |
| P95 latency                        | 2.1 sec         | 3.8 sec         |
| Cost per request                   | $0.020          | $0.035          |
| Meets 3-second latency requirement | Yes             | No              |

Configuration B has better accuracy but violates the operational requirement.

The appropriate next step is to optimize B or evaluate another configuration, rather than deploy B solely because it achieves higher accuracy.

### **Canonical Skeleton**

```
Define Hypothesis
        ↓
Establish Baseline and Success Criteria
        ↓
Create Candidate Configuration
        ↓
Run Paired Offline Evaluations
        ↓
Check Regression and Security Gates
        ↓
Shadow Test / Randomized A/B Test
        ↓
Analyze Effect Size and Uncertainty
        ↓
Canary Deployment
        ↓
Monitor → Expand or Roll Back
```

### **Best Practices**

- Establish evaluation criteria and a baseline before experimenting.
- Use identical offline test cases and comparable environments across candidates.
- Randomize online traffic and maintain stable user assignments where conversational continuity matters.
- Account for nondeterminism, sample size, statistical uncertainty and practical significance.
- Complement automated evaluation with production measurements and periodic human review.

## **4.4 Diagnose system issues: prompt failure, hallucinations and model mismatch**

### **Core Concept**

Poor LLM application performance does not necessarily indicate poor model performance.

An enterprise Claude solution comprises several components, each introducing its own failure modes.

⭐ **Diagnose the failing component before selecting a corrective action.**

A stronger model cannot reliably compensate for incorrect retrieved information, defective tools, missing permissions or contradictory business instructions.

### **⭐ Failure diagnosis matrix**

| Observed symptom                          | Likely cause                        | First investigation                        |
| ----------------------------------------- | ----------------------------------- | ------------------------------------------ |
| Incorrect answers after document refresh  | Retrieval or indexing failure       | Retrieved chunks and document versions     |
| Correct facts but incorrect output format | Prompt or output contract failure   | Instructions and schema                    |
| Correct context but wrong conclusion      | Reasoning or model capability issue | Prompt complexity and model suitability    |
| Fabricated facts                          | Insufficient grounding              | Available evidence and source verification |
| Incorrect tool selection                  | Ambiguous tool definitions          | Names, descriptions and tool scope         |
| Incorrect tool arguments                  | Poor parameter definitions          | Input schema and examples                  |
| Failure in lengthy conversations          | Context degradation                 | Context size, truncation and compaction    |
| Intermittent failures under load          | Infrastructure or rate limits       | Traces, HTTP errors and retries            |
| Sudden cost increase                      | Token growth or agent loops         | Token usage, caching and tool iterations   |
| Increased refusals                        | Guardrail or instruction conflict   | Safety configuration and refusal examples  |

### **⭐ Diagnosing RAG failures**

A RAG application has at least two distinct quality layers: retrieval and generation.

```
User Query
    ↓
Query Processing
    ↓
Retrieval
    ↓
Reranking
    ↓
Context Assembly
    ↓
Claude Generation
    ↓
Answer Validation
```

| Observation                                       | Likely problem               | Recommended action                                  |
| ------------------------------------------------- | ---------------------------- | --------------------------------------------------- |
| Relevant evidence is absent                       | Retrieval failure            | Improve indexing, chunking or search                |
| Relevant evidence ranks poorly                    | Ranking failure              | Improve reranking                                   |
| Evidence is retrieved but omitted from the prompt | Context assembly failure     | Fix filtering or truncation                         |
| Evidence is present but ignored                   | Prompt or generation failure | Improve grounding instructions                      |
| Evidence is present but complex reasoning fails   | Model capability mismatch    | Evaluate a stronger model or thinking configuration |

### **Oracle context testing**

Oracle context testing isolates retrieval failures from generation failures by supplying known-correct evidence directly to the model.

```
Run A: Normal retrieved context
Run B: Known-correct reference context

A fails, B succeeds:
    Retrieval or context assembly problem

A fails, B fails:
    Investigate prompt, reasoning, model
    capability or output validation
```

This technique provides much stronger diagnostic evidence than arbitrarily switching models.

### **⭐ Hallucination mitigation**

Hallucination refers to generated information that is fabricated, incorrect or unsupported by available evidence.

| Technique           | Purpose                                            |
| ------------------- | -------------------------------------------------- |
| Evidence grounding  | Restrict factual claims to trusted context         |
| Direct quotation    | Identify supporting evidence before synthesis      |
| Citations           | Make factual claims traceable                      |
| Explicit abstention | Allow the model to acknowledge missing information |
| Verification step   | Check generated claims against source material     |
| Human escalation    | Handle uncertain, high-impact conclusions          |

Example grounding instruction:

```
Answer using only the supplied source documents.

For every material factual claim:
- Identify the supporting source.
- Verify that the source supports the claim.

If sufficient evidence is unavailable,
state that the information cannot be determined.

Do not infer missing dates, amounts or obligations.
```

Anthropic recommends uncertainty acknowledgment, source grounding and claim verification in its [hallucination reduction guidance](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations?utm_source=chatgpt.com).

### **Model mismatch diagnosis**

| Workload                              | Initial model strategy                     |
| ------------------------------------- | ------------------------------------------ |
| Simple classification and routing     | Haiku-class model                          |
| General-purpose enterprise generation | Sonnet-class model                         |
| Complex reasoning and analysis        | Evaluate Opus-class models                 |
| Difficult multi-step reasoning        | Evaluate supported thinking configurations |
| Mixed-complexity workload             | Model routing or escalation                |

These are starting points, not mandatory assignments. Select the model that meets measured workload requirements.

### **Prompt failure diagnosis**

Common causes include contradictory instructions, ambiguous requirements, unrepresentative examples and insufficient context.

| Failure                           | Appropriate intervention                |
| --------------------------------- | --------------------------------------- |
| Ambiguous task                    | Clarify instructions                    |
| Incorrect classification boundary | Add representative examples             |
| Missing output fields             | Define a schema                         |
| Fabricated evidence               | Strengthen grounding and abstention     |
| Complex reasoning errors          | Decompose the task or evaluate thinking |
| Excessive irrelevant output       | Specify scope and response constraints  |

### **Canonical Skeleton**

```
Observe Failure
        ↓
Reproduce Using Original Configuration
        ↓
Inspect Request, Context and Execution Trace
        ↓
Classify Failure
        │
        ├── Retrieval → Inspect Index and Ranking
        ├── Prompt    → Inspect Instructions
        ├── Model     → Evaluate Capability
        ├── Tools     → Inspect Schema and Execution
        └── Infra     → Inspect Errors and Latency
        ↓
Apply Targeted Fix
        ↓
Run Regression Evaluations
```

### **Best Practices**

- Inspect actual inputs, retrieved evidence and execution outcomes before changing models.
- Compare failing cases with successful cases to identify systematic patterns.
- Use explicit grounding, representative examples and uncertainty handling.
- Evaluate stronger models or reasoning configurations only when evidence indicates a capability limitation.
- Convert confirmed production failures into permanent regression test cases.

## **4.5 Optimize token usage, latency and cost-performance trade-offs**

### **Core Concept**

The objective is not to minimize tokens, latency or model costs independently. It is to maximize business value while satisfying quality, security and performance constraints.

⭐ **Select the least expensive configuration that reliably meets all mandatory requirements.**

### **Optimization techniques**

| Technique                  | Cost reduction               | Latency improvement                  | Main trade-off                        |
| -------------------------- | ---------------------------- | ------------------------------------ | ------------------------------------- |
| Smaller model              | Often significant            | Often significant                    | Potential quality loss                |
| Prompt caching             | Significant for reused input | Reduces repeated prefill work        | Requires reusable prefixes            |
| Message Batches API        | 50% token-price discount     | Not intended for interactive latency | Asynchronous processing               |
| Context pruning            | Reduces input tokens         | Often improves latency               | May remove relevant information       |
| Output length control      | Reduces output tokens        | Reduces generation time              | Potential loss of completeness        |
| Streaming                  | No inherent token savings    | Improves perceived responsiveness    | Partial output handling               |
| Parallel tool execution    | Workload-dependent           | Reduces sequential waiting           | Concurrency and dependency management |
| Model routing              | Reduces average cost         | Workload-dependent                   | Routing errors and complexity         |
| Reduced thinking effort    | Can reduce reasoning cost    | Can reduce latency                   | Potential reasoning degradation       |
| Progressive tool discovery | Reduces initial context      | Workload-dependent                   | Tool discovery overhead               |

### **⭐ Prompt caching**

Prompt caching reuses computation associated with a repeated prompt prefix.

It is particularly effective for workloads repeatedly sending:

- Large system prompts.
- Enterprise policies.
- Tool definitions.
- Reference documents.
- Few-shot examples.
- Repeated conversation history.

### **Correct prompt structure**

```
┌──────────────────────────────┐
│ Stable tool definitions      │
│ Stable system instructions   │
│ Shared reference documents   │
│ Few-shot examples            │
├────── Cache Breakpoint ──────┤
│ Dynamic user query           │
│ Request-specific information │
└──────────────────────────────┘
```

The stable prefix should precede frequently changing content.

The following example illustrates explicit cache control using the Python SDK:

```
response = client.messages.create(    model="claude-sonnet-4-6",    max_tokens=1024,    system=[        {            "type": "text",            "text": large_policy_document,            "cache_control": {                "type": "ephemeral"            }        }    ],    messages=[        {            "role": "user",            "content": user_question        }    ])
```

### **Prompt caching pricing**

For standard cache pricing applicable to established models such as Haiku 4.5 and Sonnet 4.6:

| Operation            | Relative input token price |
| -------------------- | -------------------------- |
| Ordinary input       | 1.0×                       |
| 5-minute cache write | 1.25×                      |
| 1-hour cache write   | 2.0×                       |
| Cache read           | 0.1×                       |

⭐ A cache hit can reduce the cost of the cached input tokens by 90%. It does not imply a 90% reduction in the entire request cost.

### **Cache behavior**

| Property                   | Exam-relevant behavior                           |
| -------------------------- | ------------------------------------------------ |
| Default TTL                | 5 minutes                                        |
| Extended TTL               | 1 hour, where supported                          |
| Explicit cache breakpoints | Up to 4                                          |
| Cache matching             | Reusable matching prefix                         |
| Minimum cacheable length   | Model-dependent                                  |
| Output caching             | Not provided by prompt caching                   |
| Cache hit refresh          | Extends cache lifetime under applicable behavior |

Prompt caching is not semantic caching. Two prompts that convey the same meaning but contain different prefix content do not necessarily share a cache entry.

See Anthropic's [prompt caching documentation](https://platform.claude.com/docs/en/build-with-claude/prompt-caching?utm_source=chatgpt.com) for model-specific eligibility and pricing.

### **Interactive prompt caching example**

## **Prompt caching cost comparison**

Illustrative Sonnet 4.6 input pricing: $3 per million tokens. Assumes one cache write and subsequent cache hits.

Number of requests

**10**

Reusable prefix tokens

**10,000**

Dynamic tokens per request

**500**

Cache lifetime

5 minutes1 hour

Without caching

# **$0.3150**

With caching

# **$0.0795**

Standard input cost

$0.3150

Cached input cost

$0.0795

## **74.8% input cost savings**

Excludes output tokens, tool charges, cache misses and other pricing modifiers.

### **⭐ Message Batches API**

The Message Batches API processes independent inference requests asynchronously at a discounted price.

It is suitable for:

- Large-scale offline evaluations.
- Historical document extraction.
- Bulk classification.
- Offline summarization.
- Dataset enrichment.

| Property                      | Value                      |
| ----------------------------- | -------------------------- |
| Processing model              | Asynchronous               |
| Standard token-price discount | 50%                        |
| Maximum requests per batch    | 100,000                    |
| Maximum batch size            | 256 MB                     |
| Processing expiration         | 24 hours                   |
| Typical completion            | Most batches within 1 hour |
| Result availability           | 29 days after creation     |
| Streaming                     | Not supported              |
| Request correlation           | `custom_id`                |

The request count and payload size limits apply together; whichever is reached first determines the maximum batch size.

### **Batch processing workflow**

```
Prepare Independent Requests
        ↓
Assign Unique custom_id Values
        ↓
Submit Message Batch
        ↓
Poll Processing Status
        ↓
Retrieve Results
        ↓
Match Results Using custom_id
        ↓
Evaluate / Store Successful Results
        ↓
Retry Eligible Failed Requests
```

Never assume results arrive in submission order.

Prompt caching can be combined with batch processing, although cache hits are best-effort because requests can execute concurrently or at different times.

See [Message Batches API documentation](https://platform.claude.com/docs/en/build-with-claude/batch-processing?utm_source=chatgpt.com).

### **⭐ Prompt caching vs. batch processing vs. streaming**

| Scenario                                            | Best choice                   |
| --------------------------------------------------- | ----------------------------- |
| Interactive chatbot with repeated enterprise policy | Prompt caching                |
| Overnight evaluation of 50,000 independent examples | Message Batches API           |
| User must start seeing an answer immediately        | Streaming                     |
| Repeated reference document in offline evaluations  | Batch processing with caching |
| Excessive generated text                            | Output length optimization    |
| Small, simple classification requests               | Evaluate a smaller model      |

### **Latency decomposition**

```
End-to-End Latency
    =
    Application / Network Overhead
    + Retrieval Time
    + Model Prefill Time
    + Model Generation Time
    + Tool Execution and Additional Model Turns
    + Post-processing Time
```

| Bottleneck                    | Appropriate optimization            |
| ----------------------------- | ----------------------------------- |
| High prefill time             | Prompt caching or context reduction |
| Slow token generation         | Faster model or shorter output      |
| Slow search                   | Optimize retrieval infrastructure   |
| Sequential independent tools  | Parallel execution                  |
| Excessive agent iterations    | Tool and loop optimization          |
| Poor perceived responsiveness | Streaming                           |
| High queue time               | Capacity and concurrency management |

**Streaming improves time to visible output, not necessarily total execution time.**

### **⭐ Model routing and escalation**

Rather than sending every request to the strongest available model, route workloads according to complexity and risk.

```
Incoming Request
        ↓
Task and Risk Classification
        │
        ├── Simple  → Haiku
        ├── Medium  → Sonnet
        └── Complex → Opus
                       │
                       ▼
               Output Validation
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Accepted       Escalate / Review
```

Escalation should be triggered by validated quality signals, task complexity or explicit risk criteria rather than relying solely on the model's self-reported confidence.

### **Extended and adaptive thinking trade-offs**

| Consideration     | Architectural implication                      |
| ----------------- | ---------------------------------------------- |
| Complex reasoning | Additional thinking may improve quality        |
| Simple extraction | Additional thinking may be unnecessary         |
| Latency           | More reasoning may increase response time      |
| Cost              | Reasoning contributes to billable output usage |
| Configuration     | Depends on model and supported API parameters  |

Do not assume every Claude model uses the same thinking parameters. Manual thinking budgets, adaptive thinking and effort controls are model-dependent.

### **Canonical Skeleton**

```
Establish Quality, Cost and Latency Baselines
        ↓
Identify Dominant Bottleneck
        ↓
Select Targeted Optimization
        │
        ├── Repeated Input → Caching
        ├── Offline Volume → Batch API
        ├── Model Cost     → Routing
        ├── Large Context  → Context Optimization
        ├── Slow Output    → Output Control
        └── Tool Overhead  → Tool Optimization
        ↓
Run Quality and Performance Evaluations
        ↓
Verify Mandatory Constraints
        ↓
Deploy and Monitor
```

### **Best Practices**

- Optimize based on measured bottlenecks instead of assumptions.
- Place reusable context before dynamic content to maximize cache reuse.
- Use asynchronous batches for workloads without immediate response requirements.
- Reduce irrelevant context, unnecessary output and redundant tool calls.
- Reevaluate quality whenever changing the model, reasoning configuration or context strategy.

## **4.6 Monitor system performance using logging and observability tools**

### **Core Concept**

Evaluation verifies expected behavior under controlled conditions. Observability helps explain actual behavior in production.

Production environments introduce changing workloads, document versions, tool failures, traffic spikes and previously unseen inputs.

⭐ **Monitor the entire application, not just the Claude API call.**

### **Three pillars of observability**

| Pillar  | Purpose                                 | Examples                                |
| ------- | --------------------------------------- | --------------------------------------- |
| Logs    | Record discrete events                  | Errors, tool calls, validation failures |
| Metrics | Measure aggregate behavior              | P95 latency, token usage, error rates   |
| Traces  | Reconstruct execution across components | Retrieval → Claude → tool → validation  |

### **Distributed tracing example**

```
Trace ID: request-4821
│
├── Authentication         20 ms
├── Query Classification   80 ms
├── Document Retrieval     450 ms
├── Reranking              120 ms
├── Claude Inference       950 ms
│   ├── Input tokens       8,420
│   ├── Output tokens      630
│   └── Cache hit          Yes
├── Tool Execution         300 ms
└── Output Validation      50 ms
```

A trace allows the architect to isolate the component responsible for high latency, errors or unexpected costs.

### **⭐ What to capture in production**

| Category       | Recommended telemetry                                   |
| -------------- | ------------------------------------------------------- |
| Correlation    | Request ID, trace ID, session ID                        |
| Configuration  | Model ID, prompt version, tool version                  |
| Retrieval      | Document IDs, index version, retrieval scores           |
| Performance    | Component latency, end-to-end latency, TTFT             |
| Usage          | Input, output and cache token counts                    |
| Reliability    | Errors, retries, timeouts, stop reasons                 |
| Agent behavior | Tool selection, execution outcomes, iteration count     |
| Quality        | Validation scores, sampled groundedness, task success   |
| Security       | Policy violations, denied actions, injection detections |
| Business       | User feedback, escalation rate, completed tasks         |

Avoid indiscriminately storing raw prompts, retrieved documents, access tokens, PII or other confidential information.

### **Claude API usage and diagnostic fields**

| Field                         | Meaning                                 |
| ----------------------------- | --------------------------------------- |
| `input_tokens`                | Ordinary uncached input tokens          |
| `output_tokens`               | Generated billable output usage         |
| `cache_creation_input_tokens` | Tokens written into the cache           |
| `cache_read_input_tokens`     | Tokens retrieved from the cache         |
| `stop_reason`                 | Reason generation ended                 |
| `request-id`                  | Provider request correlation identifier |
| `model`                       | Model identifier returned by the API    |

For the native Messages API usage object:

```
Total Input Tokens =
    input_tokens
    + cache_creation_input_tokens
    + cache_read_input_tokens
```

The three input categories have different billing rates when prompt caching is used.

### **Important stop reasons**

| Stop reason     | Interpretation                                 |
| --------------- | ---------------------------------------------- |
| `end_turn`      | Model completed its turn naturally             |
| `max_tokens`    | Output reached its configured generation limit |
| `tool_use`      | Model requests tool execution                  |
| `stop_sequence` | Configured stop sequence was encountered       |

Other stop reasons exist depending on model and API features.

A `max_tokens` stop reason can indicate incomplete output. A `tool_use` response is not necessarily a completed task.

### **⭐ Production monitoring tools**

| Tool                                          | Primary capability                                                     |
| --------------------------------------------- | ---------------------------------------------------------------------- |
| Claude Console                                | API usage and cost visibility                                          |
| Usage and Cost API                            | Programmatic usage and expenditure analysis, as available in July 2026 |
| Anthropic SDK and application instrumentation | Request-level usage and error capture                                  |
| OpenTelemetry                                 | Vendor-neutral metrics and distributed tracing                         |
| Azure Application Insights                    | Application and dependency monitoring                                  |
| Amazon CloudWatch                             | AWS application metrics and logs                                       |
| Google Cloud Monitoring                       | GCP infrastructure and application monitoring                          |
| Langfuse / Arize Phoenix                      | LLM-specific tracing and evaluation                                    |

OpenTelemetry and cloud monitoring services can be integrated through application-level instrumentation. Do not assume that every Claude deployment surface provides native tracing with identical capabilities.

### **Operational alert design**

| Alert                       | Potential response                           |
| --------------------------- | -------------------------------------------- |
| P95 latency exceeds SLO     | Inspect component-level traces               |
| Cache hit rate drops        | Inspect prefix changes and expiration        |
| Cost per task increases     | Inspect model routing and agent iterations   |
| Retrieval recall declines   | Inspect indexing and document refresh        |
| Groundedness score declines | Evaluate retrieval and generation separately |
| Unauthorized tool action    | Block affected workflow and investigate      |
| HTTP 429 errors increase    | Inspect rate limits and concurrency          |
| HTTP 529 errors increase    | Apply overload handling and bounded retries  |

Thresholds should reflect workload-specific baselines and service requirements.

### **⭐ Monitoring and continuous evaluation**

Production telemetry should feed the evaluation lifecycle.

```
Production Traffic
        ↓
Logs + Metrics + Traces
        ↓
Quality Sampling and Alerting
        ↓
Failure Investigation
        ↓
Sanitized Failure Examples
        ↓
Golden / Regression Dataset
        ↓
Targeted Improvement
        ↓
Controlled Deployment
        ↓
Production Monitoring
```

### **Canonical Skeleton**

```
Claude Application
        ↓
Instrumentation Layer
        │
        ├── Structured Logs
        ├── Metrics
        ├── Distributed Traces
        └── Evaluation Scores
        ↓
Observability Platform
        ↓
Dashboards + SLO Alerts
        ↓
Root Cause Analysis
        ↓
Incident Resolution + Regression Tests
```

### **Best Practices**

- Instrument individual stages and propagate correlation identifiers across services.
- Monitor quality, usage, latency, errors and security events together.
- Protect sensitive telemetry through redaction, access controls and retention policies.
- Sample production outputs for automated evaluation and human calibration.
- Use production incidents to strengthen regression suites and deployment gates.

# **Part 2: Consolidated Exam Preparation**

# **5. Exam Traps**

These are plausible distractors in architect-level scenario questions. They are not actual exam questions.

| Trap                                                      | Why it is tempting                            | Why it is wrong                                                              |
| --------------------------------------------------------- | --------------------------------------------- | ---------------------------------------------------------------------------- |
| ⭐ Upgrade to Opus whenever accuracy declines              | Stronger models generally handle harder tasks | The failure may originate in retrieval, prompts or tools                     |
| ⭐ Optimize before building evaluations                    | Appears to accelerate development             | Improvement cannot be objectively established                                |
| Use accuracy as the only metric                           | Simple to understand                          | Ignores latency, cost, safety and business impact                            |
| Use average latency exclusively                           | Easy to calculate                             | Conceals slow requests and tail latency                                      |
| Use exact match for every generated response              | Deterministic and inexpensive                 | Rejects semantically correct alternative answers                             |
| Use an LLM judge for everything                           | Flexible and scalable                         | Deterministic validation is cheaper and more reliable for objective criteria |
| Use human reviewers for every test                        | Appears to maximize quality                   | Expensive and unsuitable for frequent large-scale regression testing         |
| Trust an uncalibrated LLM judge                           | Produces convenient numerical scores          | Judge biases can distort conclusions                                         |
| Tune prompts using the held-out test set                  | Improves measured test performance            | Causes evaluation overfitting                                                |
| Test only happy-path scenarios                            | Generates reassuring results                  | Misses dangerous and expensive failure modes                                 |
| ⭐ Change multiple variables in an A/B test                | May produce faster improvements               | Prevents attribution of improvement to a specific change                     |
| Deploy based on the first positive experiment result      | Reduces rollout time                          | Ignores statistical uncertainty and guardrail metrics                        |
| ⭐ Use the Batch API for interactive chat                  | Offers a 50% discount                         | Asynchronous processing cannot satisfy interactive response requirements     |
| Assume streaming reduces token costs                      | Responses appear faster                       | Streaming changes delivery, not inherently token usage                       |
| ⭐ Put dynamic content before static cached content        | Natural request construction                  | Reduces reusable prefix length and cache effectiveness                       |
| Assume prompt caching caches final answers                | Confuses it with application response caching | Prompt caching reuses input computation                                      |
| Assume a 90% cache discount applies to the entire request | Simplifies cost estimation                    | The discount applies to eligible cached input tokens                         |
| Enable maximum thinking for every request                 | More reasoning seems beneficial               | Can introduce unnecessary latency and cost                                   |
| Increase retrieval top-K indefinitely                     | More evidence appears useful                  | Can add noise, distract the model and increase token costs                   |
| Set temperature to zero to eliminate hallucinations       | Lower randomness improves consistency         | Consistency does not establish factual correctness                           |
| Trust citations without checking evidence                 | Citations appear authoritative                | A citation may not support the associated claim                              |
| ⭐ Grade agents using only their final answer              | Easy to automate                              | The agent may falsely report successful execution                            |
| Require identical agent trajectories                      | Simplifies test assertions                    | Multiple valid execution paths may exist                                     |
| Treat `pass@k` as a consistency metric                    | Higher values appear reassuring               | It measures whether at least one attempt succeeds                            |
| Log every raw prompt and tool response                    | Maximizes debugging information               | May expose confidential data and credentials                                 |
| Monitor only the model API                                | Claude appears to be the central component    | Retrieval, orchestration and tools also fail                                 |

# **6. Key Facts to Memorize**

## **6.1 Evaluation metrics and definitions**

| Metric                   | Definition                                                   |
| ------------------------ | ------------------------------------------------------------ |
| Precision                | Fraction of predicted positives that are correct             |
| Recall                   | Fraction of actual positives successfully identified         |
| F1                       | Harmonic mean of precision and recall                        |
| Precision\@K             | Fraction of top-K retrieved items that are relevant          |
| Recall\@K                | Fraction of all relevant items retrieved in the top K        |
| MRR                      | Mean reciprocal rank of the first relevant result            |
| nDCG                     | Ranking quality accounting for relevance and position        |
| Faithfulness             | Whether generated claims are supported by supplied evidence  |
| Groundedness             | Degree to which an answer is anchored to trusted sources     |
| Task success rate        | Successful tasks divided by attempted tasks                  |
| TTFT                     | Time to first token                                          |
| P95 latency              | Latency at or below which approximately 95% of requests fall |
| Cost per successful task | Total execution cost divided by successful tasks             |
| False refusal rate       | Legitimate requests incorrectly rejected                     |
| Attack success rate      | Successful adversarial attempts divided by tested attempts   |

## **6.2 Evaluation methodology**

| Requirement                        | Preferred approach             |
| ---------------------------------- | ------------------------------ |
| Objective correctness              | Code-based grading             |
| Semantic correctness               | Calibrated model-based grading |
| Subjective or high-stakes judgment | Human expert review            |
| Mixed enterprise workloads         | Hybrid grading framework       |
| Repeatable development comparisons | Fixed evaluation dataset       |
| Previously working behavior        | Regression evaluation          |
| New desired behavior               | Capability evaluation          |
| Actual user impact                 | Online A/B testing             |
| Low-risk production validation     | Shadow testing                 |
| Limited deployment exposure        | Canary rollout                 |

## **6.3 Agent evaluation**

| Concept            | Memorize                                                |
| ------------------ | ------------------------------------------------------- |
| Task               | An evaluation scenario with expected success conditions |
| Trial              | One execution of a task                                 |
| Grader             | Component assessing execution or outcome                |
| Transcript         | Record of agent interactions and actions                |
| Outcome            | Final result or verified environmental state            |
| Evaluation harness | Infrastructure that executes and grades evaluations     |
| `pass@1`           | First-attempt success                                   |
| `pass@k`           | At least one success across k attempts                  |
| `pass^k`           | Success in every one of k attempts                      |

For independent trials with success probability \\(p\\):

\\[ \operatorname{pass\@k}=1-(1-p)^k \\]

\\[ \operatorname{pass^k}=p^k \\]

## **6.4 Prompt caching and batch limits**

| Fact                                 | Value                     |
| ------------------------------------ | ------------------------- |
| ⭐ Default cache TTL                  | 5 minutes                 |
| Extended cache TTL                   | 1 hour, where supported   |
| Standard 5-minute write multiplier   | 1.25×                     |
| Standard 1-hour write multiplier     | 2×                        |
| ⭐ Standard cache-read multiplier     | 0.1×                      |
| Explicit cache breakpoints           | Up to 4                   |
| Minimum cacheable tokens             | Model-dependent           |
| ⭐ Message Batches API discount       | 50%                       |
| Maximum batch requests               | 100,000                   |
| Maximum batch payload                | 256 MB                    |
| Batch expiration                     | 24 hours                  |
| Batch result retention               | 29 days                   |
| Batch result correlation             | `custom_id`               |
| Recommended starting agent eval size | Approximately 20–50 tasks |

The recommended evaluation dataset size is a starting point from Anthropic's agent evaluation guidance, not a platform limit or mandatory exam threshold.

## **6.5 Claude model selection and pricing**

The following models are established examples available within the July 2026 scope. They are not an exhaustive model catalog.

| Model      | Input / million tokens | Output / million tokens | Typical workload                           |
| ---------- | ---------------------- | ----------------------- | ------------------------------------------ |
| Haiku 4.5  | $1                     | $5                      | Classification, routing, simple extraction |
| Sonnet 4.6 | $3                     | $15                     | General enterprise applications            |
| Opus 4.6   | $5                     | $25                     | Complex reasoning and agentic workflows    |

These are standard base API prices, excluding applicable caching, batch, tool and platform-specific modifiers.

⭐ Memorize the **model selection principle** more strongly than the individual price figures: benchmark models against the real workload and choose the lowest-cost configuration that satisfies the requirements.

## **6.6 Important tools and features**

| Tool or feature                | Primary purpose                                                 |
| ------------------------------ | --------------------------------------------------------------- |
| Claude Console Evaluation tool | Prompt testing and comparison                                   |
| Messages API                   | Synchronous inference                                           |
| Message Batches API            | Discounted asynchronous inference                               |
| Token Counting API             | Estimate input token usage before inference                     |
| Prompt caching                 | Reduce repeated input-processing overhead                       |
| Structured outputs             | Enforce supported output schemas                                |
| Extended/adaptive thinking     | Improve performance on suitable reasoning tasks                 |
| Streaming                      | Deliver incremental output                                      |
| Tool Search Tool               | Discover relevant deferred tools                                |
| Programmatic tool calling      | Reduce orchestration and context overhead in suitable workflows |
| Usage and Cost API             | Analyze API consumption and expenditure                         |
| OpenTelemetry                  | Instrument distributed application behavior                     |

Availability and supported parameters can vary by model, API version and deployment platform.

## **6.7 Cost optimization formulas**

### **Ordinary request**

```
Request Cost =
    Input Token Cost
    + Output Token Cost
    + Applicable Tool Charges
```

### **Cached request**

```
Request Cost =
    Ordinary Input Cost
    + Cache Write Cost
    + Cache Read Cost
    + Output Cost
    + Applicable Tool Charges
```

### **Business-oriented cost measurement**

```
Cost per Successful Task =
    Total Execution Cost / Successful Tasks
```

Cost per successful task is often more useful than cost per API call.

A cheap model that requires multiple retries and frequent human corrections can be more expensive overall than a stronger model.

## **6.8 Always/Never decision rules**

| Always                                    | Never                                                       |
| ----------------------------------------- | ----------------------------------------------------------- |
| Define success criteria before optimizing | Assume that a larger model fixes every failure              |
| Diagnose the root cause                   | Change multiple components when trying to isolate causality |
| Verify actual agent outcomes              | Trust an agent's success claim without validation           |
| Maintain regression evaluations           | Tune against the final held-out dataset                     |
| Measure quality and operational metrics   | Select a model solely from public benchmarks                |
| Protect sensitive telemetry               | Indiscriminately log confidential data                      |
| Validate critical output and actions      | Treat schema compliance as proof of factual accuracy        |
| Check all mandatory release constraints   | Sacrifice mandatory safety requirements for cost savings    |

# **7. Exam-Style Questions**

The following five original practice questions collectively cover all six task statements. Their difficulty and emphasis are study choices rather than an official question distribution.

### **Domain 4 practice exam**

0 of 5 reviewed

Question 1 · 4.1 and 4.2 — Evaluation metrics and mixed methodologies

**An enterprise contract intelligence application extracts contractual obligations using Claude. The application produces valid JSON, but occasionally omits obligations or invents dates. The organization wants to evaluate 20,000 documents every week while maintaining low evaluation costs. Which evaluation strategy is BEST?**

A. Use exact string matching against a single reference JSON response for every document.

B. Use deterministic schema validation, field-level precision and recall, source-grounded LLM grading, and sampled expert reviews.

C. Use an Opus model to assign an overall quality score from 1 to 10 without an explicit rubric.

D. Require legal experts to manually review all 20,000 documents every week.

Check answer

Question 2 · 4.4 — System failure diagnosis

**A team maintains a production RAG assistant using Claude. After a scheduled document indexing refresh, users report confident but inaccurate responses. The system prompt, model version and generation parameters have not changed. Which action should the architect take FIRST?**

A. Upgrade all requests to the strongest available Opus model.

B. Increase the number of retrieved documents from 5 to 50.

C. Set the temperature to zero and increase the maximum output tokens.

D. Inspect document versions and retrieved chunks, then test the model using known-correct reference context.

Check answer

Question 3 · 4.5 — Cost and latency optimization

**An enterprise chatbot sends the same 25,000-token policy document with every request, followed by a short employee question. The application must provide interactive responses within its latency SLO. Management wants to reduce costs without changing model quality. Which approach is BEST?**

A. Place the stable policy document before the dynamic question and enable prompt caching.

B. Move all requests to the Message Batches API to obtain a 50% discount.

C. Replace the policy document with its first 2,000 tokens.

D. Enable streaming and assume it will reduce the number of billable input tokens.

Check answer

Question 4 · 4.3 — A/B testing and iterative improvement

**An AI team introduces a revised system prompt that improves offline evaluation accuracy from 91% to 95%. However, it also increases output length. The team wants to determine whether the change improves real-world user outcomes. Which approach is BEST?**

A. Deploy the revised prompt to every user because offline accuracy improved.

B. Change the prompt and model simultaneously, then compare average response quality.

C. Run a randomized controlled A/B test while keeping the model and other configuration fixed; measure task success, latency, cost and safety.

D. Ask five employees which prompt feels better and deploy their preferred version.

Check answer

Question 5 · 4.6 and 4.2 — Observability and agent outcome evaluation

**An enterprise support agent uses Claude and multiple CRM tools. Aggregate API success rates remain above 99%, but users report that the agent sometimes claims to have created support tickets that do not exist. The operations team cannot identify the failing component. Which solution is BEST?**

A. Increase the model's thinking budget for every support request.

B. Implement distributed tracing across the agent and CRM tools, verify ticket existence, monitor task-level success and add failure cases to regression evaluations.

C. Track only Claude API latency and HTTP response status.

D. Log all raw prompts, CRM credentials and customer records to provide complete debugging context.

Check answer

## **Answer key**

| Question | Correct answer | Primary concept                                |
| -------- | -------------- | ---------------------------------------------- |
| 1        | B              | Mixed evaluation methodologies                 |
| 2        | D              | Root cause analysis and oracle context testing |
| 3        | A              | Prompt caching                                 |
| 4        | C              | Controlled A/B testing                         |
| 5        | B              | Observability and verified agent outcomes      |

# **8. High-Yield Architectural Decision Matrix**

Use this matrix when eliminating plausible distractors in scenario-based questions.

| Scenario contains...                             | Think first of...                               |
| ------------------------------------------------ | ----------------------------------------------- |
| Repeated large system prompt                     | Prompt caching                                  |
| Thousands of non-urgent inference requests       | Message Batches API                             |
| Users waiting for visible responses              | Streaming                                       |
| Excessive P95 latency                            | Component-level tracing                         |
| Accurate but expensive solution                  | Model routing and token optimization            |
| Poor performance after document refresh          | Retrieval and indexing                          |
| Relevant evidence available but answer incorrect | Prompt or reasoning diagnosis                   |
| Complex reasoning fails despite correct context  | Model capability or thinking configuration      |
| JSON formatting failures                         | Structured outputs and deterministic validation |
| Subjective response evaluation                   | Rubric-based LLM judge                          |
| High-risk legal or compliance judgment           | Human expert review                             |
| Multiple valid agent execution paths             | Outcome-based grading                           |
| Agent reports an action succeeded                | Verify external system state                    |
| Reliability across repeated trials               | `pass^k`                                        |
| At least one successful solution is required     | `pass@k`                                        |
| Candidate prompt outperforms baseline offline    | Controlled A/B testing                          |
| Production validation without user exposure      | Shadow testing                                  |
| Limited-risk production release                  | Canary deployment                               |
| Unexplained cost increase                        | Token, cache and agent-loop telemetry           |
| Repeated production failure                      | Add a regression test                           |
| Higher accuracy but violated mandatory SLA       | Reject or further optimize configuration        |

# **9. Recommended Anthropic Reading**

Prioritize these materials for Domain 4. The documentation is maintained continuously, so use only features and model configurations applicable by July 2026.

| Priority | Resource                                                                                                                                          | Focus                                                    |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| ⭐⭐⭐      | [Demystifying evals for AI agents](https://www.anthropic.com/research/demystifying-evals-for-ai-agents?utm_source=chatgpt.com)                    | Graders, agent outcomes, regression, reliability metrics |
| ⭐⭐⭐      | [Define success criteria and build evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests?utm_source=chatgpt.com)       | Evaluation design and success criteria                   |
| ⭐⭐⭐      | [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching?utm_source=chatgpt.com)                                     | Prefix structure, pricing and cache behavior             |
| ⭐⭐⭐      | [Message Batches API](https://platform.claude.com/docs/en/build-with-claude/batch-processing?utm_source=chatgpt.com)                              | Asynchronous inference and cost reduction                |
| ⭐⭐       | [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations?utm_source=chatgpt.com) | Grounding and verification                               |
| ⭐⭐       | [Contextual Retrieval](https://www.anthropic.com/research/contextual-retrieval?utm_source=chatgpt.com)                                            | Retrieval quality and RAG optimization                   |
| ⭐⭐       | [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents?utm_source=chatgpt.com)                       | Tool evaluation and optimization                         |
| ⭐⭐       | [Advanced tool use](https://www.anthropic.com/research/advanced-tool-use?utm_source=chatgpt.com)                                                  | Tool discovery and context efficiency                    |

# **10. Last-Minute Revision Checklist**

- ⭐ Define measurable success criteria before changing prompts, models or architectures.
- ⭐ Understand precision, recall, F1, groundedness, P95 latency and cost per successful task.
- ⭐ Differentiate code-based, model-based and human evaluation.
- Understand golden datasets, regression suites, judge calibration, `pass@k` and `pass^k`.
- Differentiate offline evaluation, A/B testing, shadow testing and canary deployment.
- ⭐ Diagnose retrieval, prompt, model and tool failures independently.
- ⭐ Memorize prompt caching: stable prefix, 5-minute TTL, 1-hour option, standard 0.1× cache-read pricing.
- ⭐ Memorize Message Batches API: asynchronous, 50% discount, 100,000 requests, 256 MB, 24-hour expiration.
- Understand model routing, thinking trade-offs, streaming and context optimization.
- Monitor end-to-end task success, distributed traces, token costs, security events and production regressions.
