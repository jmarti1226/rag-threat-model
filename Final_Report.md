# Adversarial Threat Modeling of a RAG-Enabled Corporate AI Assistant

**Jose Martinez**
College of Science, Technology, Engineering and Mathematics, California State University San Marcos,
333 S. Twin Oaks Valley Rd., San Marcos, CA 92096
jmarti1226@gmail.com

**CYBR 472 — Applied Cybersecurity | Spring 2026 | Dr. Joshua Harguess**

---

## Abstract

Retrieval-Augmented Generation (RAG) has rapidly become the dominant architecture for deploying large language models (LLMs) in enterprise environments. By combining a database of proprietary documents with an external LLM, organizations enable employees to query internal knowledge through a natural language interface. However, this architecture introduces a distinct and underexplored attack surface that traditional security frameworks were not designed to address.

This report presents a comprehensive, adversarial threat model of a conceptual RAG-enabled corporate AI assistant. Using a dual-framework methodology, STRIDE for systematic threat enumeration and MITRE ATLAS for AI-specific attack intelligence, this analysis identifies 48 discrete threats across eight system components. These threats are mapped to specific, actionable technical mitigations in a Threat Traceability Matrix, and validated through two tabletop exercise scenarios that simulate realistic corporate attack chains.

The central finding of this analysis is that the greatest risk to a RAG system does not come from the LLM itself, but from the data pipeline that feeds it. Specifically, the document ingestion pathway (from upload through chunking through embedding storage) is the entry point for the most severe and persistent attacks. A security model that focuses exclusively on the LLM layer while neglecting the ingestion layer is fundamentally incomplete.

Keywords: threat modeling, retrieval-augmented generation, RAG security, STRIDE, MITRE ATLAS, prompt injection, AI security, large language model


---

## 1. Introduction

Any business that wants to use AI without exposing its employees' data, violating compliance
requirements, or suffering a reputational breach should care about this project. More
specifically, this analysis is directly relevant to:

- **Security and IT teams** responsible for evaluating AI deployments before they go live
- **Legal and compliance teams** managing data governance obligations (GDPR, CCPA, HIPAA)
  for organizations where the AI assistant touches sensitive employee or customer data
- **Executives and product owners** making the build-vs-buy decision for internal AI tooling
  without a clear view of what security review that decision requires

If this project is successful, it provides a **repeatable safety checklist** that any of these
stakeholders can apply to their own RAG deployment. It moves the conversation from
"we deployed AI" to "we deployed AI and we know what we secured and why." For security
teams specifically, it demonstrates that AI-specific threats like prompt injection [3] and
knowledge base poisoning can be systematically enumerated, they are not mysterious or
unquantifiable, and that each has a concrete, implementable mitigation.

The broader impact is a shift in industry posture away from what this project calls
"lazy AI integration" (deploying capable systems without architectural security review) toward a model where security is a pre-deployment requirement, not a post-incident reaction.

---

## 2. Background

### 2.1 How AI Assistants Are Deployed Today

The most common enterprise AI pattern today is Retrieval-Augmented Generation (RAG):
a company takes an off-the-shelf LLM and connects it to a proprietary document store,
allowing employees to query internal knowledge through a natural language chat interface [6].
This architecture is appealing because it requires no model training, produces answers
grounded in real company data, and can be stood up quickly using available frameworks
such as LangChain and LlamaIndex.

### 2.2 Limits of Current Practice

The speed at which organizations adopt this architecture is the core problem. Companies
chase the new trend and look for the quickest route to productivity gains, but rarely take
the time to conduct the same architectural security review they would apply to a new
customer-facing web application. The result is a growing class of systems that are
functionally capable but architecturally unexamined.

Current security practice for AI deployments tends to be **reactive and ad hoc**:
- Security controls are applied after deployment, in response to incidents rather than
  before deployment in anticipation of them
- Teams apply traditional security frameworks (firewall rules, authentication policies,
  WAF configurations) without accounting for AI-specific attack surfaces
- Threat modeling, when it occurs at all, treats the LLM as a black box and focuses
  exclusively on the network perimeter, missing the data pipeline entirely

Existing frameworks each address part of the problem but not all of it. **STRIDE** [1]
provides a rigorous structural methodology for enumerating threats but was designed
for traditional software systems and has no native vocabulary for AI-specific attacks
like prompt injection [5] or knowledge base poisoning. **MITRE ATLAS** [2] catalogs
real-world AI attack techniques with empirical grounding but does not prescribe a systematic
methodology for applying them to a specific architecture. No existing standard combines
both into a deployable threat modeling process for RAG systems specifically. The OWASP
Top 10 for LLM Applications [3] identifies classes of risk but does not provide a
component-level traceability framework.

