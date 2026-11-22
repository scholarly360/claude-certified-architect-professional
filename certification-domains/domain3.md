# **Claude Certified Architect – Professional (CCA-P)**

## **Domain 3: Integration (19%) — High-Yield Study Guide**

**Scope:** July 2026 | **Difficulty:** Professional / Solution Architect | **Focus:** Architecture decisions, integration trade-offs, production security, and operational reliability.

Domain 3 is the highest-weighted domain in this exam blueprint. It tests whether you can integrate Claude into enterprise systems while controlling security exposure, retrieval quality, latency, cost, and operational complexity.

⭐ **Core exam principle:** Choose the simplest integration architecture that meets the functional requirements, enforces least privilege, and satisfies measurable quality and performance targets.

# **Part 1: Per Sub-Domain Breakdown**

## **3.1 Evaluate tool/agent configuration for capability bloat**

### **Core Concept**

**Capability bloat** occurs when an agent has more tools, integrations, permissions, or operational capabilities than its assigned responsibilities require.

For example, an HR assistant designed to retrieve employee policies should not have access to payroll modification, account deletion, or employee termination tools.

Excessive capabilities introduce three problems:

| Dimension        | Impact                                                                   |
| ---------------- | ------------------------------------------------------------------------ |
| Security         | Larger attack surface; unnecessary read/write privileges                 |
| Accuracy         | More opportunities for incorrect tool selection                          |
| Cost and latency | Larger tool definitions consume tokens and can complicate tool selection |
| Maintainability  | More schemas, dependencies, failure modes, and permissions to manage     |

### **Decision Criteria**

- ⭐ **Remove unnecessary capabilities entirely.** Logging, instructions, and confirmation prompts cannot provide the same protection as eliminating access.
- Prefer a small set of task-specific tools over exposing every tool in an enterprise application.
- Use progressive tool discovery when the approved tool catalog is large but individual tasks need only a subset.
- Separate read-only and state-changing capabilities, with stronger authorization for the latter.
- Evaluate tool-selection accuracy, not merely whether the agent can invoke tools successfully.

### **Canonical Skeleton**

```
Business Use Case
       |
       v
Identify Required Capabilities
       |
       v
Inventory Tools and Permissions
       |
       v
Remove Unnecessary Tools
       |
       v
Assign Least-Privilege Credentials
       |
       v
Define Clear Tool Schemas
       |
       v
Test Tool Selection + Authorization
       |
       v
Deploy + Monitor + Periodic Review
```

**Example: HR policy assistant**

| Tool                     | Required?   | Decision                   |
| ------------------------ | ----------- | -------------------------- |
| `search_hr_policy`       | Yes         | Allow                      |
| `get_leave_balance`      | Conditional | Allow for authorized users |
| `modify_salary`          | No          | Remove                     |
| `terminate_employee`     | No          | Remove                     |
| `delete_employee_record` | No          | Remove                     |

### **Best Practices**

1. ⭐ Expose only the minimum capabilities needed for the workflow; avoid broad, general-purpose administrative tools.
2. Give tools descriptive names, unambiguous descriptions, and well-defined input schemas.
3. Return concise, relevant tool results rather than entire database records or long API responses.
4. Use tool-selection evaluations and test cases to identify confused or unnecessary tool invocations.
5. Apply allowlists, server-side authorization, and tool-level restrictions to limit the consequences of mistakes.

Anthropic emphasizes well-defined, context-efficient tools and evaluation-driven refinement.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+1



**⭐ Exam takeaway:** Removing an unnecessary privileged tool is preferable to keeping it behind stronger prompting or an approval dialog.

## **3.2 Analyze authentication and authorization requirements to identify security gaps**

### **Core Concept**

Enterprise Claude integrations must distinguish **authentication (AuthN)** from **authorization (AuthZ)**.

| Concept                 | Question answered                        | Example                                                               |
| ----------------------- | ---------------------------------------- | --------------------------------------------------------------------- |
| Authentication          | Who is making the request?               | Employee authenticated through Microsoft Entra ID                     |
| Authorization           | What can that identity access or do?     | Employee can read their own payslip, not another employee's           |
| Least privilege         | What is the minimum access required?     | Read-only HR policy permissions                                       |
| Delegated authorization | On whose behalf is the action performed? | Assistant retrieves documents under the requesting user's permissions |
| Auditability            | Who performed which action, and when?    | User, tool, resource, action, result, correlation ID                  |

**Critical distinction:** A Claude API key authenticates access to Claude. It does not authorize the resulting agent to read or modify enterprise resources.

### **Security Gaps to Recognize**

**1. Shared service-account privilege**

An assistant uses a powerful backend service account to access every employee's records.

The application may authenticate users correctly but still expose resources through an overprivileged backend.

**2. Authorization through prompting**

A system prompt tells Claude not to access confidential documents.

This does not constitute an enforceable access-control boundary.

**3. Missing downstream authorization**

A user provides an employee ID, and Claude invokes `get_employee(id)` without the backend checking whether that user may access the record.

This is a form of broken object-level authorization.

**4. Privileged MCP integration**

A remote MCP server exposes sensitive tools using credentials with broader scopes than its users need.

An authenticated connection alone does not make every tool invocation authorized.

**5. Prompt-injection-driven tool misuse**

Retrieved documents or tool results contain instructions designed to trigger unauthorized tool calls.

External content must remain lower trust than system policy and authorization controls.

### **Canonical Skeleton**

```
Enterprise User
      |
      v
SSO / Identity Provider
      |
      v
Application Backend
      |
      +-- Validate User Identity
      |
      +-- Resolve Roles, Groups, Tenant
      |
      +-- Authorize Requested Operation
      |
      v
Claude API / Agent
      |
      v
Tool Gateway / MCP Server
      |
      +-- Verify Caller Identity
      |
      +-- Enforce Resource-Level Permissions
      |
      +-- Validate Parameters
      |
      +-- Audit Sensitive Actions
      |
      v
Enterprise System
(SAP / CRM / HRMS / Database)
```

