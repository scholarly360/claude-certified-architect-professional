# **Claude Certified Architect – Professional (CCA-P)**

## **Domain 5: Governance, Safety, and Risk Management (16%)**

**Exam scope:** December 2026 blueprint, effective December 7, 2026.

**Technical scope:** Product capabilities verified through November 28, 2026. Changes introduced during December cannot yet be verified.

**Expected exam coverage:** Approximately 9–10 of the 60 scored questions.

This domain tests your ability to identify architectural vulnerabilities, select appropriate security controls, prioritize risks, satisfy compliance obligations, and preserve human accountability without unnecessarily compromising system performance.

The central principle of this domain is:

> ⭐ Use Claude for reasoning and decision support, but enforce critical security, authorization, and compliance requirements outside the model through deterministic architectural controls.

### **Sub-domain overview**

| Sub-domain                                            | Primary competency                                            | Study priority |
| ----------------------------------------------------- | ------------------------------------------------------------- | -------------- |
| 5.1 Guardrail Placement                               | Selecting the right control at the right architectural layer  | ⭐ Very high    |
| 5.2 Prompt Injection Surface and Controls             | Protecting trust boundaries                                   | ⭐ Very high    |
| 5.3 Risk Assessment and Ranking                       | Prioritizing risks by exposure and impact                     | High           |
| 5.4 Human-in-the-Loop Checkpoints                     | Risk-based approval mechanisms                                | ⭐ Very high    |
| 5.5 Architectural Controls for Compliance Obligations | Translating compliance requirements into enforceable controls | ⭐ Very high    |
| 5.6 Bias Source Identification and Mitigation         | Diagnosing and mitigating sources of bias                     | High           |

*The six task-statement titles follow the December blueprint. Numbering and study priorities are added for revision purposes; individual sub-domain weights are not specified.*

# **Part 1: Per Sub-Domain Breakdown**

## **5.1 Guardrail Placement ⭐**

### **Core Concept**

Guardrails are technical and procedural controls that constrain an AI system's behavior within acceptable operational, security, and ethical boundaries.

The exam focuses less on defining guardrails and more on identifying **where a guardrail must be placed to interrupt a specific misuse path**.

A production Claude application should employ **defense in depth**, rather than relying exclusively on system prompts or the model's built-in safety mechanisms.

| Guardrail layer | Purpose                                              | Example                            |
| --------------- | ---------------------------------------------------- | ---------------------------------- |
| Input           | Detect prohibited or malicious requests              | Jailbreak detection, PII redaction |
| Context         | Prevent untrusted data from influencing instructions | Screening retrieved documents      |
| Model           | Guide acceptable behavior                            | System prompt restrictions         |
| Tool            | Prevent unauthorized operations                      | Parameter validation, RBAC         |
| Execution       | Contain the effects of compromised behavior          | Sandboxing, network restrictions   |
| Output          | Prevent unsafe or confidential responses             | DLP, schema validation             |
| Monitoring      | Detect violations and emerging threats               | Security alerts, audit trails      |

### **Canonical Skeleton**

```
User Request
     |
     v
Authentication and Authorization
     |
     v
Input Validation / Safety Screening
     |
     v
Context Retrieval and Screening
     |
     v
Claude + System Prompt
     |
     +----> Proposed Tool Call
     |           |
     |      Policy Enforcement
     |           |
     |      Approval (if required)
     |           |
     |      Tool Execution
     |           |
     |      Tool Result Screening
     |           |
     +-----------+
     |
     v
Output Validation / DLP
     |
     v
User Response

Audit and monitoring across all layers
```

### **Guardrail Selection Criteria**

| Observed problem                                   | Best control placement                        |
| -------------------------------------------------- | --------------------------------------------- |
| Users submit prohibited requests                   | Input screening                               |
| Retrieved documents contain malicious instructions | Context ingestion and tool-result boundary    |
| Agent attempts unauthorized database access        | Retrieval service and database authorization  |
| Agent attempts unauthorized financial transactions | Pre-execution tool authorization              |
| Responses contain confidential information         | Data-access controls and output filtering     |
| Agent executes dangerous shell commands            | Tool permissions and sandbox                  |
| Users repeatedly attempt jailbreaks                | Rate limiting and abuse detection             |
| Response contains invalid JSON                     | Structured outputs and application validation |

### **Cost, Quality and Latency Trade-offs**

| Approach                      | Benefit                           | Limitation                                |
| ----------------------------- | --------------------------------- | ----------------------------------------- |
| Rule-based validation         | Fast, inexpensive, deterministic  | Limited semantic understanding            |
| Lightweight Claude classifier | Detects nuanced misuse            | Adds inference cost and latency           |
| Secondary LLM reviewer        | Handles complex policy violations | Higher cost; can also make mistakes       |
| Human review                  | Strong contextual judgment        | Expensive and slow                        |
| Sandbox                       | Limits execution impact           | Does not establish business authorization |

⭐ **Exam decision rule:** Use deterministic controls for rules that can be expressed programmatically. Use model-based classification when semantic interpretation is required.

### **Best Practices**

- Use a lightweight model to classify suspicious inputs before invoking the main model.
- Define explicit behavioral boundaries and refusal conditions in system prompts.
- Enforce authorization in application code rather than asking Claude to decide whether a user is authorized.
- Measure both unsafe acceptance and legitimate-request rejection when tuning guardrails.
- Monitor recurring violations and adjust throttling, access restrictions, and defensive controls.

Anthropic recommends combining input screening, prompt hardening, abuse monitoring, and protection against malicious third-party content.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+2



## **5.2 Prompt Injection Surface and Controls ⭐**

### **Core Concept**

Prompt injection occurs when adversarial content attempts to manipulate Claude into disregarding authorized instructions, disclosing protected information, or performing unintended actions.

The key architectural issue is the **trust boundary between instructions and data**.

| Attack type               | Source                           | Example                                                |
| ------------------------- | -------------------------------- | ------------------------------------------------------ |
| Direct prompt injection   | User input                       | Attempting to override system instructions             |
| Indirect prompt injection | External content                 | Malicious instructions inside a retrieved document     |
| Tool-result injection     | API or tool response             | An external response attempting to redirect the agent  |
| MCP-based injection       | MCP tool descriptions or results | Untrusted content requesting an unauthorized operation |
| Skill-based injection     | Third-party Agent Skills         | A compromised Skill containing unauthorized behavior   |

