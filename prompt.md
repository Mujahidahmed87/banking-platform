MASTER PROMPT FOR LLM — HIGH-LEVEL SOLUTION DESIGN (BANKING PLATFORM ON AWS)

Role and Output
- Role: Act as a senior cloud/solution architect with deep experience in regulated financial services in the UAE/GCC, AWS Well-Architected, and secure payments integrations.
- Output: Produce an 8–12 page proposal (suitable for PDF) plus diagram-as-code snippets and an optional Excel-ready cost matrix.
- Voice: Executive-technical hybrid, targeted to senior engineering and compliance-aware stakeholders.
- Structure and clarity: Use clear headings, concise paragraphs, and limited bullet points. All assumptions must be explicit and parameterized.

Business and Regulatory Context
- Geography: Primary UAE; future expansion to GCC. Architect for UAE data residency with GCC expansion-readiness.
- Regulations and standards: Adhere to UAE Central Bank standards, PCI DSS scope for cardholder data where applicable, GDPR considerations for any EU data subjects, and local data residency. Explicitly map architectural controls to each framework section-by-section in the document (e.g., which components/controls satisfy PCI DSS requirements for transmission/storage of PAN; which controls align to GDPR data subject rights; how UAE Central Bank expectations for outsourcing, resilience, and cybersecurity are addressed).
- Certifications: No mandated ISO 27001/SOC 2/NIST CSF, but align to AWS Well-Architected, NIST CSF-aligned practices, and PCI controls where relevant.
- Product scope: Retail banking only (Phase 1).
- Channels: Web banking and native mobile (iOS/Android).
- Localization: Website to support all languages; assume Arabic/English with RTL support as baseline; parameterize others.
- Launch/scale targets: No dates or volumes provided—design for scale with conservative, cost-aware elasticity. Parameterize MAU/DAU and peak TPS; provide default ranges and clearly flag as assumptions.
- Operating model: In-house operations (NOC/SOC). High availability, regional resiliency, minimal RTO/RPO.

Functional Scope (Design for breadth; phase where prudent)
1) Account Management
- Capabilities: Balances, transaction history, statements (PDF/e-statement), preferences (language, alerts, limits, biometrics, consent, marketing opt-in).
- Data domains: Accounts, cards, loans, deposits, investments.
- Core banking: Core not finalized—abstract behind an anti-corruption layer and API façade to allow Temenos/Finacle/Flexcube/in-house interchangeability.

2) Payments & Transfers
- Capabilities: Domestic (UAE), international (SWIFT), beneficiary management, limits, approvals, one-off/scheduled/standing orders/bulk. Design for ISO 20022 readiness; support MT where necessary.
- Risk and controls: Maker-checker, role-based approvals, transaction risk scoring; limits per txn/daily/tiered by segment/KYC.
- FX/fees: Support real-time quotes, fee/tolerance; parameterize integration to FX provider.
- Rail options: UAEFTS/UAESWITCH/IPN/Instant Payments unspecified—design rail abstraction to swap providers; recommend patterns for both host-to-host and via service bureau.

3) Notification System
- Channels: Push, SMS, email; extensible to WhatsApp. Low latency objective with cost-awareness. Preference management, event-driven triggers, templates, localization, throttling, quiet hours, and A/B testing capabilities.

4) Utilities Payments
- Biller aggregation, linking, presentment, payment; real-time vs. cached presentment; reconciliation, settlement files, exception handling; surcharges/fees and invoice archiving. UAE aggregators not fixed—design pluggable integration strategy.

Security, Risk, and Compliance (Security is paramount)
- IAM: Staff SSO with enterprise IdP (parameterize Okta/Azure AD). Customer auth via CIAM (e.g., Amazon Cognito) with policy-driven MFA (OTP SMS/email, TOTP, passkeys/WebAuthn) and device binding. Session management with inactivity timeouts, trusted devices, jailbreak/root detection.
- Data protection: AWS KMS (cloud-managed), key hierarchy, envelope encryption, automatic rotation, HSM-backed CMKs for high-sensitivity workloads. Data classification, masking, and pseudonymization for non-prod. Data residency in nearest AWS region to UAE (parameterize exact region).
- Fraud/AML: Integrate with transaction monitoring (parameterize Actimize/SAS/in-house), sanctions/PEP screening (e.g., Firco/SafeWatch), device fingerprinting and behavioral biometrics (pluggable). Place these within synchronous vs. asynchronous paths with circuit breakers for availability.
- Audit & logging: Retention policies, immutability/WORM, chain-of-custody, regulator read-only access, comprehensive audit trails for customer actions and staff overrides; clock synchronization and signed logs.