The model can propose an operation, but the trusted integration layer decides whether that operation is permitted.

### **MCP Authorization Considerations**

For remote, protected MCP services, evaluate OAuth-based authorization, token validity, scopes, and audience restrictions.

In an OAuth-enabled MCP architecture:

- The MCP server validates tokens intended for that server.
- The caller should obtain only the scopes required for the operation.
- The server must reject tokens issued for unrelated resources.
- Downstream service access should use appropriately scoped credentials rather than blindly forwarding tokens.

**Important:** MCP is an integration protocol, not an automatic security boundary. Local and remote MCP deployments can have different authentication arrangements.

### **Best Practices**

1. ⭐ Enforce authorization in the backend, MCP server, or underlying resource service—not solely in Claude's instructions.
2. Use least-privilege service identities or delegated user credentials, with resource-level permission checks.
3. Keep API keys, OAuth tokens, and other secrets out of prompts and model-visible tool results.
4. Require explicit user confirmation or human approval for sensitive state-changing actions, in addition to backend authorization.
5. Log authorization decisions and administrative actions without exposing credentials or unnecessary personal data.

Anthropic's MCP integration capabilities support authenticated connections and tool-level configuration, but application owners remain responsible for enforcing their own security policies.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



**⭐ Exam takeaway:** Authentication proves identity. Authorization must be enforced at the resource boundary. Prompt instructions are neither.

## **3.3 Evaluate accuracy-latency trade-offs and justify configuration decisions**

### **Core Concept**

A production Claude architecture balances three competing objectives:

\\[ \text{Quality} \longleftrightarrow \text{Latency} \longleftrightarrow \text{Cost} \\]

Increasing reasoning effort, retrieval depth, or the number of tool interactions can improve accuracy, but may also increase response time and expense.

The architect's responsibility is not simply to select the fastest configuration. It is to select the **lowest-cost configuration that meets the application's quality threshold and latency SLO**.

### **Decision Matrix**

| Configuration               | Accuracy impact                           | Latency impact                     | Best use                                         |
| --------------------------- | ----------------------------------------- | ---------------------------------- | ------------------------------------------------ |
| Smaller/faster Claude model | May reduce quality on complex tasks       | Usually lower                      | Classification, routing, simple extraction       |
| More capable Claude model   | Potentially better reasoning and tool use | Often higher                       | Complex multi-step tasks                         |
| Extended thinking           | Can improve difficult reasoning           | More reasoning time and tokens     | Complex analysis where available                 |
| Larger retrieval `top_k`    | May improve recall; can add noise         | More retrieval/context processing  | Complex RAG questions                            |
| Reranking                   | Often improves relevance                  | Adds processing latency            | Precision-sensitive search                       |
| Prompt caching              | Preserves quality with reused context     | Reduces repeated-prefix processing | Stable system instructions and reference context |
| Parallel independent tools  | Same intended task coverage               | Reduces critical-path latency      | Multiple independent data sources                |
| Streaming                   | No inherent accuracy improvement          | Improves perceived responsiveness  | Interactive applications                         |
| Batch processing            | Same underlying model capability          | Asynchronous, not interactive      | Bulk extraction and offline processing           |

### **Canonical Skeleton**

```
Define Quality and Performance SLOs
                 |
                 v
Create Representative Test Dataset
                 |
                 v
Measure Baseline
(Quality, p50/p95 latency, Cost)
                 |
                 v
Identify Bottleneck
                 |
          +------+------+
          |             |
          v             v
    Model/Prompt    Retrieval/Tools
    Optimization    Optimization
          |             |
          +------+------+
                 |
                 v
Run Comparative Evaluations
                 |
                 v
Check Quality Thresholds
                 |
                 v
Select Best Configuration
                 |
                 v
Monitor Production Performance
```

### **Worked Example**

An enterprise research assistant has a 10-second p95 response-time SLO.

Its current pipeline requires:

- Query embedding and retrieval
- Sequential calls to three independent sources
- Reranking
- Claude generation

The three source calls can execute independently.

**Best initial optimization:** Parallelize those independent calls, then benchmark end-to-end quality and latency.

Switching to a smaller model or removing reranking may also reduce latency, but those changes risk sacrificing output quality. Parallelization addresses the orchestration bottleneck without deliberately reducing retrieved evidence.

### **Best Practices**

1. ⭐ Establish measurable acceptance criteria for task success, p95 latency, error rate, and per-request cost.
2. Optimize the demonstrated bottleneck before changing models or removing useful context.
3. Cache reusable prompt prefixes and minimize unnecessary tool calls and large intermediate results.
4. Parallelize genuinely independent operations; preserve sequential execution when operations have dependencies.
5. Validate optimizations against representative evaluations rather than assuming faster always means better.

Anthropic identifies tool search, programmatic tool calling, caching, and batching as different optimizations for different bottlenecks.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+2



**⭐ Exam takeaway:** Optimize under constraints. A configuration that is cheaper and faster but violates accuracy requirements is not the best architecture.

## **3.4 Analyze observability challenges and select monitoring strategies at scale**

### **Core Concept**

Traditional application observability focuses on requests, exceptions, infrastructure utilization, and service latency.

Claude-based applications introduce additional failure modes:

- An API request succeeds, but the final answer is incorrect.
- The agent selects the wrong tool.
- Retrieval returns irrelevant or stale information.
- A multi-agent workflow loops unnecessarily.
- Output quality deteriorates after a prompt, index, or model change.
- Token consumption increases without additional business value.

⭐ **A successful HTTP 200 response does not mean the AI task succeeded.**