A trusted user does not imply trusted content. An authenticated employee can unknowingly upload a malicious document.

### **Canonical Skeleton**

```
Trusted Instructions
  - Application system prompt
  - Authorized user objective
  - Approved enterprise policies
                 |
                 v
               Claude
                 ^
                 |
       Untrusted Data Boundary
                 |
     +-----------+------------+
     |           |            |
 Documents   MCP Results   Web Content
     |           |            |
     +-----------+------------+
                 |
      Screening and Validation
                 |
       Provenance Information
                 |
       Structured Tool Result

Claude proposes an action
                 |
                 v
 Independent Authorization Check
                 |
         Allow / Deny / Escalate
```

### **Defensive Control Selection**

| Control                      | Purpose                                             | Limitation                               |
| ---------------------------- | --------------------------------------------------- | ---------------------------------------- |
| `tool_result` content blocks | Distinguish external data from trusted instructions | Not an absolute security boundary        |
| JSON encoding                | Preserve structural boundaries                      | Does not eliminate semantic injection    |
| XML delimiters               | Separate instructions from data                     | Not an authorization mechanism           |
| Input classifiers            | Detect direct injection                             | Can produce false positives or negatives |
| Tool-result classifiers      | Detect indirect injection                           | Additional latency and cost              |
| Least-privilege tools        | Reduce possible damage                              | Does not prevent every manipulation      |
| Egress restrictions          | Restrict unauthorized transmission                  | Requires infrastructure configuration    |
| Human approval               | Prevent selected dangerous actions                  | Introduces operational friction          |
| Sandboxing                   | Restrict execution environment                      | Does not enforce every business rule     |

Anthropic specifically recommends delivering external content through `tool_result` blocks, preserving provenance, encoding untrusted strings appropriately, and screening suspicious results.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Example: Secure Document Intelligence Agent**

An enterprise contract-analysis agent processes uploaded contracts, searches SharePoint, and generates legal summaries.

A contract contains text attempting to redirect the agent toward an unrelated external action.

The appropriate architecture is:

```
Contract ingestion
    -> Document content screening
    -> Read-only retrieval
    -> Claude extracts contractual provisions
    -> Application validates extracted data
    -> Authorized result returned
```

The agent should have no unrestricted email, file-sharing, or external upload capability unless the business workflow explicitly requires it.

### **MCP and Agent Skills Security**

| Component          | Recommended controls                                          |
| ------------------ | ------------------------------------------------------------- |
| MCP servers        | Approved server registry, scoped credentials, tool allowlists |
| MCP tools          | Validate names, schemas, permissions and arguments            |
| MCP responses      | Treat external content as untrusted                           |
| Agent Skills       | Review instructions, scripts and dependencies                 |
| Skill distribution | Use approved repositories and controlled versions             |
| Subagents          | Restrict tools and context to their assigned tasks            |
| Secrets            | Store outside model context in a secret manager               |

### **Best Practices**

- Keep external documents, emails, search results, and tool responses separate from trusted instructions.
- Explicitly instruct Claude not to treat external content as new operating instructions.
- Prefer narrowly scoped tools over unrestricted shell or generic HTTP capabilities.
- Deploy Skills from trusted sources and audit third-party Skills before approval.
- Test direct and indirect injection paths, including attempts involving tool chains and external destinations.

Anthropic's Agent Skills security guidance emphasizes trusted distribution, dependency inspection, and reviewing what a Skill can execute or access.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



⭐ **Exam decision rule:** Do not assume that prompt injection can be completely eliminated. Design the system so that successful manipulation cannot produce an unauthorized effect.

## **5.3 Risk Assessment and Ranking**

### **Core Concept**

Risk assessment identifies potential failures, estimates their consequences and likelihood, and determines which controls deserve priority.

For Claude-based applications, the exam emphasizes four broad risk categories:

| Risk category                    | Examples                                                         |
| -------------------------------- | ---------------------------------------------------------------- |
| Attack surface                   | Prompt injection, excessive permissions, insecure integrations   |
| Failure modes                    | Hallucination, incorrect tool calls, cascading agent failures    |
| Token and quota exhaustion       | Excessive reasoning, runaway loops, API quota depletion          |
| Dependency and supply-chain risk | Compromised Skills, vulnerable packages, unavailable MCP servers |

Risks should be ranked by actual exposure rather than treating all theoretical vulnerabilities as equally important.

### **Risk Evaluation Factors**

| Factor            | Question to ask                                                      |
| ----------------- | -------------------------------------------------------------------- |
| Likelihood        | How probable is the failure or attack?                               |
| Impact            | What business, financial, security or regulatory damage could occur? |
| Exposure          | How accessible is the vulnerable component?                          |
| Blast radius      | How many users, tenants or systems could be affected?                |
| Reversibility     | Can the damage be undone?                                            |
| Detectability     | How quickly would the organization identify the failure?             |
| Existing controls | What safeguards already reduce the risk?                             |

### **Canonical Skeleton**

```
Identify Assets and Business Objectives
                  |
                  v
Map Data Flows and Trust Boundaries
                  |
                  v
Identify Threats and Failure Modes
                  |
                  v
Assess Inherent Risk
                  |
                  v
Evaluate Existing Controls
                  |
                  v
Rank Residual Risk
                  |
                  v
Mitigate / Avoid / Transfer / Accept
                  |
                  v
Assign Risk Owner and Monitor
```

A common risk-scoring method is:

**Risk score = Likelihood × Impact**

This is a useful enterprise risk-management convention, not a mandatory Anthropic scoring formula.

### **Example Risk Register**

The following scores are illustrative, using a 1–5 scale.