Architecture Expectations
- Architectural style: Prefer microservices for regulated domain separation and scalability; allow a modular monolith option if justified for time-to-market and team maturity. Provide a decision rationale.
- Service boundaries: Define domain-driven boundaries for customer, accounts, statements, payments, beneficiaries, notifications, utilities billing, limits/approvals, risk & compliance, reporting/analytics, and integration adapters for core/rails/aggregators.
- Data flow: Event-driven architecture for state changes and notifications; synchronous APIs for UX-critical flows; CDC where available from core banking.
- Protocols: REST and gRPC; message-driven with FIFO/SQS, EventBridge, and where needed MSK/Kafka for high-throughput streams; SFTP/ISO 20022 where unavoidable; ISO 8583 only if card rails require it.
- API strategy: Public, partner, and private APIs with API Gateway; define rate limits, throttling, monetization readiness, and developer portal approach.

AWS Reference Design (be explicit and justify)
- Compute: Compare AWS Lambda (serverless), ECS Fargate, and EKS (Kubernetes) for core services. Default: mission-critical transactional services on EKS for control/throughput; event handlers and glue on Lambda; batch on ECS Fargate where cost-effective.
- Data stores: Compare Aurora PostgreSQL vs. DynamoDB for each domain; default: ACID/relational (Aurora) for core transactional data; DynamoDB for high-scale, low-latency profile/caching/ledger adjuncts; OpenSearch for search; ElastiCache (Redis) for sessions/rate limits; Amazon S3 + Glacier for statements, reconciliations, archival with S3 Object Lock for WORM.
- Integration: EventBridge for domain events; SQS FIFO for ordered processing (payments); SNS for fanout; MSK only where strict streaming or exactly-once semantics justify the cost/ops overhead.
- Networking and security: Multi-AZ VPC design; AWS WAF + CloudFront; ALB/NLB patterns; PrivateLink for third parties; VPC endpoints; SGs/NACLs; Secrets Manager; SSM Parameter Store; IAM least-privilege with ABAC; KMS everywhere; CloudHSM only if required.
- CI/CD & IaC: Terraform for IaC; GitHub Actions for pipelines; OIDC to AWS; multi-account strategy (sandbox/dev/test/QA/UAT/perf/pre-prod/prod) with account vending and landing zone guardrails (SCPs, AWS Config, Security Hub).
- Observability: Prefer cost-aware stack—CloudWatch + OpenTelemetry; compare Datadog and Splunk options with cost/benefit. Structured logging, trace propagation, metrics SLOs, synthetic monitoring, runbooks, and chaos testing posture.
- DR & availability: Active-active multi-AZ; cross-region DR with RTO/RPO per domain (parameterize minimal targets). Warm-standby or pilot light based on cost vs. RTO. Data backups, PITR, cross-region replication (where residency allows).
- Mobility/web: Native iOS/Android; enforce certificate pinning, device attestation, secure keychain/Keystore storage, runtime protections; for web, SPA with SSR where helpful, CDN offload, strict CSP, and WAF.

Non-Functional Requirements
- Availability targets: Banking-grade HA across critical paths; parameterize SLOs and error budgets.
- Performance targets: Provide recommended P99 targets per capability (login, balance, payment initiation, notification) and justify.
- Scalability: Horizontal pod autoscaling, KEDA for event workloads; read replicas; caching strategy; backpressure and queue depth controls.
- Cost management (FinOps): Tagging, per-domain cost allocation, unit economics; autoscaling policies; right-sizing; Savings Plans/RI strategy; demand shaping (queues); serverless where justified; disruption testing aligned with budget.

DevOps, SDLC, Quality
- Pipelines: Trunk-based development with PR checks; stages with promotion policies and release governance.
- Security gates: SAST, DAST, SCA, IaC scanning, SBOM, signed artifacts (SLSA), secret scanning; break-glass controls.
- Release strategies: Blue/green, canary, and feature flags; dark launches; progressive delivery; mobile phased rollout (no constraints assumed).
- Testing: Contract, performance, chaos, resilience, and E2E synthetic tests; test data strategy with anonymization and synthetic data policy.
- Data lifecycle: Retention schedules per domain; archival to Glacier; privacy workflows for GDPR-like rights.