Production observability should connect model behavior, tool execution, retrieval quality, and business outcomes.

### **Canonical Skeleton**

```
User Request
     |
     v
Application Gateway
     |
     +---- Trace ID / Correlation ID
     |
     v
Agent Orchestrator
     |
     +---- Model Call Metrics
     |
     +---- Tool Call Traces
     |
     +---- Retrieval / Reranking Metrics
     |
     +---- Token and Cost Metrics
     |
     +---- Authorization / Safety Events
     |
     v
Response + Feedback
     |
     v
Telemetry Pipeline
     |
     v
Dashboards / Alerts / Evaluation Jobs
     |
     v
Root-Cause Analysis
     |
     v
Architecture Improvements
```

### **Four Observability Layers**

| Layer             | Key metrics                                                | What they reveal                           |
| ----------------- | ---------------------------------------------------------- | ------------------------------------------ |
| Application       | Request success, p50/p95/p99 latency, availability         | Operational performance                    |
| Model             | Input/output tokens, cost, model ID, refusals, cache usage | Model consumption and behavior             |
| Agent/tool        | Tool selection accuracy, tool errors, retries, loop count  | Orchestration reliability                  |
| Retrieval/quality | Recall\@K, retrieval relevance, groundedness, task success | Whether responses have sufficient evidence |

### **Metrics Worth Knowing**

**Performance**

- End-to-end latency and time to first token (TTFT)
- p50, p95, p99 latency
- Timeout rate, retry rate, and error rate

**Cost**

- `input_tokens`
- `output_tokens`
- `cache_creation_input_tokens`
- `cache_read_input_tokens`
- Cost per completed task

**Agent behavior**

- Correct-tool selection rate
- Tool success/failure rate
- Number of tool calls per task
- Agent completion rate
- Human escalation rate

**RAG**

- Recall\@K
- Precision\@K
- Relevance of retrieved chunks
- Citation correctness
- Groundedness and answer correctness

These are a combination of API-provided usage fields and application-defined metrics. Claude does not automatically calculate every metric listed.

### **Root-Cause Diagnosis**

| Symptom                                    | Investigate first                                                 |
| ------------------------------------------ | ----------------------------------------------------------------- |
| Latency increases after adding MCP servers | Tool discovery, schemas, network calls                            |
| Answers deteriorate after document updates | Ingestion, indexing, retrieval, stale chunks                      |
| Token cost increases                       | Repeated context, large tool results, cache misses                |
| Agent repeatedly calls one tool            | Tool description, stopping conditions, orchestration              |
| API failures increase during peak load     | Rate limits, retries, concurrency                                 |
| Unauthorized data appears in results       | Retrieval authorization, tenant isolation, permission propagation |

### **Best Practices**

1. ⭐ Instrument the complete workflow, not only the Claude API call.
2. Propagate correlation IDs across model calls, tools, retrieval services, and downstream systems.
3. Track both technical SLOs and outcome quality; use automated evaluators and sampled human review when appropriate.
4. Collect privacy-preserving traces and structured metadata, avoiding unnecessary storage of sensitive prompts, tool results, and credentials.
5. Alert on meaningful deviations: error rates, p95 latency, cost per successful task, and quality regressions.

Anthropic's production multi-agent research experience emphasizes monitoring agent decision patterns and interaction structures while protecting conversation privacy.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+2



**⭐ Exam takeaway:** Monitor the system as a collection of model, retrieval, tool, and orchestration components. Logs alone cannot establish answer quality.

## **3.5 Design a RAG pipeline with appropriate chunking and indexing strategies**

### **Core Concept**

**Retrieval-Augmented Generation (RAG)** supplies Claude with relevant information from external knowledge sources at inference time.

Instead of requiring the model to remember proprietary or frequently changing information, RAG retrieves evidence and places it in the model's context.

For enterprise systems, RAG architecture must address:

- Document ingestion and parsing
- Chunking strategy
- Metadata and index design
- Retrieval quality
- Authorization
- Context construction
- Source attribution
- Data freshness

### **Canonical Skeleton**

```
               OFFLINE INDEXING
                      |
Documents / PDFs / Knowledge Base
                      |
                      v
            Parse and Normalize
                      |
                      v
          Structure-Aware Chunking
                      |
                      v
          Add Metadata and Context
                      |
             +--------+--------+
             |                 |
             v                 v
       Embedding Model     Lexical Index
             |                 |
             v                 v
        Vector Index        BM25 Index


                ONLINE QUERY
                      |
                User Question
                      |
                      v
             Authenticate User
                      |
                      v
                Query Processing
                      |
                      v
          ACL-Filtered Hybrid Search
                      |
                      v
              Candidate Retrieval
                      |
                      v
                   Rerank
                      |
                      v
             Construct Context
                      |
                      v
             Claude Generation
                      |
                      v
         Answer + Source Attribution
                      |
                      v
          Feedback and Evaluation
```

### **A. Chunking Strategies**

Chunking determines how documents are divided into retrievable units.

| Strategy        | Description                                             | Best suited for                          | Main limitation                              |
| --------------- | ------------------------------------------------------- | ---------------------------------------- | -------------------------------------------- |
| Fixed-size      | Split by token/character count                          | Uniform text, quick baselines            | May break semantic boundaries                |
| Recursive       | Split by paragraph, sentence, then smaller units        | General documents                        | Does not fully understand document structure |
| Semantic        | Split near changes in meaning                           | Narrative or conceptual content          | Extra preprocessing cost                     |
| Structure-aware | Split by headings, sections, clauses, tables            | Legal, financial, policy documents       | Requires reliable document parsing           |
| Parent-child    | Retrieve small chunks, expand to larger parent sections | Long manuals and complex policies        | More storage and retrieval logic             |
| Contextual      | Add explanatory document context to each chunk          | Chunks dependent on surrounding material | Additional preprocessing cost                |

