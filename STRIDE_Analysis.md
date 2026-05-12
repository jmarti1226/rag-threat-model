# STRIDE Threat Analysis — RAG-Enabled Corporate AI Assistant
**CYBR 472 | Jose Martinez | Spring 2026**

---

## How to Read This Document

Each component from the DFD (C1–C8) is analyzed against all six STRIDE categories.
Each identified threat is assigned a **Threat ID** (used later in the Threat Traceability Matrix),
a **STRIDE category**, an **affected data flow** from the DFD, a **MITRE ATLAS technique**
where applicable, and a **preliminary mitigation**.


---

## STRIDE Quick Reference

| Letter | Category | Question to ask |
|---|---|---|
| S | Spoofing | Can an attacker pretend to be a legitimate user or component? |
| T | Tampering | Can an attacker modify data in transit or at rest? |
| R | Repudiation | Can a user or attacker deny having performed an action? |
| I | Information Disclosure | Can sensitive data be read by an unauthorized party? |
| D | Denial of Service | Can an attacker degrade or break availability? |
| E | Elevation of Privilege | Can an attacker gain access or permissions they should not have? |

---

## C1 — Chat UI

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C1-S1 | Spoofing | Session hijacking: attacker steals session token and impersonates a legitimate employee | F1, F2 | — | HttpOnly/Secure cookies; short-lived session tokens; re-auth on sensitive actions |
| T-C1-T1 | Tampering | XSS injection into the chat UI intercepts or modifies responses displayed to the user | F11 | — | Content Security Policy (CSP); output encoding; input sanitization |
| T-C1-R1 | Repudiation | No record of which employee submitted which query, preventing audit or forensics | F1 | — | Log all queries with authenticated user ID and timestamp |
| T-C1-I1 | Info Disclosure | Sensitive AI responses cached in browser history or stored in localStorage | F11 | — | Disable client-side caching for response data; clear session on logout |
| T-C1-D1 | Denial of Service | Browser-layer flooding or automated scripts submit high-volume queries | F1 | — | Rate limiting at UI layer; CAPTCHA for unauthenticated access |
| T-C1-E1 | Elevation of Privilege | XSS leads to session token exfiltration, allowing attacker to act as a higher-privileged user | F2 | — | CSP; token binding; enforce least-privilege roles |

---

## C2 — API Gateway / Auth

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C2-S1 | Spoofing | Forged or replayed JWT tokens used to authenticate as another employee | F2, F3 | — | Short token expiry; token revocation list; mutual TLS |
| T-C2-T1 | Tampering | Attacker modifies request headers (e.g., user role) between the UI and gateway | F2 | — | Sign all internal tokens; validate claims server-side |
| T-C2-R1 | Repudiation | Gateway does not log rejected requests, hiding brute-force or scanning attempts | F2 | — | Log all requests (success and failure) with source IP and user ID |
| T-C2-I1 | Info Disclosure | Verbose error messages reveal internal system structure or valid usernames | F2 | — | Generic error responses; suppress stack traces in production |
| T-C2-D1 | Denial of Service | Rate limit bypass via distributed requests or header manipulation | F2 | — | IP-based and account-based rate limiting; WAF rules |
| T-C2-E1 | Elevation of Privilege | Auth bypass vulnerability grants attacker access to admin or privileged API endpoints | F3 | — | Mandatory auth on all routes; deny-by-default access control |

---

## C3 — Orchestrator (LangChain / LlamaIndex)

> The Orchestrator is the highest-value target: it touches every component and constructs
> the full prompt sent to the LLM. Compromise here affects the entire pipeline.

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C3-S1 | Spoofing | Attacker intercepts internal service calls and impersonates the orchestrator to extract LLM responses | F8, F9 | — | Mutual TLS between orchestrator and all downstream services |
| T-C3-T1 | Tampering | Indirect Prompt Injection: malicious content in retrieved document chunks overwrites system prompt instructions | F7 → F8 | AML.T0051.001 | Prompt structure hardening; separate system/user/context sections; output validation |
| T-C3-R1 | Repudiation | No logging of the full constructed prompt, impossible to audit what was sent to the LLM | F8 | — | Log all prompts (redacted if needed) with request ID and timestamp |
| T-C3-I1 | Info Disclosure | The full prompt (including retrieved corporate document chunks) is sent in plaintext to an external LLM API | F8 | AML.T0035 | TLS in transit; evaluate on-premise LLM options for sensitive data; data classification before retrieval |
| T-C3-D1 | Denial of Service | Adversarial queries trigger recursive LLM calls or runaway tool-use chains, exhausting API budget | F8 | — | Set hard token/call limits per request; circuit breaker pattern |
| T-C3-E1 | Elevation of Privilege | Indirect Prompt Injection tricks the orchestrator into executing privileged tool calls (e.g., file writes, external API calls) on behalf of the attacker | F7 → F8 | AML.T0051.001 | Restrict tool-use permissions; human-in-the-loop for destructive actions; allowlist of permitted tool calls |