| Risk                        | Likelihood | Impact | Inherent score | Priority |
| --------------------------- | ---------- | ------ | -------------- | -------- |
| Cross-tenant data leakage   | 4          | 5      | 20             | Critical |
| Unauthorized tool execution | 4          | 5      | 20             | Critical |
| Prompt injection            | 4          | 4      | 16             | High     |
| Runaway agent execution     | 4          | 3      | 12             | High     |
| Third-party MCP outage      | 3          | 3      | 9              | Medium   |
| Formatting errors           | 4          | 1      | 4              | Low      |

Mandatory compliance requirements and catastrophic outcomes may override ordinary numerical ranking.

### **Token and Quota Exhaustion Controls**

| Risk                                  | Appropriate mitigation              |
| ------------------------------------- | ----------------------------------- |
| Infinite agent loop                   | Maximum iteration count             |
| Excessive model calls                 | Per-task call budget                |
| Excessive token consumption           | Token and spend budgets             |
| Growing conversation context          | Context editing, compaction         |
| Excessive reasoning                   | Model-supported effort controls     |
| API rate-limit exhaustion             | Admission control, queuing, backoff |
| Expensive requests from one tenant    | Per-tenant quotas                   |
| Tool execution never completes        | Timeouts and cancellation           |
| Persistent dependency failure         | Circuit breaker                     |
| Duplicate side effects during retries | Idempotency keys                    |

⭐ A `max_tokens` setting limits output generation for an individual request. It is **not** a complete spending or execution limit for a multi-step agent.

Anthropic's Messages API rate limits are expressed through RPM, ITPM and OTPM, with limits depending on model class and account configuration.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **Best Practices**

- Map every model, tool, external dependency and data store in the architecture before evaluating risks.
- Restrict network connectivity and credentials to the minimum required for the task.
- Apply cumulative token, cost, execution-time and tool-call limits to autonomous agents.
- Audit and version external Skills, packages and integration dependencies.
- Reassess risks after model upgrades, permission changes, new integrations or production incidents.

⭐ **Exam decision rule:** Prioritize the vulnerability with the greatest realistic business exposure and blast radius, not necessarily the one that occurs most frequently.

## **5.4 Human-in-the-Loop Checkpoints ⭐**

### **Core Concept**

Human-in-the-loop (HITL) introduces human judgment and approval at selected points in an AI workflow.

The important architectural decision is determining **which actions require approval and exactly when that approval must occur**.

Requiring approval for every action reduces productivity. Removing every approval creates unacceptable exposure when actions are consequential or irreversible.

### **Human Oversight Decision Matrix**

| Action                                | Reversibility | Potential impact | Typical control                                              |
| ------------------------------------- | ------------- | ---------------- | ------------------------------------------------------------ |
| Search authorized documents           | High          | Low              | Automated                                                    |
| Generate a draft                      | High          | Low              | Automated                                                    |
| Produce an internal summary           | High          | Low              | Automated                                                    |
| Update a noncritical internal record  | High          | Low–medium       | Validation and audit                                         |
| Send sensitive external communication | Low           | High             | Pre-execution approval                                       |
| Authorize financial payment           | Low           | High             | Mandatory approval                                           |
| Delete production records             | Low           | High             | Approval and recovery controls                               |
| Provide high-stakes decision support  | Variable      | High             | Qualified human oversight and applicable policy restrictions |

### **Canonical Skeleton**

```
User Request
     |
     v
Claude Analyzes and Plans
     |
     v
Proposed Action
     |
     v
Deterministic Risk Classification
     |
     +--------------------------+
     |                          |
 Low-Risk Action          High-Risk Action
     |                          |
 Authorization Check      Persist Pending Action
     |                          |
 Execute                  Human Approval
     |                     /           \
     |                 Approved       Rejected
     |                    |              |
     |              Revalidate Action   Stop
     |                    |
     |                 Execute
     |                    |
     +--------------------+
              |
         Audit Result
```

### **Where to Place the Checkpoint**

| Checkpoint            | Appropriate use                                          |
| --------------------- | -------------------------------------------------------- |
| Before planning       | Restricted or particularly sensitive workflow initiation |
| After planning        | Review complex strategy or task scope                    |
| Before tool execution | Prevent unauthorized or irreversible effects             |
| After execution       | Verify reversible changes and detect errors              |
| Before publishing     | Review sensitive externally visible content              |

⭐ **Approval immediately before execution is generally the strongest checkpoint for high-risk side effects.**

A human reviewing the initial plan does not guarantee that subsequent tool calls will match that plan.

### **Human-in-the-Loop Design Example**

Consider an accounts-payable agent:

```
Read invoice                  -> Automated
Extract invoice fields        -> Automated
Validate purchase order        -> Automated
Check duplicate invoice       -> Automated
Prepare payment proposal      -> Automated
Approve payment               -> Human
Revalidate amount/payee       -> Deterministic
Execute payment               -> Automated
Record transaction            -> Automated
```

An approval should be bound to the exact transaction parameters, including amount, payee, currency and transaction identifier. If any material parameter changes, the previous approval becomes invalid.

### **Claude-Specific Implementation Mechanisms**

| Runtime                | Mechanism                                                             |
| ---------------------- | --------------------------------------------------------------------- |
| Messages API           | Intercept proposed client-side `tool_use` before execution            |
| Claude Agent SDK       | Permission callbacks and tool policies                                |
| Claude Agent SDK       | `PreToolUse` hooks for inspection or denial                           |
| Claude Code            | Allow, ask and deny permission rules                                  |
| Claude Managed Agents  | Supported tool permission policies and application-mediated approvals |
| Enterprise application | Persistent approval workflow outside the model                        |

These mechanisms support different runtime architectures; their APIs and availability should not be assumed to be identical.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+3



### **Best Practices**

- Place mandatory gates before actions with serious or irreversible consequences.
- Automate routine, authorized and low-risk steps to avoid approval fatigue.
- Bind approvals to immutable action details and revalidate authorization before execution.
- Define expiration, rejection, escalation and recovery behavior for pending approvals.
- Use independent policy checks instead of relying on Claude's self-reported confidence.

⭐ **Exam decision rule:** Approval requirements should follow the consequences of the proposed action, not the model's confidence in that action.

### **Anthropic's High-Risk Use Case Requirements**

For applicable high-risk use cases affecting health, legal rights, finances, employment or access to essential services, Anthropic requires qualified human oversight and disclosure of AI involvement to affected individuals. The human reviewer must have the authority to change the recommendation, rather than merely endorse it.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+1