**Example: Contract analysis**

Suppose a contract has these sections:

```
Agreement
 ├── Definitions
 ├── Payment Terms
 ├── Confidentiality
 ├── Indemnification
 └── Termination
```

For a question about termination rights, a structure-aware approach should preserve the relevant clauses and associated definitions rather than arbitrarily splitting the termination provision halfway through.

Where clauses depend on other sections, retrieve or expand the surrounding information.

### **B. Indexing Strategies**

| Index                 | Mechanism                               | Strength                                               |
| --------------------- | --------------------------------------- | ------------------------------------------------------ |
| Vector index          | Embedding similarity                    | Semantic matching and paraphrases                      |
| BM25 / inverted index | Lexical term weighting                  | Exact terminology, rare keywords, identifiers          |
| Hybrid index          | Combines lexical and semantic retrieval | Broad enterprise search                                |
| Metadata index/filter | Filters documents by attributes         | Tenant, department, date, document type, access rights |

Store useful metadata such as:

```
{
  "document_id": "contract_1042",
  "chunk_id": "termination_03",
  "section": "Termination",
  "version": "v2",
  "tenant_id": "tenant_001",
  "effective_date": "2026-04-01",
  "access_scope": "legal_team",
  "source_reference": "section_12"
}
```

This is a representative application schema, not an Anthropic-mandated format.

### **C. Contextual Retrieval**

A particular chunk may make little sense without document context.

For example:

> The above exception shall not apply after the renewal date.

A contextualized chunk should also identify which exception, agreement, and renewal period the sentence refers to.

Anthropic's **Contextual Retrieval** approach generates brief explanatory context for chunks before indexing, improving both semantic and lexical retrieval.

In Anthropic's published experiments:

| Approach                                | Top-20 retrieval failure rate |
| --------------------------------------- | ----------------------------- |
| Baseline                                | 5.7%                          |
| Contextual embeddings + Contextual BM25 | 2.9%                          |
| Contextual retrieval + reranking        | 1.9%                          |

These correspond to approximately **49%** and **67%** relative reductions in retrieval failure. They are experimental findings, not guaranteed improvements for every RAG system.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic



### **Best Practices**

1. ⭐ Choose chunk boundaries based on document structure and question patterns; do not treat a fixed token count as universally optimal.
2. Preserve source metadata, document version, section hierarchy, and access-control information.
3. Evaluate hybrid retrieval and reranking when vector-only retrieval misses precise terms or contextual relevance.
4. Apply document permissions during retrieval and ensure updated or deleted content is reflected in the index.
5. Evaluate retrieval and generation separately so you can distinguish missing evidence from model reasoning failures.

Anthropic's contextual retrieval research supports combining meaningful chunk context, lexical and semantic indexes, and reranking, subject to latency and cost constraints.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic



**⭐ Exam takeaway:** RAG failures often originate in document parsing, chunk boundaries, metadata, or retrieval—not in the Claude model.

## **3.6 Apply retrieval strategies matched to data shape and query pattern**

### **Core Concept**

Retrieval should be selected according to the **shape of the data** and the **type of question**.

Vector search is powerful for semantic similarity, but it is not the best retrieval mechanism for every enterprise query.

### **Retrieval Decision Matrix**

| Data / query pattern                       | Preferred strategy                        | Why                                         |
| ------------------------------------------ | ----------------------------------------- | ------------------------------------------- |
| Exact policy ID or invoice number          | Keyword / exact-match / metadata          | Preserves exact identifiers                 |
| Natural-language paraphrase                | Semantic vector search                    | Matches meaning rather than literal words   |
| Legal clauses with precise terminology     | Hybrid BM25 + vector                      | Preserves both lexical and semantic signals |
| Large document with hierarchical structure | Parent-child or structure-aware retrieval | Retains surrounding context                 |
| SQL-style aggregation or joins             | Structured database query/API             | Produces exact calculations                 |
| Rapidly changing operational information   | Live API lookup                           | Avoids stale index results                  |
| Ambiguous multi-part investigation         | Query decomposition + iterative retrieval | Finds evidence for different subquestions   |
| High-precision evidence retrieval          | Candidate retrieval + reranking           | Improves result ordering                    |

### **Canonical Skeleton**

```
User Query
    |
    v
Identify Query Intent and Data Shape
    |
    +-- Exact Lookup ------> Keyword / Metadata
    |
    +-- Semantic Question -> Vector Search
    |
    +-- Mixed Terminology -> Hybrid Search
    |
    +-- Structured Data --> Database / API
    |
    +-- Complex Question -> Query Decomposition
    |
    v
Merge Relevant Results
    |
    v
Rerank / Deduplicate / Expand Context
    |
    v
Check Evidence Sufficiency
    |
    +-- Insufficient --> Refine Query / Retrieve Again
    |
    v
Generate Grounded Answer
```

### **Retrieval Trade-offs**

| Decision                      | Benefit                                         | Cost / risk                               |
| ----------------------------- | ----------------------------------------------- | ----------------------------------------- |
| Increase `top_k`              | Potentially higher recall                       | More tokens, noise, latency               |
| Use reranking                 | Better relevance ordering                       | Additional computation                    |
| Use hybrid search             | Improved coverage of exact and semantic matches | More indexing and ranking complexity      |
| Use iterative retrieval       | Better handling of ambiguous questions          | More tool calls and latency               |
| Retrieve parent sections      | Restores context around small chunks            | Larger model input                        |
| Apply strict metadata filters | Improves relevance and access isolation         | Incorrect metadata can hide valid results |

### **Evaluation Metrics**