---

## C4 — Embedding Model API

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C4-S1 | Spoofing | DNS hijacking or MITM substitutes a malicious embedding endpoint, returning crafted vectors | F4, I3 | — | Certificate pinning; verify endpoint identity; monitor embedding output distributions |
| T-C4-T1 | Tampering | Adversarial input crafted to manipulate the embedding space, causing unrelated documents to appear similar | F4, I3 | AML.T0043 | Input validation; anomaly detection on retrieved chunk relevance scores |
| T-C4-R1 | Repudiation | No audit of what text was sent to the embedding API | F4, I3 | — | Log all embedding requests with content hash |
| T-C4-I1 | Info Disclosure | Corporate document text is transmitted to an external embedding API provider | I3 | AML.T0035 | Use a self-hosted embedding model for sensitive data; review provider data retention policy |
| T-C4-D1 | Denial of Service | External embedding API outage brings down the entire query and ingestion pipeline | F4, F5, I3, I4 | — | Fallback to cached embeddings; retry logic with exponential backoff; local embedding model |
| T-C4-E1 | Elevation of Privilege | Embedding Inversion: attacker with access to stored vectors reconstructs the original source text | F5, I4 | AML.T0035 | Encrypt vectors at rest; restrict read access to the vector DB |

---

## C5 — Vector Database

> The Vector DB is the injection point for RAG poisoning. Whatever is stored here
> directly shapes what the LLM is told is true.

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C5-S1 | Spoofing | Attacker gains write access and inserts embeddings attributed to legitimate source documents | I5, F6 | AML.T0020 | Strict write-access controls; sign metadata at ingestion time |
| T-C5-T1 | Tampering | **RAG Poisoning** — malicious document chunks injected into the vector store surface in LLM responses | I5, F7 | AML.T0020 | Document provenance tracking; content integrity hashes; human review of ingestion |
| T-C5-R1 | Repudiation | No audit log of when embeddings were inserted, modified, or deleted | I5 | — | Immutable audit log for all vector DB write operations |
| T-C5-I1 | Info Disclosure | Unauthorized read access to the vector DB exposes all embedded corporate knowledge | F6, F7 | AML.T0035 | Role-based access control; encrypt at rest; network segmentation |
| T-C5-D1 | Denial of Service | Flooding similarity search with high-dimensional queries degrades or crashes retrieval | F6 | — | Query timeout limits; resource quotas per user session |
| T-C5-E1 | Elevation of Privilege | Injected chunks contain prompt injection payloads that, when retrieved, instruct the LLM to act with elevated authority | F7 → F8 | AML.T0051.001 | Output filtering; content moderation on retrieved chunks before prompt insertion |

---

## C6 — LLM API (External)

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C6-S1 | Spoofing | Attacker redirects LLM API calls to a malicious or substitute model endpoint | F8, F9 | — | Hardcode and pin the API endpoint; validate response signatures if available |
| T-C6-T1 | Tampering | Direct Prompt Injection: user crafts a query that overrides the system prompt | F8 | AML.T0051.000 | System prompt hardening; prompt delimiters; instruction hierarchy enforcement |
| T-C6-R1 | Repudiation | No logging of full prompts and responses sent to/from the external LLM | F8, F9 | — | Store prompt/response logs with request ID; establish data retention policy |
| T-C6-I1 | Info Disclosure | Corporate data in the prompt is processed and potentially retained by a third-party LLM provider | F8 | AML.T0035 | Review provider Terms of Service and data handling; use enterprise API tiers with no-training agreements |
| T-C6-D1 | Denial of Service | Token exhaustion attack: adversarial queries maximize token usage per request, driving up cost and latency | F8 | — | Hard token limits per request; cost alerts; request timeout |
| T-C6-E1 | Elevation of Privilege | LLM Jailbreak: attacker bypasses system prompt restrictions to make the model produce unauthorized content or reveal system instructions | F8 | AML.T0054 | Output content filtering; red-team testing of system prompt robustness |

---