## **5.5 Architectural Controls for Compliance Obligations ⭐**

### **Core Concept**

Compliance requirements must be translated into enforceable architectural controls supported by verifiable evidence.

The architect must distinguish between:

- Where data is processed.
- Where data is stored.
- How long data is retained.
- Whether data is used for model training.
- Who can access the data.
- Whether activities can be audited.

These are independent requirements. Satisfying one does not automatically satisfy the others.

### **Compliance Requirement-to-Control Mapping**

| Requirement          | Architectural control                            | Evidence                   |
| -------------------- | ------------------------------------------------ | -------------------------- |
| Data residency       | Region-restricted deployment and inference       | Deployment configuration   |
| Data retention       | ZDR or approved retention settings               | Contract and configuration |
| PII protection       | Detection, redaction, tokenization               | Validation results         |
| Tenant isolation     | Tenant-scoped retrieval, storage and credentials | Access-control tests       |
| Auditability         | Protected application and tool logs              | Audit records              |
| Data minimization    | Send only necessary information                  | Data-flow documentation    |
| Encryption           | Encryption in transit and at rest                | Security configuration     |
| Right to deletion    | Deletion across applicable data stores           | Deletion records           |
| Human accountability | Approval gates and reviewer identification       | Approval history           |

### **Canonical Skeleton**

```
User / Enterprise Application
              |
      Identity Verification
              |
      Tenant Authorization
              |
      Data Classification
              |
      PII Detection / Redaction
              |
      Approved Regional Endpoint
              |
            Claude
              |
      Output Validation / DLP
              |
      Authorized Destination

Cross-cutting controls:
  - Encryption
  - Retention enforcement
  - Tenant isolation
  - Audit logging
  - Secrets management
  - Compliance monitoring
```

### **A. Data Residency vs. Data Retention**

This distinction is particularly important for the exam.

| Concept                   | Meaning                                                                            |
| ------------------------- | ---------------------------------------------------------------------------------- |
| Data residency            | Geographic location of data processing and storage                                 |
| Data retention            | Duration for which data is retained                                                |
| Data sovereignty          | Jurisdictional and legal authority over data                                       |
| Zero Data Retention (ZDR) | Applicable arrangement restricting provider storage of customer inputs and outputs |
| Data minimization         | Limiting collection and processing to necessary information                        |
| No model training         | Restrictions on using customer data to train models                                |

⭐ **ZDR does not guarantee data residency.**

For example, processing a confidential document outside the approved jurisdiction and deleting it immediately may still violate a residency obligation.

### **B. Deployment Surface Selection**

| Deployment surface | Governance consideration                                                                       |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| Anthropic API      | Verify inference geography, workspace geography, contractual retention and feature eligibility |
| Amazon Bedrock     | AWS IAM, regional deployment and inference-routing configuration                               |
| Google Vertex AI   | Google Cloud IAM, regional endpoints and processing guarantees                                 |
| Microsoft Foundry  | Azure identity, deployment type and geographic processing boundaries                           |

Do not assume that using a cloud provider automatically guarantees regional processing. Global and cross-region deployment configurations may conflict with residency obligations.

Anthropic also provides geographic controls, including the `inference_geo` parameter for supported API configurations. Inference geography and workspace storage geography are distinct settings.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **C. Zero Data Retention (ZDR)**

Under Anthropic's standard commercial API policy, inputs and outputs are ordinarily deleted within 30 days, subject to exceptions involving particular services, contractual arrangements, safety enforcement and legal obligations.

ZDR requires an applicable arrangement and eligible features. It is not automatically available merely because an organization uses the commercial API.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://privacy.claude.com\&sz=32)

Anthropic Privacy Center+1



#### **Feature-level retention considerations**

| Claude feature                  | Conventional/legacy ZDR eligibility                   |
| ------------------------------- | ----------------------------------------------------- |
| Standard Messages API           | Eligible                                              |
| Token Counting API              | Eligible                                              |
| Prompt caching                  | Eligible under documented conditions                  |
| Extended thinking               | Eligible                                              |
| Structured outputs              | Eligible with qualifications                          |
| Client-side tools               | Generally eligible; external storage must be assessed |
| Files API                       | Not eligible                                          |
| Message Batches API             | Not eligible                                          |
| API Agent Skills                | Not eligible                                          |
| Server-side code execution      | Not eligible                                          |
| API MCP connector               | Not eligible                                          |
| Anthropic-hosted Managed Agents | Not eligible under conventional arrangements          |

**Version-dependent:** Retention eligibility can differ by model, contract, feature, deployment surface and runtime configuration. Some Covered Models have additional retention obligations. Self-hosted runtime options can also change the applicable data-handling boundary.

The feature-level retention matrix must therefore be verified before deployment.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



⭐ **Exam trap:** An organization cannot automatically combine strict ZDR requirements with every stateful Claude API feature.

### **D. PII Detection and Redaction**

For sensitive documents, the architecture should remove or tokenize unnecessary personal information before sending content to Claude.

```
Original document
       |
PII detection
       |
Redaction / Tokenization
       |
Sanitized document
       |
Claude processing
       |
Output validation
       |
Authorized re-identification, if required
```

| Technique        | Characteristics                                                |
| ---------------- | -------------------------------------------------------------- |
| Redaction        | Removes or masks sensitive information                         |
| Tokenization     | Replaces sensitive values with controlled identifiers          |
| Pseudonymization | Reduces direct identification but may remain reversible        |
| Anonymization    | Seeks to prevent reasonable re-identification                  |
| Encryption       | Protects data from unauthorized access but does not remove PII |

Masking names alone may be insufficient because combinations of attributes can enable re-identification.

### **E. Multi-Tenant Isolation**

A multi-tenant Claude-powered SaaS application must enforce isolation independently of the model.

| Component           | Required isolation                             |
| ------------------- | ---------------------------------------------- |
| Vector database     | Tenant filters or separate tenant indexes      |
| Relational database | Row-level security or separate databases       |
| Object storage      | Tenant-scoped access policies                  |
| Agent memory        | Tenant-specific namespaces                     |
| Cache               | Appropriate tenant-aware access and cache keys |
| API credentials     | Scoped service identities                      |
| Audit logs          | Restricted tenant and administrator access     |