Buy vs. Build
- Default posture: Build UX and domain logic; buy/integrate for SWIFT gateway/service bureau, biller aggregators, notifications (SMS/email/push providers), fraud/AML, and possibly device fingerprinting. Provide justification: time-to-market, compliance overhead, and long-term TCO.
- Provide a decision log and vendor criteria; leave placeholders for final vendor selection.

Deliverables
- Executive summary (1 page).
- Architecture diagrams:
  - C4 Context, Container, and Component views.
  - AWS deployment diagram (networking, security zones, services).
  - 2–3 sequence diagrams (login, domestic payment, notification).
- Comparative cost analysis:
  - EKS vs. ECS vs. Lambda for core workloads.
  - Aurora vs. DynamoDB per domain.
  - MSK vs. EventBridge/SNS/SQS.
  - API Gateway vs. ALB/NLB for north-south traffic.
  - Pinpoint vs. third-party (Twilio/Infobip/email providers).
  - Kinesis vs. MSK (if analytics streaming arises).
- Buy vs. build recommendations table with rationale.
- Risks and trade-offs.
- Bill of materials.
- Optional: Excel-ready cost input matrix and sample workload calculator.

Formatting Requirements
- Keep within 8–12 pages excluding appendix.
- Use clear headings and short paragraphs for readability.
- Include diagram-as-code (Mermaid or PlantUML) for:
  - C4 Context/Container/Component.
  - AWS Deployment (at least one).
  - Two Sequence diagrams.
- Include a compact cost table for each comparison and a one-paragraph narrative conclusion per table.
- Clearly mark all assumptions and provide a parameter block up front to enable easy substitution.

Assumptions and Parameter Block (fill defaults but mark “ASSUMED”)
- Region: me-central-1 (ASSUMED; adjust to nearest UAE-compliant region).
- MAU/DAU: MAU=500,000; DAU=150,000 (ASSUMED).
- Peak TPS: Login 600 TPS; Balance 400 TPS; Payments 150 TPS; Notifications fanout 1,000 msg/s (ASSUMED).
- RTO/RPO targets:
  - Auth: RTO 15min, RPO 0–5min (ASSUMED).
  - Payments: RTO 15min, RPO 0–5min (ASSUMED).
  - Notifications: RTO 60min, RPO 15min (ASSUMED).
  - Utilities: RTO 60min, RPO 15min (ASSUMED).
- Data retention: Transactional 7 years; logs 1 year hot + 6 years archive (ASSUMED; align to UAE Central Bank guidance).
- Channels: iOS/Android native; Web SPA+SSR (ASSUMED).
- CIAM: Amazon Cognito (ASSUMED).
- Observability: CloudWatch + OpenTelemetry; optional Datadog/Splunk if budget permits (ASSUMED).

What to Produce

1) Executive Summary
- One page synthesizing business context, regulatory alignment, architecture style, availability/DR posture, security anchors, and cost-aware choices.

2) C4 and AWS Deployment (Diagram-as-Code)
- Provide Mermaid or PlantUML code for:
  - C4 Context: Users, mobile/web apps, core banking, payment rails, notification providers, utilities aggregators, fraud/AML, reporting.
  - C4 Container: Microservices (account, payments, beneficiaries, notifications, utilities, limits/approvals, risk & compliance), data stores, message buses, API gateway, CIAM, observability, and shared services.
  - C4 Component: Payments service internals (API layer, validation, risk checks, limits service, ledger writes, rail adapters).
  - Sequence diagrams: 
    - Customer login with CIAM, device binding, OTP/passkeys, token issuance.
    - Domestic payment with maker-checker, sanctions check, risk scoring, rail submission, notification.
  - AWS Deployment: VPC, subnets (public/private), ALB/NLB, EKS/ECS/Lambda, RDS/Aurora, DynamoDB, ElastiCache, S3+Glacier, EventBridge/SNS/SQS/MSK, CloudFront+WAF, NAT/IGW, KMS/Secrets Manager, IAM roles, CloudWatch.

3) Architecture Rationale
- Microservices vs. modular monolith: decision matrix and recommended path; risks and team maturity considerations.
- Data topology: Aurora vs. DynamoDB per domain; multi-tenant vs. per-service DB; CDC from core banking if available.
- Eventing: When to use SQS FIFO vs. EventBridge vs. MSK; idempotency, exactly-once semantics strategies, and dead-letter handling.
- API ingress: API Gateway vs. ALB/NLB selection per interface; private vs. public endpoints and WAF protection.
- Mobile/web security: Certificate pinning, runtime attestation, secure storage, CSP, anti-automation controls, bot mitigation.