| Metric       | What it measures                                        |
| ------------ | ------------------------------------------------------- |
| Recall\@K    | Fraction of all relevant items retrieved in the top K   |
| Precision\@K | Fraction of top K retrieved items that are relevant     |
| Hit\@K       | Whether at least one relevant item appears in the top K |
| MRR          | How highly the first relevant item is ranked            |
| nDCG\@K      | Ranking quality accounting for graded relevance         |
| Groundedness | Whether the generated answer is supported by evidence   |

**Example:** For employee policy retrieval, high Recall\@K helps ensure important provisions are not missed. For a search interface showing only three results, Precision\@3 and ranking quality may matter more.

### **Best Practices**

1. ⭐ Match the retrieval method to the query instead of forcing all enterprise data into embeddings.
2. Combine lexical and vector retrieval where complementary strengths materially improve results.
3. Tune candidate count, `top_k`, reranking, and context expansion using a labeled evaluation set.
4. Use structured queries for calculations and relational operations rather than asking Claude to infer exact totals from retrieved prose.
5. Return source identifiers and sufficient surrounding evidence so generated claims can be checked.

Anthropic's retrieval experiments show the value of combining retrieval methods, while its broader context-engineering guidance favors supplying the smallest useful set of high-signal information.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+1



**⭐ Exam takeaway:** The correct retrieval architecture depends on the data and task. More retrieved chunks do not automatically produce more accurate answers.

## **3.7 Evaluate connection protocols and select the appropriate integration mechanism (MCP, API/CLI, agent-to-agent)**

### **Core Concept**

Claude can integrate with enterprise systems through different mechanisms. The correct choice depends on reuse requirements, execution environment, control, autonomy, and operational complexity.

⭐ **MCP standardizes access to capabilities. Direct APIs execute explicit operations. Agent-to-agent integration delegates work to another agent.**

These mechanisms are complementary, not universally interchangeable.

### **Integration Decision Matrix**

| Mechanism                | What it provides                                             | Best use case                                           | Main trade-off                                                      |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------------- |
| **Direct API / SDK**     | Explicit application-to-service integration                  | A small number of stable integrations                   | Tighter coupling; custom integration code                           |
| **CLI**                  | Scriptable command execution                                 | Developer tools, infrastructure automation              | Execution environment and command security                          |
| **MCP**                  | Standardized discovery and invocation of tools and resources | Multiple agents/clients sharing enterprise capabilities | Additional protocol and server-management complexity                |
| **Agent-to-agent (A2A)** | Delegation between autonomous or specialized agents          | Distributed, multi-agent workflows                      | Coordination, identity propagation, and failure-handling complexity |

### **A. Direct API Integration**

Choose direct API integration when:

- The application has a limited number of well-defined operations.
- You control the complete integration.
- Deterministic orchestration is preferable to dynamic discovery.
- You do not need to expose the same capabilities to many different agent clients.

**Example:**

```
User
  |
  v
FastAPI Backend
  |
  v
Claude Messages API
  |
  v
Tool Use Request
  |
  v
Backend Executes CRM API
  |
  v
Tool Result Returned to Claude
  |
  v
Final Answer
```

For client-defined tools, Claude produces a `tool_use` request. The application executes the actual operation and supplies a corresponding `tool_result`.

**Remember:** Defining a tool in a Claude API request does not mean Anthropic automatically executes your custom application code.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs



### **B. Model Context Protocol (MCP)**

MCP provides a standardized interface between AI applications and external capabilities.

Its principal primitives are:

| Primitive | Purpose                           | Example                               |
| --------- | --------------------------------- | ------------------------------------- |
| Tools     | Execute operations                | `search_tickets`, `create_issue`      |
| Resources | Expose data/context               | Document contents, schema information |
| Prompts   | Provide reusable prompt templates | Incident investigation template       |

**Typical architecture:**

```
             Claude Application
                    |
                 MCP Client
                    |
             MCP Protocol
                    |
          +---------+---------+
          |                   |
          v                   v
     CRM MCP Server      HR MCP Server
          |                   |
          v                   v
       CRM API             HRMS API
```

MCP can use local `stdio` transport or network-based transport such as Streamable HTTP. Transport choice depends on deployment and security requirements.

**Important distinction:** The MCP protocol supports tools, resources, and prompts, but individual Claude integrations may support only a subset. For example, the Messages API MCP connector has supported tool invocation; it should not be assumed to expose every MCP primitive through the same interface.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **C. CLI Integration**

CLI-based integrations are appropriate when a supported command-line tool already performs the necessary operation.

Example:

```
Claude Agent
     |
     v
Validated Tool Invocation
     |
     v
Restricted CLI Runner
     |
     v
Infrastructure / Repository / Build Tool
```

Use CLI execution for controlled development and operational workflows, not as a justification for unrestricted shell access.

The execution environment must enforce the permitted commands, working directories, credentials, network access, and resource limits.

### **D. Agent-to-Agent Integration**

Agent-to-agent integration is useful when the receiving component is itself an agent capable of independently planning and executing a delegated task.

```
                  Orchestrator Agent
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Research Agent          Compliance Agent
             |                       |
             v                       v
       Search / RAG             Policy / Rules
             |                       |
             +-----------+-----------+
                         |
                         v
                  Synthesized Result
```

A delegated task should have an explicit input contract, expected output, timeout, permission scope, and failure-handling policy.

**Do not confuse an architectural delegation pattern with guaranteed support for a specific A2A protocol.** Exact interoperability and protocol-version support are implementation-dependent.

### **Canonical Skeleton**

```
Identify Integration Requirement
                 |
                 v
Simple Deterministic Operation?
                 |
        +--------+--------+
        | Yes             | No
        v                 v
   Direct API/SDK    Shared Reusable Tools?
                           |
                    +------+------+
                    | Yes         | No
                    v             v
                   MCP       Autonomous Delegation?
                                    |
                              +-----+-----+
                              | Yes       | No
                              v           v
                          Agent-to-Agent  Direct API
```