⭐ Apply authorization before retrieving documents into Claude's context.

An instruction such as *only retrieve documents belonging to the current customer* is insufficient unless the retrieval service enforces it.

### **F. Audit Logging**

A production system should record enough information to reconstruct consequential actions without unnecessarily retaining sensitive content.

| Audit field                  | Purpose                   |
| ---------------------------- | ------------------------- |
| Request or correlation ID    | End-to-end traceability   |
| User and tenant ID           | Accountability            |
| Model identifier             | Reproducibility           |
| Prompt/configuration version | Change tracking           |
| Tool invocation              | Operational traceability  |
| Policy decision              | Guardrail evidence        |
| Approval identifier          | Human accountability      |
| Execution outcome            | Incident investigation    |
| Timestamp                    | Event reconstruction      |
| Token consumption            | Cost and abuse monitoring |

Avoid indiscriminately logging raw prompts, access tokens, confidential documents and personal information.

### **Best Practices**

- Select the deployment surface according to binding security and compliance obligations before optimizing convenience, cost or model availability.
- Verify retention eligibility at the individual feature and model level.
- Minimize sensitive information sent to Claude and external tools.
- Enforce tenant authorization at retrieval, storage and execution boundaries.
- Maintain privacy-preserving audit evidence demonstrating that controls operate effectively.

⭐ **Exam decision rule:** A compliance control must satisfy the exact obligation. Encryption cannot substitute for residency, and ZDR cannot substitute for access control.

## **5.6 Bias Source Identification and Mitigation**

### **Core Concept**

Bias occurs when an AI system produces systematically different or inappropriate outcomes due to characteristics of its data, prompts, model behavior or evaluation methodology.

The exam focuses on **identifying the source of bias from the available evidence and selecting the corresponding mitigation**.

Changing the model or modifying the prompt without diagnosing the underlying cause may leave the actual problem unresolved.

### **Sources of Bias**

| Bias source            | Description                                   | Mitigation                                     |
| ---------------------- | --------------------------------------------- | ---------------------------------------------- |
| Data bias              | Unrepresentative or historically biased data  | Improve dataset quality and coverage           |
| Sampling bias          | Certain populations are underrepresented      | Stratified sampling                            |
| Historical bias        | Existing discrimination reflected in records  | Review labels and historical decision criteria |
| Prompt-framing bias    | Leading language influences responses         | Neutralize prompts                             |
| Few-shot example bias  | Examples favor particular groups or outcomes  | Diversify examples                             |
| Model bias             | Learned associations produce unequal behavior | Evaluate alternative configurations and models |
| Retrieval bias         | Unequal retrieval quality across populations  | Improve indexing and retrieval coverage        |
| Evaluation-design bias | Grading methodology favors certain responses  | Recalibrate evaluators and rubrics             |

### **Canonical Skeleton**

```
Define Fairness Objectives
            |
            v
Build Representative Test Dataset
            |
            v
Generate Matched / Counterfactual Cases
            |
            v
Run Claude Evaluations
            |
            v
Measure Outcomes Across Groups
            |
            v
Identify Source of Disparity
            |
            v
Apply Targeted Mitigation
            |
            v
Re-evaluate and Monitor
```

### **A. Counterfactual Testing ⭐**

Counterfactual testing changes a sensitive characteristic while keeping other task-relevant information constant.

For example, when evaluating the fairness of a professional-development recommendation:

| Attribute      | Test case A | Test case B |
| -------------- | ----------- | ----------- |
| Experience     | 10 years    | 10 years    |
| Qualification  | Identical   | Identical   |
| Performance    | Identical   | Identical   |
| Gender         | Male        | Female      |
| Recommendation | Compare     | Compare     |

Systematically different recommendations across many matched examples may indicate sensitivity to a demographic characteristic.

Anthropic has published research using systematic demographic variation to evaluate discriminatory behavior and test prompt-based mitigations.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+1



### **B. Fairness Metrics**

| Metric                     | What it measures                                           |
| -------------------------- | ---------------------------------------------------------- |
| Demographic parity         | Differences in positive outcome rates across groups        |
| Equal opportunity          | Differences in true positive rates across groups           |
| Equalized odds             | Differences in both true positive and false positive rates |
| Disparate impact ratio     | Ratio of positive outcome rates between comparison groups  |
| Counterfactual consistency | Whether matched inputs receive consistent outcomes         |
| Subgroup accuracy          | Accuracy measured separately for each relevant group       |
| False refusal rate         | Frequency of legitimate requests being incorrectly refused |

These are general fairness evaluation concepts. Anthropic does not prescribe one universal fairness metric for every Claude application.

### **C. Diagnose Before Mitigating**

| Observation                                                   | Most likely area to investigate |
| ------------------------------------------------------------- | ------------------------------- |
| Dataset contains few examples from one population             | Data representation             |
| Prompt explicitly favors particular characteristics           | Prompt framing                  |
| Retrieved evidence differs significantly between populations  | Retrieval bias                  |
| Human reviewers see equal quality but LLM judges do not       | Evaluation-design bias          |
| Changing only a demographic attribute changes recommendations | Model or prompt sensitivity     |
| Aggregate accuracy is high but one subgroup performs poorly   | Subgroup performance disparity  |

### **D. Evaluation Trade-offs**

| Evaluation method            | Strength                                    | Limitation                              |
| ---------------------------- | ------------------------------------------- | --------------------------------------- |
| Deterministic grading        | Cheap and reproducible                      | Suitable mainly for objective criteria  |
| Statistical subgroup testing | Identifies systematic disparities           | Requires sufficient representative data |
| LLM-as-judge                 | Scalable semantic assessment                | Can introduce its own bias              |
| Human evaluation             | Better contextual judgment                  | Expensive and potentially inconsistent  |
| Counterfactual testing       | Isolates sensitivity to selected attributes | Does not capture every structural bias  |

### **Best Practices**

