# Threat Traceability Matrix — RAG-Enabled Corporate AI Assistant
**CYBR 472 | Jose Martinez | Spring 2026**

---

## How to Read This Matrix

| Column | Description |
|---|---|
| Threat ID | Unique identifier from the STRIDE Analysis document |
| Component | System component from the DFD (C1–C8) |
| STRIDE | Threat category |
| Threat | Short name for the threat |
| ATLAS Technique | MITRE ATLAS technique ID (verify at atlas.mitre.org) |
| Likelihood | How likely is exploitation in a real corporate deployment (H/M/L) |
| Impact | Business/security impact if exploited (H/M/L) |
| Risk | Overall risk rating derived from Likelihood × Impact |
| Mitigation | Specific, actionable control to address the threat |
| Mitigation Type | Preventive (stops it), Detective (detects it), Corrective (recovers from it) |

### Risk Rating Logic

| Likelihood/Impact | Low | Medium | High |
|---|---|---|---|
| **High** | Medium | High | Critical |
| **Medium** | Low | Medium | High |
| **Low** | Low | Low | Medium |

---

## Full Threat Traceability Matrix

| Threat ID | Component | STRIDE | Threat | ATLAS Technique | Likelihood | Impact | Risk | Mitigation | Mitigation Type |
|---|---|---|---|---|---|---|---|---|---|
| T-C1-S1 | C1 Chat UI | Spoofing | Session hijacking: attacker steals session token and impersonates an employee | — | Medium | High | High | HttpOnly + Secure cookie flags; short-lived tokens (15 min); force re-auth on sensitive actions | Preventive |
| T-C1-T1 | C1 Chat UI | Tampering | XSS injection modifies or intercepts AI responses displayed to the user | — | Medium | Medium | Medium | Content Security Policy (CSP) header; output encoding on all rendered content; sanitize user input before display | Preventive |
| T-C1-R1 | C1 Chat UI | Repudiation | No record of which employee submitted which query | — | High | Medium | High | Log all queries with authenticated user ID, session ID, and UTC timestamp; store in tamper-evident log | Detective |
| T-C1-I1 | C1 Chat UI | Info Disclosure | Sensitive AI responses cached in browser history or localStorage | — | Medium | Medium | Medium | Disable client-side caching for response payloads (Cache-Control: no-store); clear session data on logout | Preventive |
| T-C1-D1 | C1 Chat UI | Denial of Service | Automated scripts flood the UI with high-volume queries | — | Medium | Medium | Medium | Client-side rate limiting; server-side throttling per session; CAPTCHA after threshold | Preventive |
| T-C1-E1 | C1 Chat UI | Elevation of Privilege | XSS leads to session token exfiltration; attacker acts as a higher-privileged user | — | Medium | High | High | CSP blocks inline scripts; token binding to device fingerprint; enforce least-privilege roles at the API layer | Preventive |
| T-C2-S1 | C2 API Gateway | Spoofing | Forged or replayed JWT tokens used to authenticate as another employee | — | Medium | High | High | Short token expiry (15 min); server-side token revocation list; mutual TLS for service-to-service calls | Preventive |
| T-C2-T1 | C2 API Gateway | Tampering | Attacker modifies request headers (e.g., user role claims) between UI and gateway | — | Low | High | Medium | Sign all internal tokens (RS256 JWT); validate all claims server-side; reject unsigned or modified tokens | Preventive |
| T-C2-R1 | C2 API Gateway | Repudiation | Gateway does not log rejected requests, hiding attack attempts | — | High | Medium | High | Log all requests (success and failure) with source IP, user ID, endpoint, and HTTP status code | Detective |
| T-C2-I1 | C2 API Gateway | Info Disclosure | Verbose error messages reveal internal system structure or valid usernames | — | High | Medium | High | Return generic error messages in production; suppress stack traces; use error reference codes instead | Preventive |
| T-C2-D1 | C2 API Gateway | Denial of Service | Rate limit bypass via distributed requests or manipulated headers | — | Medium | High | High | IP-based and account-based rate limiting; WAF rules; automatic IP block after threshold | Preventive |
| T-C2-E1 | C2 API Gateway | Elevation of Privilege | Auth bypass grants attacker access to admin or privileged API endpoints | — | Low | High | Medium | Deny-by-default access control; mandatory auth on all routes; regular penetration testing of auth logic | Preventive |
| T-C3-S1 | C3 Orchestrator | Spoofing | Attacker impersonates the orchestrator to intercept LLM API responses | — | Low | High | Medium | Mutual TLS between orchestrator and all downstream services; service mesh with identity verification | Preventive |
| T-C3-T1 | C3 Orchestrator | Tampering | **Indirect Prompt Injection**: malicious content in retrieved chunks overrides system prompt instructions | AML.T0051.001 | High | Critical | Critical | Prompt structure hardening (clear delimiters between system/context/user); output validation layer; treat retrieved content as untrusted data | Preventive + Detective |
| T-C3-R1 | C3 Orchestrator | Repudiation | No logging of constructed prompts: impossible to audit what was sent to the LLM | — | High | High | High | Log all constructed prompts with request ID and timestamp; redact PII before storage; retain logs for 90 days | Detective |
| T-C3-I1 | C3 Orchestrator | Info Disclosure | Full prompt including corporate document chunks sent in plaintext to external LLM | AML.T0035 | High | High | High | TLS in transit (verified); evaluate on-premise LLM for sensitive data; classify data before retrieval; minimize chunks sent | Preventive |
| T-C3-D1 | C3 Orchestrator | Denial of Service | Adversarial queries trigger runaway LLM call chains, exhausting API budget | — | Medium | High | High | Hard token/call limits per request; circuit breaker pattern; cost alert thresholds; async queue with job limits | Preventive |
| T-C3-E1 | C3 Orchestrator | Elevation of Privilege | Indirect Prompt Injection tricks orchestrator into executing privileged tool calls (file writes, external APIs) | AML.T0051.001 | High | Critical | Critical | Restrict tool-use to allowlist; require human approval for destructive actions; never derive permissions from LLM output | Preventive |
| T-C4-S1 | C4 Embedding Model | Spoofing | DNS hijacking substitutes a malicious embedding endpoint returning crafted vectors | — | Low | High | Medium | Certificate pinning on embedding API endpoint; monitor embedding output distribution for anomalies | Preventive + Detective |
| T-C4-T1 | C4 Embedding Model | Tampering | Adversarial inputs manipulate the embedding space: unrelated documents appear semantically similar | AML.T0043 | Medium | High | High | Input validation before embedding; monitor cosine similarity scores for anomalous retrieval patterns | Detective |
| T-C4-R1 | C4 Embedding Model | Repudiation | No audit of what text was submitted to the embedding API | — | High | Medium | High | Log all embedding requests with content hash and timestamp; retain for traceability | Detective |
| T-C4-I1 | C4 Embedding Model | Info Disclosure | Corporate document text transmitted to an external embedding API provider | AML.T0035 | High | High | High | Self-hosted embedding model for sensitive data (e.g., sentence-transformers locally); review provider data retention policy | Preventive |
| T-C4-D1 | C4 Embedding Model | Denial of Service | External embedding API outage brings down query and ingestion pipelines | — | Medium | High | High | Fallback to cached embeddings; local embedding model as backup; retry with exponential backoff | Corrective |
| T-C4-E1 | C4 Embedding Model | Elevation of Privilege | Embedding inversion: attacker with vector access reconstructs original source text | AML.T0035 | Medium | High | High | Encrypt vector DB at rest; restrict read access with RBAC; do not store raw text alongside embeddings | Preventive |
| T-C5-S1 | C5 Vector Database | Spoofing | Attacker gains write access and inserts embeddings attributed to legitimate source documents | AML.T0020 | Medium | High | High | Strict write-access controls (service accounts only); sign embedding metadata at ingestion time; verify signatures on read | Preventive |
| T-C5-T1 | C5 Vector Database | Tampering | **RAG Poisoning**: malicious chunks injected into vector store surface in LLM responses | AML.T0020 | High | Critical | Critical | Document provenance tracking with cryptographic hashes; content integrity verification; human review workflow for new document sources | Preventive + Detective |
| T-C5-R1 | C5 Vector Database | Repudiation | No audit log of embedding insertions, modifications, or deletions | — | High | High | High | Immutable audit log for all vector DB write operations (who, what, when); alerts on bulk deletions | Detective |
| T-C5-I1 | C5 Vector Database | Info Disclosure | Unauthorized read access to vector DB exposes all embedded corporate knowledge | AML.T0035 | Medium | High | High | RBAC on vector DB (read access only for orchestrator service account); encrypt at rest; network segmentation | Preventive |
| T-C5-D1 | C5 Vector Database | Denial of Service | Flooding similarity search with high-dimensional queries crashes retrieval | — | Medium | Medium | Medium | Query timeout limits; per-user resource quotas; index health monitoring | Preventive |
| T-C5-E1 | C5 Vector Database | Elevation of Privilege | Injected chunks contain prompt injection payloads that instruct LLM to act with elevated authority | AML.T0051.001 | High | Critical | Critical | Content moderation scan on retrieved chunks before prompt insertion; output filtering on LLM responses | Preventive + Detective |
| T-C6-S1 | C6 LLM API | Spoofing | LLM API calls redirected to a malicious or substitute model endpoint | — | Low | High | Medium | Hardcode and pin the API endpoint URL; validate TLS certificate on every connection | Preventive |
| T-C6-T1 | C6 LLM API | Tampering | **Direct Prompt Injection**: user query overrides system prompt instructions | AML.T0051.000 | High | High | High | System prompt hardening; strong instruction delimiters; test with adversarial prompt suite | Preventive + Detective |
| T-C6-R1 | C6 LLM API | Repudiation | No logging of prompts and responses sent to/from the external LLM | — | High | High | High | Log full prompt and response with request ID; establish data retention and access policy | Detective |
| T-C6-I1 | C6 LLM API | Info Disclosure | Corporate data in prompts processed and potentially retained by third-party LLM provider | AML.T0035 | High | High | High | Enterprise API tier with zero data-retention agreement; legal review of provider ToS; minimize sensitive data in prompts | Preventive |
| T-C6-D1 | C6 LLM API | Denial of Service | Token exhaustion attack: adversarial queries maximize tokens per request, driving cost and latency | — | High | Medium | High | Hard max-token limit per request; per-user cost tracking; alert on anomalous spend | Preventive |
| T-C6-E1 | C6 LLM API | Elevation of Privilege | **LLM Jailbreak**: attacker bypasses system prompt to produce unauthorized content or reveal system instructions | AML.T0054 | High | High | High | Output content filtering layer; regular red-team testing of system prompt; monitor for system prompt leakage patterns | Preventive + Detective |
| T-C7-S1 | C7 Document Store | Spoofing | Attacker impersonates an admin to upload malicious documents | — | Medium | High | High | MFA required for document upload; role-based upload permissions; upload action logged and alerted | Preventive + Detective |
| T-C7-T1 | C7 Document Store | Tampering | **Document Poisoning**: prompt injection payload embedded inside a PDF or DOCX | AML.T0020 | High | Critical | Critical | Content scanning at upload (keyword detection, YARA rules); document approval workflow before ingestion; strip executable macros | Preventive |
| T-C7-R1 | C7 Document Store | Repudiation | No audit trail of who uploaded, modified, or deleted documents | — | High | High | High | Immutable upload log with uploader identity, file hash, and timestamp; alert on deletions | Detective |
| T-C7-I1 | C7 Document Store | Info Disclosure | Unauthorized access to document store exposes all raw corporate files | — | Medium | High | High | ACL on document store (least-privilege); encrypt at rest (AES-256); network segmentation from public-facing services | Preventive |
| T-C7-D1 | C7 Document Store | Denial of Service | Storage exhaustion via bulk large-file uploads | — | Medium | Medium | Medium | Per-user file size and upload quota limits; storage monitoring with threshold alerts | Preventive |
| T-C7-E1 | C7 Document Store | Elevation of Privilege | Document containing role-elevation instructions causes LLM to treat a regular user as admin | AML.T0051.001 | High | Critical | Critical | Role/permission information must originate only from the auth layer, never from LLM-generated or retrieved content | Preventive |
| T-C8-S1 | C8 Ingestion Pipeline | Spoofing | Attacker mimics the ingestion service to inject fake embeddings directly into the vector DB | — | Low | High | Medium | Service account authentication for ingestion pipeline; sign embeddings at creation; reject unsigned writes to vector DB | Preventive |
| T-C8-T1 | C8 Ingestion Pipeline | Tampering | Malformed documents manipulate chunking to obscure embedded malicious instructions | AML.T0020 | Medium | High | High | Validate chunk output with content filters; anomaly detection on chunk semantic content; fixed chunking strategy | Detective |
| T-C8-R1 | C8 Ingestion Pipeline | Repudiation | No logging of which source document produced which embedding | — | High | Medium | High | Log source document hash, chunk index, and resulting embedding ID for full provenance traceability | Detective |
| T-C8-I1 | C8 Ingestion Pipeline | Info Disclosure | Chunked corporate text transmitted to external embedding API without sanitization | AML.T0035 | High | High | High | PII/sensitive data detection before embedding (e.g., regex + NER scanner); data masking for high-sensitivity fields | Preventive |
| T-C8-D1 | C8 Ingestion Pipeline | Denial of Service | Malformed documents crash the chunking/parsing library | — | Medium | Medium | Medium | Sandboxed document parsing (isolated container); timeout limits on ingestion jobs; dead-letter queue for failed documents | Preventive + Corrective |
| T-C8-E1 | C8 Ingestion Pipeline | Elevation of Privilege | Compromised parser with elevated OS privileges accesses other internal systems | — | Low | High | Medium | Run ingestion pipeline in least-privilege container; restrict network access to embedding API and vector DB only | Preventive |