CLI may replace a direct service integration when the operation is fundamentally command-line-driven.

### **Best Practices**

1. ⭐ Prefer direct integration for a small, stable set of operations; introduce MCP when standardization and reuse justify it.
2. Provide narrow, clearly described tools instead of exposing unrestricted generic operations.
3. Enforce authentication, authorization, input validation, and audit controls at each integration boundary.
4. Use agent delegation only when the task benefits from independent planning or specialized expertise.
5. Define timeouts, retries, idempotency, and failure contracts for every external integration.

Anthropic's tool-use guidance emphasizes intentionally designed tools, clear schemas, and minimizing unnecessary complexity.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+1



**⭐ Exam takeaway:** A reusable enterprise tool interface suggests MCP. A simple deterministic operation usually does not need MCP or a second agent.

## **3.8 Evaluate progressive discovery vs. monolithic context strategy**

### **Core Concept**

**Monolithic context** loads a large amount of tool or reference information upfront.

**Progressive discovery** starts with a smaller context and loads additional capabilities or information when needed.

This is especially important when an enterprise agent connects to dozens of MCP servers or hundreds of tools.

### **Architecture Comparison**

**Monolithic Context**

Claude

All tool definitions loaded upfront

CRM · HR · GitHub · Jira · Finance · Search · More

No discovery step, but high baseline context consumption.

**Progressive Discovery**

Claude

Core Tools + Tool Search

Load relevant tool definitions on demand

Smaller initial context, with discovery overhead.

### **Decision Criteria**

| Consideration                  | Monolithic                    | Progressive                       |
| ------------------------------ | ----------------------------- | --------------------------------- |
| Number of available tools      | Small                         | Large                             |
| Initial tool-selection latency | Usually lower                 | Additional discovery step         |
| Tool-definition token cost     | Higher as catalog grows       | Lower baseline                    |
| Relevance of loaded tools      | All included                  | Selected on demand                |
| Implementation complexity      | Simpler                       | Requires discovery configuration  |
| Best use                       | Small, stable tool collection | Large, varied enterprise catalogs |

### **Tool Search Tool**

Anthropic introduced the Tool Search Tool in November 2025, enabling deferred tool definitions and on-demand discovery.

A representative configuration:

```
{
  "tools": [
    {
      "type": "tool_search_tool_regex_20251119",
      "name": "tool_search_tool_regex"
    },
    {
      "name": "search_employee_policy",
      "description": "Search approved HR policies",
      "input_schema": {
        "type": "object",
        "properties": {
          "query": {
            "type": "string"
          }
        },
        "required": ["query"]
      },
      "defer_loading": true
    }
  ]
}
```

This illustrates the historical Tool Search Tool schema. The example is not a complete API request; feature enablement and model support were version-dependent.

Anthropic's November 2025 guidance identified **10+ tools** or **more than approximately 10,000 tokens of tool definitions** as situations where tool search could be useful.

These are optimization heuristics, not mandatory API thresholds.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic



### **Canonical Skeleton**

```
Approved Tool Registry
         |
         v
Classify Tools by Frequency
         |
         +---- Frequently Used
         |           |
         |           v
         |      Load Initially
         |
         +---- Infrequently Used
                     |
                     v
                Defer Loading
                     |
                     v
              Tool Search Index
                     |
                     v
User Request -> Claude -> Search Tools
                     |
                     v
              Load Matching Tools
                     |
                     v
              Execute Authorized Tool
                     |
                     v
                  Response
```

### **Best Practices**

1. ⭐ Keep a small set of frequently used tools available and defer less common capabilities.
2. Use meaningful tool names and descriptions to improve discovery accuracy.
3. Separate capability discovery from authorization; a discoverable tool must still pass permission checks.
4. Evaluate the context savings against additional search latency and potential discovery failures.
5. Prefer just-in-time retrieval of reference material rather than repeatedly loading entire documents or tool catalogs.

Anthropic recommends targeting the actual source of inefficiency: tool search for definition bloat, programmatic tool calling for excessive intermediate results, and tool-use examples for ambiguous parameters.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic



**⭐ Exam takeaway:** Progressive discovery reduces initial context consumption; it does not eliminate authorization requirements or guarantee lower end-to-end latency.

# **Part 2: Consolidated Exam Preparation**

## **4. Exam Traps**

| #  | Trap                                                 | Why it's tempting                              | Why it's wrong                                                            |
| -- | ---------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------- |
| 1  | ⭐ Keep all privileged tools but add approval prompts | Appears to improve safety                      | Unnecessary capabilities still increase exposure; remove them             |
| 2  | Authenticate the user, then trust every tool request | User identity has been established             | Authorization must be checked for each protected operation                |
| 3  | Treat an MCP connection as proof of secure access    | MCP provides standardized connectivity         | Protocol interoperability does not guarantee least privilege              |
| 4  | Use MCP for every integration                        | MCP seems more extensible                      | Simple direct integrations can be easier and more reliable                |
| 5  | Use another agent for every complex tool call        | Multi-agent architectures appear sophisticated | Delegation adds coordination cost and failure modes                       |
| 6  | Increase RAG `top_k` indefinitely                    | More evidence seems better                     | Additional irrelevant chunks can reduce answer quality                    |
| 7  | Use vector search for exact identifiers              | Embeddings are powerful                        | Lexical and structured retrieval are often more reliable                  |
| 8  | Fix every incorrect RAG answer with a larger model   | Better models may reason better                | A model cannot reliably recover evidence that retrieval failed to provide |
| 9  | ⭐ Treat successful API calls as successful tasks     | HTTP status is easy to measure                 | Technical success does not establish answer correctness                   |
| 10 | Optimize latency by immediately removing reranking   | Reranking adds processing time                 | It may be crucial to retrieval quality; identify the bottleneck first     |
| 11 | Preload all tools to avoid discovery latency         | Avoids an extra step                           | Large catalogs consume context and can worsen selection                   |
| 12 | Assume progressive discovery enforces permissions    | Only relevant tools are loaded                 | Relevance filtering and access control solve different problems           |
| 13 | Assume streaming makes model execution faster        | Users see output sooner                        | Streaming primarily changes perceived latency                             |
| 14 | Use batch processing for interactive workloads       | Batch pricing is attractive                    | Asynchronous completion may violate response-time requirements            |
| 15 | Log complete prompts and responses by default        | Rich logs simplify debugging                   | May expose personal information, credentials, and confidential data       |