## C7 — Document Store

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C7-S1 | Spoofing | Attacker impersonates an admin or document owner to upload malicious files | I1 | — | MFA for document upload; role-based upload permissions |
| T-C7-T1 | Tampering | Document Poisoning: malicious actor embeds prompt injection payloads inside a PDF or DOCX that appears legitimate | I1, I2 | AML.T0020 | Content scanning at upload (YARA rules, keyword detection); document approval workflow |
| T-C7-R1 | Repudiation | No audit trail of who uploaded, modified, or deleted which documents | I1 | — | Immutable upload log with uploader identity and timestamp |
| T-C7-I1 | Info Disclosure | Unauthorized access to the document store exposes all raw corporate files | I2 | — | ACL on document store; encrypt at rest; network segmentation |
| T-C7-D1 | Denial of Service | Storage exhaustion via bulk large-file uploads | I1 | — | File size limits; upload quotas per user; storage monitoring |
| T-C7-E1 | Elevation of Privilege | A document containing role-elevation instructions is processed and causes the LLM to treat a regular user as an admin | I1 → I5 → F7 → F8 | AML.T0051.001 | Role information must come from the auth layer, never from LLM-generated content |

---

## C8 — Ingestion Pipeline

| Threat ID | STRIDE | Threat Description | Affected Flow | ATLAS Technique | Mitigation |
|---|---|---|---|---|---|
| T-C8-S1 | Spoofing | Attacker mimics the ingestion service to inject fake pre-computed embeddings directly into the vector DB | I5 | — | Authenticate ingestion pipeline service account; sign embeddings at creation |
| T-C8-T1 | Tampering | Malformed documents manipulate chunking logic to split content in ways that obscure embedded malicious instructions | I2, I3 | AML.T0020 | Validate chunk output; randomize chunk boundaries; anomaly detection on chunk content |
| T-C8-R1 | Repudiation | No logging of chunking decisions or which source document produced which embedding | I2–I5 | — | Log source document hash, chunk index, and resulting embedding ID for full traceability |
| T-C8-I1 | Info Disclosure | Chunked corporate text transmitted to an external embedding API without sanitization | I3 | AML.T0035 | PII/sensitive data detection before embedding; data masking where possible |
| T-C8-D1 | Denial of Service | Malformed or specially crafted documents crash the chunking/parsing library | I2 | — | Sandboxed document parsing; timeout limits on ingestion jobs; dead-letter queue for failed documents |
| T-C8-E1 | Elevation of Privilege | If the ingestion pipeline runs with elevated OS privileges, a compromised parser could access other internal systems | I2 | — | Run ingestion pipeline in a least-privilege container; no network access except to the embedding API and vector DB |

---

## Threat Summary Count

| Component | S | T | R | I | D | E | Total |
|---|---|---|---|---|---|---|---|
| C1 Chat UI | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| C2 API Gateway | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| C3 Orchestrator | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| C4 Embedding Model | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| C5 Vector Database | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| C6 LLM API | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| C7 Document Store | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| C8 Ingestion Pipeline | 1 | 1 | 1 | 1 | 1 | 1 | 6 |
| **Total** | **8** | **8** | **8** | **8** | **8** | **8** | **48** |

---

## High-Priority Threats (Top 10)

These are ranked by exploitability and potential impact in a corporate RAG deployment:

| Rank | Threat ID | Name | Why High Priority |
|---|---|---|---|
| 1 | T-C3-T1 | Indirect Prompt Injection via retrieved chunks | Bridges document poisoning to LLM manipulation; hard to detect |
| 2 | T-C5-T1 | RAG Poisoning (vector DB tampering) | Attacker controls what the LLM "knows" without touching the LLM |
| 3 | T-C7-T1 | Document Poisoning at upload | Entry point for the entire poisoning attack chain |
| 4 | T-C6-E1 | LLM Jailbreak | Bypasses all system prompt guardrails |
| 5 | T-C6-T1 | Direct Prompt Injection | User directly manipulates LLM behavior |
| 6 | T-C3-I1 | Corporate data exfiltration via prompt to LLM API | Sensitive data leaves the corporate network on every query |
| 7 | T-C3-E1 | Privilege escalation via tool-use injection | Attacker executes actions (file writes, API calls) via the LLM |
| 8 | T-C4-I1 | Corporate text sent to external embedding API | Data exposure at ingestion time, often overlooked |
| 9 | T-C2-E1 | Auth bypass on API gateway | Entire downstream pipeline exposed without auth |
| 10 | T-C5-I1 | Unauthorized read access to vector DB | Exposes the full embedded corporate knowledge base |