- Evaluate on representative datasets rather than relying exclusively on aggregate performance.
- Create matched examples that vary demographic attributes while preserving relevant qualifications.
- Use neutral instructions, balanced examples and task-relevant criteria.
- Validate automated graders against qualified human reviewers.
- Repeat fairness evaluations after changes to models, prompts, retrieval systems or datasets.

Anthropic's research and evaluation guidance support multidimensional testing, explicit success criteria and careful assessment of discriminatory behavior.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+1



⭐ **Exam decision rule:** Match the mitigation to the diagnosed source. Data imbalance requires data intervention; evaluator bias requires evaluator correction.

# **Part 2: Consolidated Exam Preparation**

## **7. Exam Traps**

These are high-probability distractor patterns derived from the architectural competencies in this domain.

| Trap                                                   | Why it is tempting                                 | Why it is wrong                                                         |
| ------------------------------------------------------ | -------------------------------------------------- | ----------------------------------------------------------------------- |
| ⭐ A strong system prompt is sufficient security        | Simple to implement                                | Model instructions are not enforceable access controls                  |
| ⭐ Output filtering prevents unauthorized tool actions  | Centralized and easy to maintain                   | The action may already have executed                                    |
| XML tags prevent prompt injection                      | Clearly separate content                           | Delimiters do not guarantee instruction isolation                       |
| Trusted MCP servers return trusted instructions        | The integration has been approved                  | The server may return adversarial third-party content                   |
| Structured outputs guarantee safe behavior             | JSON conforms to a schema                          | Valid JSON can still contain unauthorized parameters                    |
| A larger Claude model eliminates injection risks       | Better reasoning and resilience                    | Stronger models remain vulnerable to manipulation                       |
| Sandboxing eliminates all security risks               | Execution is isolated                              | Permitted network access and credentials may still be abused            |
| ⭐ Approve every tool call                              | Appears maximally secure                           | Creates unnecessary latency and approval fatigue                        |
| Approve only the initial agent plan                    | One approval is efficient                          | Actual execution can diverge from the approved plan                     |
| Use model confidence for mandatory approvals           | Enables dynamic automation                         | Confidence is not reliable authorization evidence                       |
| Review irreversible actions afterward                  | Audit logs provide visibility                      | Detection does not prevent the initial damage                           |
| ⭐ ZDR guarantees data residency                        | Both concern data privacy                          | They address different obligations                                      |
| Commercial API means ZDR is enabled                    | Commercial data has stronger privacy protections   | ZDR requires applicable arrangements and eligibility                    |
| All cloud-hosted Claude models satisfy residency       | Enterprise cloud providers offer regional services | Actual endpoint and routing configuration determine processing location |
| Tenant restrictions in system prompts ensure isolation | Easy to implement                                  | Isolation must be enforced by application and storage layers            |
| Log every prompt for compliance                        | Maximizes visibility                               | Can violate privacy and minimization obligations                        |
| `max_tokens` prevents runaway agents                   | Limits generation                                  | Does not limit cumulative calls or agent iterations                     |
| Retry every failed tool call                           | Improves reliability                               | May duplicate side effects without idempotency                          |
| ⭐ High overall accuracy demonstrates fairness          | Aggregate metric looks strong                      | Subgroup disparities may remain hidden                                  |
| Remove demographic attributes to eliminate bias        | Removes direct sensitive information               | Proxy attributes and historical bias may persist                        |
| Use another model whenever bias is detected            | Appears to solve model limitations                 | The actual bias may originate from data, prompts or evaluation          |
| LLM-as-judge provides unbiased grading                 | Automated and consistent                           | Judges can exhibit systematic preferences                               |
| Human review makes any AI use case permissible         | Provides accountability                            | Usage restrictions and compliance obligations still apply               |

## **8. Key Facts to Memorize**

### **8.1 Security and Prompt Injection**

| Term               | Key fact                                                          |
| ------------------ | ----------------------------------------------------------------- |
| Defense in depth   | Multiple independent defensive controls                           |
| Least privilege    | Minimum permissions necessary for the task                        |
| Direct injection   | Adversarial instructions originating from user input              |
| Indirect injection | Adversarial instructions originating from external content        |
| Trust boundary     | Separation between components or data with different trust levels |
| `tool_result`      | Recommended content boundary for external tool data               |
| `tool_use`         | Model-generated request to invoke a tool                          |
| Sandboxing         | Restriction of filesystem, execution and network capabilities     |
| Egress filtering   | Restriction of outbound network destinations                      |
| DLP                | Data Loss Prevention                                              |
| RBAC               | Role-Based Access Control                                         |
| ABAC               | Attribute-Based Access Control                                    |
| Red teaming        | Adversarial testing to identify vulnerabilities                   |
| Fail closed        | Deny or stop execution when critical validation fails             |

### **8.2 Claude Tools and Governance Features**

| Feature              | Exam relevance                                             |
| -------------------- | ---------------------------------------------------------- |
| System prompt        | Behavioral guidance, not deterministic security            |
| Claude Haiku         | Lightweight safety screening and classification            |
| Claude Sonnet        | Balanced capability, cost and latency                      |
| Claude Opus          | Complex analysis and reasoning                             |
| Structured outputs   | Schema-constrained model responses                         |
| Strict tool use      | Schema-constrained tool arguments                          |
| MCP                  | Standardized integration with external tools and resources |
| Agent Skills         | Reusable capability packaging requiring security review    |
| `PreToolUse`         | Hook for inspecting proposed tool execution                |
| `PostToolUse`        | Hook for post-execution processing                         |
| Permission callbacks | Application-controlled authorization decisions             |
| Managed settings     | Centralized Claude Code configuration controls             |
| Prompt caching       | Cost and latency optimization, not access control          |
| Extended thinking    | Reasoning capability, not safety assurance                 |

### **8.3 Claude Code Permission Modes**

| Permission mode     | General behavior                                                                          |
| ------------------- | ----------------------------------------------------------------------------------------- |
| `default`           | Standard permission checks and approval prompts                                           |
| `acceptEdits`       | Automatically accepts qualifying file edits                                               |
| `plan`              | Planning and read-oriented exploration with restricted execution                          |
| `dontAsk`           | Denies actions requiring approval rather than prompting                                   |
| `auto`              | Uses automated safety decisions for supported operations                                  |
| `bypassPermissions` | Bypasses ordinary permission prompts; unsuitable for unrestricted production environments |