### **Common Diagnostic Patterns**

| Exam scenario                                      | Most likely architectural response                               |
| -------------------------------------------------- | ---------------------------------------------------------------- |
| Wrong tool selected among many similar tools       | Improve schemas and descriptions; reduce or defer tool inventory |
| Unauthorized record returned                       | Investigate downstream resource authorization                    |
| Wrong answers after document refresh               | Investigate ingestion, versions, indexing, and retrieval         |
| Correct results but excessive latency              | Inspect per-stage traces and critical-path dependencies          |
| Large, rarely used tool catalog                    | Progressive discovery                                            |
| Multiple applications need the same business tools | Reusable MCP integration                                         |
| Exact identifier search performs poorly            | Lexical/metadata retrieval                                       |
| Agent executes repetitive unnecessary calls        | Tighten orchestration and stopping conditions                    |

## **5. Key Facts to Memorize**

### **A. Tool and Agent Integration**

| Item                      | Key fact                                                                       |
| ------------------------- | ------------------------------------------------------------------------------ |
| ⭐ Least privilege         | Expose only required tools, actions, and resource permissions                  |
| Capability bloat          | Excess tools increase security exposure, context cost, and selection ambiguity |
| Tool descriptions         | Names, descriptions, and schemas influence tool-selection accuracy             |
| `tool_use`                | Model-generated request to invoke a tool                                       |
| `tool_result`             | Tool execution result returned to Claude by the client application             |
| `tool_choice`             | Historically includes `auto`, `any`, `tool`, and `none`; model support varies  |
| Tool Search Tool          | Discovers and loads tools on demand                                            |
| `defer_loading`           | Marks tools for deferred loading when supported                                |
| Tool Use Examples         | Clarify complex or ambiguous parameter usage                                   |
| Programmatic Tool Calling | Enables code-driven orchestration and intermediate-result processing           |

### **B. Integration Protocols**

| Item                     | Key fact                                                                 |
| ------------------------ | ------------------------------------------------------------------------ |
| MCP                      | Model Context Protocol                                                   |
| MCP primitives           | Tools, resources, prompts                                                |
| MCP local transport      | Commonly `stdio`                                                         |
| MCP network transport    | Streamable HTTP                                                          |
| Direct API               | Explicit service-level integration                                       |
| CLI                      | Command-line integration requiring execution controls                    |
| A2A                      | Delegation to another autonomous agent; specific protocol support varies |
| AuthN                    | Authentication: identity verification                                    |
| AuthZ                    | Authorization: permission enforcement                                    |
| OAuth                    | Common authorization mechanism for protected remote MCP integrations     |
| ⭐ Resource authorization | Enforce outside the model, at trusted service boundaries                 |

### **C. Retrieval and RAG**

| Item                   | Key fact                                                          |
| ---------------------- | ----------------------------------------------------------------- |
| RAG                    | Retrieval-Augmented Generation                                    |
| BM25                   | Lexical relevance ranking                                         |
| Embeddings             | Vector representations used for semantic similarity               |
| Hybrid retrieval       | Combines lexical and semantic search                              |
| Reranker               | Reorders retrieved candidates by relevance                        |
| Contextual retrieval   | Adds chunk-specific explanatory context before indexing           |
| Parent-child retrieval | Retrieves focused chunks while preserving larger context          |
| Recall\@K              | Coverage of relevant items among the top K results                |
| Precision\@K           | Proportion of relevant items among the top K results              |
| MRR                    | Mean Reciprocal Rank                                              |
| nDCG                   | Normalized Discounted Cumulative Gain                             |
| Groundedness           | Degree to which generated claims are supported by evidence        |
| ACL                    | Access Control List; must be respected during protected retrieval |

### **D. Performance and Operations**

| Item           | Key fact                                                                        |
| -------------- | ------------------------------------------------------------------------------- |
| TTFT           | Time To First Token                                                             |
| p50            | Median request latency                                                          |
| p95            | Latency threshold at or below which 95% of requests complete                    |
| p99            | Latency threshold at or below which 99% complete                                |
| RPM            | Requests per minute                                                             |
| ITPM           | Input tokens per minute                                                         |
| OTPM           | Output tokens per minute                                                        |
| HTTP 429       | Rate-limiting response; retry appropriately                                     |
| HTTP 529       | API overload response                                                           |
| `request-id`   | Request correlation and troubleshooting identifier                              |
| Prompt caching | Reuses eligible stable prompt prefixes                                          |
| Batch API      | Asynchronous request processing                                                 |
| OpenTelemetry  | Distributed tracing and telemetry framework; instrumentation must be configured |

Anthropic's API documentation provides request identification, token usage, rate-limiting behavior, and error handling details.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **E. Useful Numerical Facts and Limits**