---

## 3. Methodology

### 3.1 What Is New in This Approach

This project combines STRIDE [1] with MITRE ATLAS [2] in a structured, component-by-component
threat modeling process applied to a conceptual RAG Reference Architecture. Instead of
guessing what might go wrong, this analysis uses a double-layered approach: the
architecture's Data Flow Diagram identifies where threats structurally can exist, and
MITRE ATLAS [2] fills in the empirical record of how those threats have actually been
exploited in the wild. The result is a threat model that is both structurally complete
and grounded in real attack history. It is data-driven rather than purely theoretical.

The choice of a **conceptual Reference Architecture** rather than a specific vendor
implementation was also deliberate. A reference architecture allows for security-by-design
analysis free from proprietary constraints, and the findings transfer to any RAG deployment [6]
regardless of the specific tools chosen (LangChain, LlamaIndex, OpenAI, Pinecone, etc.).

### 3.2 Reference Architecture

The Reference Architecture consists of eight components organized into two operational
pipelines: a **Query Pipeline** that processes employee queries in real time, and a
**Document Ingestion Pipeline** that processes new documents into the knowledge base.

| ID | Component | Role |
|---|---|---|
| C1 | Chat UI | Browser-based interface for employee interaction |
| C2 | API Gateway/Auth | Authentication, authorization, rate limiting |
| C3 | Orchestrator | Core logic, manages prompt construction and pipeline flow |
| C4 | Embedding Model API | Converts text to vectors for storage and search |
| C5 | Vector Database | Stores document embeddings; performs similarity search |
| C6 | LLM API | External model that generates the final response |
| C7 | Document Store | Internal file storage for raw corporate documents |
| C8 | Ingestion Pipeline | Chunking, cleaning, and embedding logic for new documents |

Four trust boundaries define where data crosses between zones of different privilege:

| Boundary | Zone Transition |
|---|---|
| TB1: Internet Perimeter | Employee browser ↔ Chat UI |
| TB2: App/Internal Services | API Gateway ↔ Orchestrator and internal services |
| TB3: Internal/External API | Orchestrator ↔ LLM API and Embedding Model (leaves corporate network) |
| TB4: App/Data Layer | Orchestrator and Ingestion Pipeline ↔ Vector DB and Document Store |

The full Data Flow Diagram with all labeled flows is provided as a companion document
([DFD.md](DFD.md)).

### 3.3 Why This Approach Will Succeed

The dual-framework methodology is stronger than either framework alone because the
two tools compensate for each other's limitations. STRIDE [1] ensures no threat category
is missed for any component. ATLAS [2] ensures that the
AI-specific threats within those categories are grounded in documented real-world
attacks rather than speculation. Together they produce a model that a security team
can defend to a skeptical stakeholder: "we identified this threat because it fits the
structural pattern, and we know it is real because it has been executed against deployed
systems."

### 3.4 Project Logistics

**Risks:**
- The primary technical risk is that AI attacks evolve faster than research. MITRE ATLAS [2]
  is a living framework so technique
  mappings require ongoing verification.
- There is also a risk of "over-securing," where controls become so restrictive that the
  AI assistant is no longer useful to employees. Mitigations were selected with usability
  in mind throughout.
- Academically, the risk is ensuring the Reference Architecture is realistic enough to
  produce findings applicable to real-world deployments.

**Cost:** Zero financial cost. This project utilizes open-source frameworks (MITRE ATLAS [2],
STRIDE [1]) and open-source tooling references (Rebuff [7], NeMo Guardrails [8]).

**Timeline:** A security team applying this framework to their own RAG deployment can
expect the following implementation timeline:
- Weeks 1–2: Map the organization's specific architecture to the Reference Architecture;
  produce a customized Data Flow Diagram with trust boundaries
- Weeks 2–4: Apply STRIDE across all components; map threats to MITRE ATLAS techniques
  relevant to the deployment
- Weeks 4–5: Build the Threat Traceability Matrix with organization-specific risk ratings
  and mitigations
- Weeks 5–6: Conduct at least one tabletop exercise scenario to validate the model
- Weeks 6–8: Begin implementing Critical and High-rated mitigations in priority order

An experienced security team familiar with the RAG architecture could compress this to
four to six weeks. A team new to AI threat modeling should expect the full eight.