⭐ Neither `auto` nor `bypassPermissions` should replace mandatory application-level authorization for high-risk business operations.

Permission modes and their precise behavior can vary by Claude Code or Agent SDK version.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.anthropic.com\&sz=32)

Anthropic+1



### **8.4 Risk, Reliability and Quota Management**

| Term                   | Key fact                                             |
| ---------------------- | ---------------------------------------------------- |
| Inherent risk          | Risk before mitigation                               |
| Residual risk          | Risk remaining after controls                        |
| Blast radius           | Maximum affected systems, users or tenants           |
| RPM                    | Requests per minute                                  |
| ITPM                   | Input tokens per minute                              |
| OTPM                   | Output tokens per minute                             |
| HTTP 429               | Rate-limit or applicable usage-limit error           |
| `retry-after`          | Retry timing guidance when supplied                  |
| HTTP 529               | Anthropic API overload error                         |
| Exponential backoff    | Increasing retry delays                              |
| Idempotency            | Prevents repeated execution from duplicating effects |
| Circuit breaker        | Stops repeated calls to a failing dependency         |
| `max_tokens`           | Per-request output generation limit                  |
| Agent execution budget | Cumulative calls, tokens, cost, iterations and time  |

There are no universal RPM, ITPM or OTPM limits across all Claude models and organizations. Limits depend on the applicable service configuration and usage tier.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://platform.claude.com\&sz=32)

Claude Platform Docs+1



### **8.5 Privacy and Compliance**

| Fact                                | Memorize                                                                                  |
| ----------------------------------- | ----------------------------------------------------------------------------------------- |
| ⭐ Standard commercial API retention | Ordinarily 30 days, subject to exceptions                                                 |
| ⭐ ZDR                               | Arrangement- and feature-dependent                                                        |
| ⭐ Data residency                    | Where data is processed and stored                                                        |
| `inference_geo`                     | Supported inference geography control                                                     |
| Workspace geography                 | Storage and applicable endpoint processing geography                                      |
| Commercial model training           | Customer data not used for training by default, subject to applicable opt-in arrangements |
| PII                                 | Personally Identifiable Information                                                       |
| PHI                                 | Protected Health Information                                                              |
| BAA                                 | Business Associate Agreement                                                              |
| DPA                                 | Data Processing Addendum                                                                  |
| HIPAA readiness                     | Requires applicable services, configuration and contractual coverage                      |
| Tenant isolation                    | Must be enforced outside Claude                                                           |
| Stateful features                   | Require specific retention assessment                                                     |
| Auditability                        | Requires evidence, not merely documented policies                                         |

### **8.6 Safety and Fairness Evaluation Metrics**

| Metric                      | What it measures                                                  |
| --------------------------- | ----------------------------------------------------------------- |
| Attack Success Rate (ASR)   | Percentage of adversarial tests achieving their defined objective |
| False acceptance rate       | Unsafe requests incorrectly allowed                               |
| False rejection rate        | Legitimate requests incorrectly blocked                           |
| Guardrail precision         | Fraction of flagged requests that are genuinely unsafe            |
| Guardrail recall            | Fraction of unsafe requests successfully detected                 |
| PII leakage rate            | Frequency of unintended sensitive-data disclosure                 |
| Unauthorized execution rate | Frequency of impermissible actions being executed                 |
| Subgroup accuracy           | Correctness for individual population segments                    |
| Counterfactual consistency  | Stability across matched inputs                                   |
| Human override rate         | Frequency of humans changing AI recommendations                   |
| Approval latency            | Time introduced by human checkpoints                              |

### **8.7 Important Thresholds and Formulas**

| Metric                            | Formula or interpretation                                  |
| --------------------------------- | ---------------------------------------------------------- |
| Risk score                        | Likelihood × Impact                                        |
| Attack success rate               | Successful attacks / Total attempted attacks               |
| False positive rate               | FP / (FP + TN)                                             |
| False negative rate               | FN / (FN + TP)                                             |
| Guardrail precision               | TP / (TP + FP)                                             |
| Guardrail recall                  | TP / (TP + FN)                                             |
| Disparate impact ratio            | Positive outcome rate of evaluated group / Reference group |
| Unauthorized cross-tenant access  | Architectural target: zero                                 |
| Unapproved irreversible execution | Architectural target: zero                                 |

There is no universal Anthropic-mandated ASR threshold, classifier confidence threshold or fairness acceptance threshold for all applications. Acceptance criteria must reflect the business risk, evaluation uncertainty and applicable policy.

### **8.8 Always/Never Rules**

| Always                                                 | Never                                                             |
| ------------------------------------------------------ | ----------------------------------------------------------------- |
| Enforce security-critical authorization outside Claude | Rely solely on system prompts for access control                  |
| Validate tool arguments before execution               | Treat schema conformity as proof of authorization                 |
| Screen external content according to risk              | Treat retrieved text as trusted instructions                      |
| Gate irreversible high-risk actions before execution   | Use self-reported model confidence as the only approval criterion |
| Verify data residency and retention independently      | Assume ZDR satisfies geographic restrictions                      |
| Apply tenant restrictions before retrieval             | Retrieve all tenant data and ask Claude to filter it              |
| Review external Skills and MCP integrations            | Grant unrestricted credentials for convenience                    |
| Limit cumulative agent resource consumption            | Depend exclusively on `max_tokens`                                |
| Evaluate fairness across relevant subgroups            | Infer fairness from aggregate accuracy alone                      |

## **9. Five Exam-Style Questions**

These are original practice questions designed to reflect professional architect-level reasoning, not actual examination questions.

### **Question 1: Guardrail Placement and Prompt Injection ⭐**

An enterprise deploys a Claude-based contract-analysis agent with access to SharePoint through MCP. The agent also has an email tool for sending approved summaries to internal legal teams.

During security testing, a retrieved contract contains malicious instructions attempting to redirect the agent toward sending confidential information to an unauthorized external recipient.