| Item                                   | July 2026 study fact                       | Important qualification                         |
| -------------------------------------- | ------------------------------------------ | ----------------------------------------------- |
| Domain 3 weight                        | **19%**                                    | Exam-domain weight                              |
| Tool search heuristic                  | Around 10+ tools or >10K definition tokens | Engineering heuristic, not a hard limit         |
| Frequently used tools                  | Keep a small core readily available        | No universal required count                     |
| Standard prompt cache TTL              | 5 minutes                                  | Cache eligibility depends on model and platform |
| Extended cache TTL                     | 1 hour option                              | Pricing/support varies                          |
| Batch API discount                     | 50% versus standard API pricing            | Applicable to supported batch processing        |
| Batch completion window                | Up to 24 hours                             | Not a real-time latency guarantee               |
| Anthropic contextual retrieval example | 49% relative retrieval-failure reduction   | Experimental result                             |
| Contextual retrieval + reranking       | 67% relative reduction                     | Experimental result, not guaranteed             |
| Universal RAG chunk size               | **None**                                   | Must be tuned for the dataset                   |
| Universal optimal `top_k`              | **None**                                   | Must be measured through retrieval evaluations  |
| Universal model latency target         | **None**                                   | Depends on application SLOs                     |

The caching and batch characteristics above were available before the requested cutoff. Model-specific token minimums, prices, rate-limit quotas, and feature flags should not be memorized as universal constants.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+3



## **6. Five Exam-Style Questions**

These are original practice questions designed to test architectural judgment, not recalled live-exam questions.

### **Question 1 — Least Privilege and Authorization**

A financial services organization deploys a Claude assistant to help accounts-payable analysts retrieve invoices and explain payment status.

The agent currently has tools to retrieve invoices, update vendor banking information, approve payments, and delete vendor records. Analysts only need invoice retrieval and payment-status explanations.

Which approach BEST addresses the integration risk?

**A.** Retain all tools, enable audit logging, and instruct Claude not to use sensitive actions.

**B.** Remove unnecessary mutation tools, use narrowly scoped read permissions, and enforce record-level authorization in backend services.

**C.** Retain all tools but require user confirmation before every mutation.

**D.** Replace the current Claude model with a more capable model to reduce accidental tool invocation.

**Answer: B**

Show explanation

### **Question 2 — RAG Design and Retrieval Quality**

A legal department uses Claude to answer questions about thousands of contracts. Users frequently search using exact clause references as well as natural-language descriptions.

After a contract repository update, the assistant starts returning older clause versions, occasionally missing termination provisions. The model configuration has not changed.

Which design change is BEST?

**A.** Increase the Claude output-token limit so it can reason more thoroughly.

**B.** Rebuild and validate affected indexes, retain document-version metadata, and use structure-aware chunking with hybrid retrieval and reranking.

**C.** Increase `top_k` to the maximum feasible value and send every retrieved chunk to Claude.

**D.** Remove semantic retrieval and use only vector similarity from a newly generated embedding model.

**Answer: B**

Show explanation

### **Question 3 — Choosing an Integration Mechanism**

An enterprise is building multiple Claude-powered assistants for sales, HR, and IT operations.

All three assistants require access to a shared set of approved internal services. The organization wants reusable tool definitions, consistent integration interfaces, and centralized authorization enforcement.

Which approach BEST fits these requirements?

**A.** Create separate, duplicated direct API integrations inside every assistant.

**B.** Build a shared MCP integration layer exposing approved, task-specific tools with enforced identity and authorization controls.

**C.** Give every assistant unrestricted CLI access to internal infrastructure.

**D.** Replace all direct service calls with independent autonomous agents, even for simple record lookups.

**Answer: B**

Show explanation

### **Question 4 — Latency Optimization and Observability**

A Claude research assistant retrieves information from three independent enterprise APIs, reranks the evidence, and generates a final answer.

The application achieves acceptable answer quality but misses its eight-second p95 latency SLO. Distributed traces show that the three API calls run sequentially and account for most of the response time.

Which approach should the architect choose FIRST?

**A.** Remove reranking to reduce computation.

**B.** Use a smaller Claude model and accept a possible accuracy reduction.

**C.** Execute the independent API calls concurrently, preserve existing quality controls, and compare latency and task success before and after the change.

**D.** Enable streaming and assume the p95 completion-time SLO will be satisfied.

**Answer: C**

Show explanation

### **Question 5 — Progressive Tool Discovery**

An enterprise agent connects to 80 approved tools spanning CRM, document management, security monitoring, and engineering systems.

Loading all tool schemas consumes a substantial portion of the context window. Tool-selection accuracy is deteriorating, but four tools account for most daily operations.

Which architecture BEST balances context efficiency and functionality?

**A.** Continue loading all tools but increase the context-window size.

**B.** Keep the four frequently used tools available, defer less common tools, and enable tool search with clear descriptions and existing permission controls.

**C.** Remove all tools except the four frequently used ones, even though some legitimate workflows require others.

**D.** Load all tool definitions into every system prompt and enable extended thinking.

**Answer: B**

Show explanation

## **7. Last-Minute Revision Checklist**

- ⭐ Remove unnecessary privileged tools rather than relying on prompts or logs.
- ⭐ Understand AuthN vs. AuthZ; enforce permissions in trusted services.
- Know when to choose MCP, direct API/CLI, or agent-to-agent delegation.
- Understand `tool_use`, `tool_result`, tool schemas, and tool-selection controls.
- Know when progressive tool discovery is better than loading all tools upfront.
- Choose RAG chunking based on document structure and retrieval requirements.
- Distinguish BM25, vector search, hybrid retrieval, and reranking.
- Diagnose RAG failures at ingestion, indexing, retrieval, and generation stages.
- Measure end-to-end p95 latency, token consumption, tool behavior, and task success.
- ⭐ Justify architecture choices using security, quality, latency, cost, and maintainability trade-offs.

**Final decision rule for Domain 3:** The best answer is generally the architecture that preserves required capabilities and answer quality while minimizing unnecessary privilege, context, latency, and operational complexity—not the answer that adds the most tools, agents, or technology layers.