---

## 4. Results & Discussion

### 4.1 Data Flow Diagram and Top 10 Threat List

**Success Criterion:** Completion of a detailed Data Flow Diagram and a Top 10 list of identified
threats across the RAG pipeline.

**Outcome:** Both delivered. The Data Flow Diagram documents all eight components,
sixteen labeled data flows, and four trust boundaries across both the query and ingestion
pipelines. It is provided as a rendered Mermaid flowchart in [DFD.md](DFD.md).

The Top 10 threat list, produced from an initial pass of the STRIDE [1] analysis, identified
the highest-priority threats by exploitability and potential impact:

| Rank | Threat | Why High Priority |
|---|---|---|
| 1 | Indirect Prompt Injection via retrieved chunks [4] | Bridges document poisoning to LLM manipulation; hard to detect |
| 2 | RAG Poisoning (vector DB tampering) | Attacker controls what the LLM "knows" without touching the LLM |
| 3 | Document Poisoning at upload | Entry point for the entire poisoning attack chain |
| 4 | LLM Jailbreak | Bypasses all system prompt guardrails |
| 5 | Direct Prompt Injection [5] | User directly manipulates LLM behavior |
| 6 | Corporate data exfiltration via prompt to LLM API | Sensitive data leaves the corporate network on every query |
| 7 | Privilege escalation via tool-use injection [4] | Attacker executes actions via the LLM |
| 8 | Corporate text sent to external embedding API | Data exposure at ingestion time, often overlooked |
| 9 | Auth bypass on API gateway | Entire downstream pipeline exposed without auth |
| 10 | Unauthorized read access to vector DB | Exposes the full embedded corporate knowledge base |

### 4.2 Threat Traceability Matrix

**Success Criterion:** A completed Threat Traceability Matrix where 100% of identified threats are
mapped to at least one specific, actionable technical mitigation.

**Outcome:** Delivered. The full matrix ([Threat_Traceability_Matrix.md](Threat_Traceability_Matrix.md)) maps
**48 threats**, six per component across all eight components, to specific mitigations
with risk ratings and mitigation types. Coverage is 100%.

**Risk Distribution:**

| Risk Level | Count | Percentage |
|---|---|---|
| Critical | 6 | 12.5% |
| High | 29 | 60.4% |
| Medium | 13 | 27.1% |
| **Total** | **48** | **100%** |

The absence of Low-rated threats reflects the nature of the architecture: every component
handles either sensitive corporate data, external network communication, or both.

Of the 48 threats, 17 map to one of six MITRE ATLAS [2] techniques:

| ATLAS Technique | Name | Threats |
|---|---|---|
| AML.T0051.000 | LLM Prompt Injection (Direct) [5] | 1 |
| AML.T0051.001 | LLM Prompt Injection (Indirect) [4] | 4 |
| AML.T0020 | Poison Training/Knowledge Data | 4 |
| AML.T0035 | ML Artifact Collection | 6 |
| AML.T0043 | Craft Adversarial Data | 1 |
| AML.T0054 | LLM Jailbreak | 1 |

> All technique IDs sourced from MITRE ATLAS [2] (atlas.mitre.org).

### 4.3 Key Finding — The Attack Chain

The most significant analytical result was the identification of a connected, multi-stage
attack chain that bypasses all UI-layer and query-layer controls entirely. This attack
pattern is consistent with documented indirect prompt injection research [4]:

```
[Attacker uploads poisoned document]        ← T-C7-T1
        ↓
[Malicious content ingested into vector DB] ← T-C5-T1
        ↓
[Poisoned chunk retrieved for a user query] ← T-C3-T1
        ↓
[Indirect Prompt Injection executes]        ← T-C3-E1 / T-C5-E1
        ↓
[Sensitive data disclosed or actions executed]
```

This finding reframes where security investment should be concentrated. An organization
that hardens the system prompt and deploys output filtering, but neglects the ingestion pipeline, has secured the wrong layer. The attack never touches the chat interface.

### 4.4 Validation — Tabletop Exercise

Two tabletop exercise scenarios were conducted to validate that the threat model
accurately represents exploitable attack paths (full documentation in [Tabletop_Exercise.md](Tabletop_Exercise.md)).

**Scenario 1 — The Poisoned Policy Document**
A malicious insider embeds a prompt injection payload [4] in a legitimate-looking HR policy
PDF using white-font hidden text. The document passes upload controls not configured
for AI-specific patterns, is ingested into the vector database, and weeks later surfaces
in a legitimate employee's query response — causing the LLM to display other employees'
salary data without alerting the user.