---

## Risk Distribution Summary

| Risk Level | Count | % of Total |
|---|---|---|
| Critical | 6 | 12.5% |
| High | 29 | 60.4% |
| Medium | 13 | 27.1% |
| Low | 0 | 0% |
| **Total** | **48** | **100%** |

---

## Critical Threats Summary

These six threats warrant immediate priority in any real deployment:

| Threat ID | Threat | Component | Why Critical |
|---|---|---|---|
| T-C3-T1 | Indirect Prompt Injection | Orchestrator | Weaponizes the retrieval system itself; hard to detect at runtime |
| T-C3-E1 | Tool-Use Privilege Escalation | Orchestrator | Attacker executes privileged actions through the LLM with no direct system access |
| T-C5-T1 | RAG Poisoning | Vector Database | Attacker controls LLM knowledge without touching the LLM or the network |
| T-C5-E1 | Prompt Injection via Retrieved Chunks | Vector Database | Persistent attack, payload survives across all future queries until remediated |
| T-C7-T1 | Document Poisoning | Document Store | Entry point for the entire poisoning chain; low technical barrier for insiders |
| T-C7-E1 | Role Elevation via Document | Document Store | Breaks the entire access control model if the LLM is used for any access decisions |

---

## Mitigation Type Distribution

| Type | Count |
|---|---|
| Preventive only | 28 |
| Detective only | 9 |
| Preventive + Detective | 9 |
| Corrective | 1 |
| Preventive + Corrective | 1 |

> **Observation:** The matrix is prevention-heavy, which is appropriate for a security-by-design
> analysis. The validation section (Tabletop Exercise) will stress-test whether detective controls
> surface attacks that bypass preventive ones.

---

## MITRE ATLAS Technique Reference

| Technique ID | Name | Threats That Reference It |
|---|---|---|
| AML.T0051.000 | LLM Prompt Injection (Direct) | T-C6-T1 |
| AML.T0051.001 | LLM Prompt Injection (Indirect) | T-C3-T1, T-C3-E1, T-C5-E1, T-C7-E1 |
| AML.T0020 | Poison Training/Knowledge Data | T-C5-S1, T-C5-T1, T-C7-T1, T-C8-T1 |
| AML.T0035 | ML Artifact Collection | T-C3-I1, T-C4-I1, T-C4-E1, T-C5-I1, T-C6-I1, T-C8-I1 |
| AML.T0043 | Craft Adversarial Data | T-C4-T1 |
| AML.T0054 | LLM Jailbreak | T-C6-E1 |