4) Security & Compliance Mapping
- For each regulatory framework (UAE Central Bank, PCI DSS, GDPR), map:
  - Identity & Access controls (CIAM, IAM, SSO, MFA, RBAC/ABAC).
  - Data protection (encryption at rest/in transit, KMS, key rotation).
  - Logging & monitoring (immutability/WORM, retention).
  - Vendor oversight (outsourcing, data processing agreements).
  - Incident response and breach notification workflows.
- PCI scope containment techniques and tokenization guidance.

5) Deployment, HA, DR, and Operations
- Multi-AZ active-active; cross-region DR options: warm standby vs. pilot light; replication strategies; runbooks; chaos game days cadence.
- SRE SLOs and error budgets; alerting policies; on-call model; capacity management and autoscaling runbooks.

6) DevOps, SDLC, Quality Gates
- Terraform mono-repo/multi-repo approach; GitHub Actions workflows (build/test/scan/deploy); OIDC to AWS.
- Security gates: SAST/DAST/SCA/IaC scanning, SBOM, signed artifacts; secrets in Secrets Manager; policy-as-code with OPA/Conftest; supply chain hardening (SLSA level targets).
- Release strategies: Feature flags, blue/green, canary; mobile phased rollout guidance.

7) Cost-Aware Design and Comparative Tables
- Provide comparison tables with qualitative and quantitative cost drivers and operational trade-offs:
  - EKS vs. ECS Fargate vs. Lambda for core services.
  - Aurora PostgreSQL vs. DynamoDB per domain.
  - MSK vs. EventBridge/SNS/SQS for events/streams.
  - API Gateway vs. ALB/NLB for APIs.
  - Pinpoint vs. third-party SMS/email/push providers.
  - Kinesis vs. MSK for analytics streaming (if applicable).
- Include both monthly modeled workload estimates and unit economics (per request/message/GB metrics). Show savings levers (autoscaling, off-peak scale-down, Spot where permissible, Savings Plans/RIs).

8) Buy vs. Build Recommendations
- Table listing component, recommendation (buy/build), rationale (time-to-market, compliance, TCO), and vendor archetypes. Include:
  - SWIFT gateway/service bureau: buy.
  - Biller aggregator: buy.
  - Notification providers (SMS/email/push): buy, abstract behind internal interface; Pinpoint as baseline, allow vendor swap.
  - Fraud/AML/sanctions: buy or integrate; design risk-engine adapter.
  - Device fingerprinting/behavioral biometrics: buy if risk appetite demands; optional in MVP.
  - UX/domain services: build.
- Add decision log with assumptions and revisit triggers.

9) Risks, Trade-offs, and Mitigations
- Unknowns (rails, volumes, vendors) and how abstraction/interfaces mitigate lock-in.
- Team readiness for EKS ops; fallback to ECS/Lambda.
- Residency and cross-region replication constraints.
- PCI scope creep; observability cost sprawl; rate limit/abuse risks.

10) Bill of Materials
- Enumerate AWS services, third-party components, and licenses; note BYOL considerations; list per-environment footprints.

11) Optional Excel-Ready Cost Matrix
- Provide a CSV-style matrix with rows for services (compute, storage, data transfer, messaging, observability, CIAM, notifications) and columns for assumptions (requests/GB/hours), unit prices (parameterize), monthly totals, and notes.

Authoring Notes
- Where exact values are unknown, present defaults labeled ASSUMED and place all such values in the parameter block.
- Use concise, unambiguous language.
- Prefer industry best practices that balance security, availability, and cost.
- Ensure the document is self-contained and understandable without external references.

Now generate the full solution proposal per the above, including:
- The complete write-up (8–12 pages worth of content).
- Diagram-as-code blocks (Mermaid or PlantUML) for C4 and AWS deployment, and 2 sequence diagrams.
- Comparative cost tables with qualitative trade-offs and parameterized numeric examples.
- Buy vs. build table, risks/trade-offs, BOM, and optional Excel-ready cost matrix.
- An explicit mapping table that shows which architecture component/control satisfies which regulation (UAE Central Bank, PCI DSS, GDPR), to enhance clarity for the reader.