The organization must preserve automated contract analysis and internal summary delivery.

Which architectural change is BEST?

**A.** Strengthen the system prompt and increase Claude's reasoning effort.

**B.** Remove the email tool completely and require legal analysts to send every summary manually.

**C.** Treat retrieved content as untrusted tool results, screen it for injection, and enforce recipient authorization in the email execution service.

**D.** Allow the agent to send emails normally but inspect its final conversational response for confidential information.

Reveal answer and explanation

### **Question 2: Risk Assessment and Ranking**

A multi-tenant Claude-powered research platform has the following characteristics:

| Finding                | Production observation                       |
| ---------------------- | -------------------------------------------- |
| Formatting failures    | 12% of responses                             |
| API throttling         | 3% of requests                               |
| Cross-tenant retrieval | Possible because tenant filters are optional |
| MCP dependency failure | Occasional temporary outages                 |

No cross-tenant disclosure has yet been reported. Management wants to prioritize engineering resources based on incident frequency.

Which action should the architect recommend FIRST?

**A.** Resolve formatting errors because they have the highest observed frequency.

**B.** Increase API limits to prevent throttling and improve availability.

**C.** Replace the unreliable MCP dependency with a direct integration.

**D.** Enforce mandatory tenant authorization at the retrieval layer and perform cross-tenant isolation testing.

Reveal answer and explanation

### **Question 3: Human-in-the-Loop Checkpoints ⭐**

A procurement agent performs the following operations:

1. Searches approved suppliers.
2. Compares quotations.
3. Creates a purchase-order draft.
4. Updates an internal procurement tracker.
5. Submits a legally binding purchase order.
6. Generates a confirmation.

Currently, every operation requires human approval. Processing times have increased significantly, and the organization wants to reduce approval-related delays.

Which architecture is BEST?

**A.** Automate authorized, reversible operations and require approval immediately before submitting the binding purchase order, with approval bound to the exact order details.

**B.** Require approval only for the initial agent plan and automate everything afterward.

**C.** Require approval only when Claude reports confidence below 90%.

**D.** Execute all operations automatically and review completed transactions at the end of each day.

Reveal answer and explanation

### **Question 4: Architectural Controls for Compliance ⭐**

A financial services company plans to deploy Claude for processing confidential customer documents.

Its security policy contains three mandatory requirements:

- Model inference must occur within an approved geographic region.
- Customer documents must not be retained by the model provider after processing.
- Customers must never access another customer's documents.

The product team also wants to use the Files API to simplify document reuse.

Which architecture is BEST?

**A.** Use the Anthropic API with ZDR, enable the Files API, and rely on a system prompt to prevent cross-tenant access.

**B.** Use any cloud-hosted Claude deployment because enterprise cloud services automatically satisfy all three requirements.

**C.** Select a deployment configuration that demonstrably satisfies regional processing and retention obligations, use only retention-eligible features, and enforce tenant authorization before retrieval.

**D.** Encrypt the documents before storing them, retain the existing global inference endpoint, and filter unauthorized information from Claude's final response.

Reveal answer and explanation

### **Question 5: Bias Source Identification and Mitigation**

An enterprise uses Claude to generate personalized professional-development recommendations.

The production evaluation reports lower recommendation-quality scores for one demographic group. However, a blind review by qualified human evaluators finds that the recommendations are equally relevant and actionable across groups.

Further investigation reveals that the LLM-as-judge rubric rewards particular linguistic styles that are more prevalent in one group's responses.

Which corrective action is BEST?

**A.** Replace Claude with a larger model to improve fairness.

**B.** Revise the evaluation rubric to assess task-relevant quality, calibrate the judge against blind human reviews, and repeat subgroup evaluations.

**C.** Remove all demographic information from the generation prompt without modifying the evaluation system.

**D.** Introduce additional few-shot examples into the generation prompt until both groups receive identical aggregate scores.

Reveal answer and explanation

## **10. Exam Decision Framework**

When answering a Domain 5 scenario, evaluate the options in the following order:

```
1. Identify the binding business or compliance constraint.
                       |
2. Locate the vulnerability or failure path.
                       |
3. Determine which trust boundary is violated.
                       |
4. Identify the consequences and reversibility.
                       |
5. Select an enforceable preventive control.
                       |
6. Preserve legitimate business functionality.
                       |
7. Evaluate latency, cost and operational trade-offs.
                       |
8. Ensure monitoring and auditable evidence.
```

### **Rapid Scenario-to-Answer Mapping**

| If the scenario mentions...     | Think immediately about...                |
| ------------------------------- | ----------------------------------------- |
| Malicious retrieved document    | Indirect prompt injection                 |
| Untrusted external response     | Tool-result screening                     |
| Unauthorized tool execution     | Deterministic pre-execution authorization |
| Third-party Skill               | Supply-chain security                     |
| Excessive approvals             | Risk-based HITL                           |
| Irreversible transaction        | Mandatory pre-execution gate              |
| Regional processing requirement | Data residency                            |
| No provider storage             | ZDR and feature eligibility               |
| Customer data leakage           | Tenant isolation                          |
| Runaway agent                   | Cumulative execution budgets              |
| Duplicate transaction           | Idempotency                               |
| Disparate subgroup outcomes     | Bias source diagnosis                     |
| Judge-human disagreement        | Evaluator calibration                     |
| Sensitive information in logs   | Data minimization                         |

## **Last-Minute Revision Checklist**

- ⭐ Understand defense in depth and correct guardrail placement.
- ⭐ Distinguish direct injection from indirect injection and enforce trust boundaries.
- Understand least privilege, MCP security, Skill auditing and sandbox limitations.
- Rank risks using likelihood, impact, exposure, reversibility and blast radius.
- ⭐ Place human approval immediately before consequential, irreversible actions.
- ⭐ Distinguish data residency, data retention, ZDR and model-training policies.
- Understand feature-specific ZDR eligibility and stateful API limitations.
- Enforce tenant isolation, PII protection and privacy-preserving audit logging.
- Understand agent budgets, rate limits, circuit breakers and idempotency.
- ⭐ Diagnose data, prompt, retrieval and evaluation bias before selecting a mitigation.