*Result:* The attack succeeded under realistic conditions. The primary gap: content
scanners at the document upload layer are tuned for malware and PII, not prompt
injection syntax. The ingestion pipeline was confirmed as the weakest link.

**Scenario 2 — The Persistent Interrogator**
An attacker systematically probes the chat interface to extract the system prompt through
rephrasing techniques [5], then crafts a DAN-style jailbreak.

*Result:* Largely mitigated by pre-deployment red-team testing. Scenario 2 is
substantially less dangerous than Scenario 1 because it requires direct interface
interaction — leaving an audit trail — and is bounded by rate limiting and logging.

**Cross-Scenario Conclusion:** The tabletop exercise validated the threat model's
central claim. Query-layer attacks (Scenario 2) are well-understood and manageable [3].
Ingestion-layer attacks (Scenario 1) are more dangerous precisely because they require
no direct interaction with the AI system — the attacker poisons the knowledge base
and waits [4].

---

## 5. Conclusion

This project demonstrated that the security challenges of a RAG-enabled corporate AI assistant are not mysterious or unquantifiable. They are enumerable, they have known attack patterns [2][3][4][5], and they have concrete mitigations. The 48-threat model, Threat Traceability Matrix, and tabletop exercise together constitute a repeatable framework that any security team can apply to their own deployment.

The most important practical finding: that the ingestion pipeline is the highest-risk surface, not the LLM or the chat interface; has direct implications for how organizations should prioritize security investment. Content moderation applied symmetrically across both pipelines, AI-specific document scanning at upload using tools such as Rebuff [7] and NeMo Guardrails [8], and full audit logging are the controls that matter most.

The framework developed here is transferable. Any organization deploying a RAG assistant [6] can substitute their specific tools and document types and produce a threat model grounded in both structural security engineering [1] and empirical AI attack intelligence [2].


**Relation to Capstone Project:**
This semester project is the applied analysis branch of the Capstone Project, which is building an AI Threat Intelligence Aggregator, a system that collects and aggregates real-world AI attack data from MITRE ATLAS [2]. The two projects form a complementary ecosystem: the Capstone is the engine that collects and maintains current threat intelligence; this semester project is the laboratory where that intelligence is applied to a specific Reference Architecture to produce actionable defenses.

---

## References

[1]	Microsoft, "The STRIDE threat model," Microsoft Security Documentation (2009). [Online]. Available: https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats 

[2]	MITRE Corporation, "MITRE ATLAS: Adversarial Threat Landscape for Artificial Intelligence Systems," (2023). [Online]. Available: https://atlas.mitre.org 

[3]	OWASP, "Top 10 for Large Language Model Applications," OWASP Foundation (2023). [Online]. Available: https://owasp.org/www-project-top-10-for-large-language-model-applications/ 

[4]	Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., and Fritz, M., "Not what you've signed up for: Compromising real-world LLM-integrated applications with indirect prompt injection," arXiv preprint arXiv:2302.12173 (2023). 

[5]	Perez, E. and Ribeiro, I., "Ignore previous prompt: Attack techniques for language models," arXiv preprint arXiv:2211.09527 (2022). 

[6]	Lewis, P. et al., "Retrieval-augmented generation for knowledge-intensive NLP tasks," arXiv preprint arXiv:2005.11401 (2020). 

[7]	Rebuff AI, "Rebuff: Detecting prompt injection attacks," GitHub (2023). [Online]. Available: https://github.com/protectai/rebuff

[8]	NVIDIA, "NeMo Guardrails: A toolkit for controllable and safe LLM applications," GitHub (2023). [Online]. Available: https://github.com/NVIDIA/NeMo-Guardrails 


---

## Appendix A — Project Deliverables Index

| Deliverable | File | Description |
|---|---|---|
| Data Flow Diagram | [DFD.md](DFD.md) | Reference Architecture with components, flows, and trust boundaries |
| STRIDE Analysis | [STRIDE_Analysis.md](STRIDE_Analysis.md) | Full 48-threat enumeration with ATLAS mappings |
| Threat Traceability Matrix | [Threat_Traceability_Matrix.md](Threat_Traceability_Matrix.md) | All threats mapped to specific mitigations with risk ratings |
| Tabletop Exercise | [Tabletop_Exercise.md](Tabletop_Exercise.md) | Two attack scenarios validating the threat model |
| Final Report | `Final_Report.md` | This document |
