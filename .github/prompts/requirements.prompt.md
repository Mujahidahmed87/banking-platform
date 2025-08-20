# Assignment Brief: High-Level Solution Design — Banking Platform on AWS

## Objective

Design a scalable, secure, and cost-effective architecture for a web and mobile banking platform hosted on AWS, covering business capabilities, core system architecture, deployment, security, DevOps, and build vs. buy decisions.

### Scope of Capabilities
Focus on four core functional areas:
1. Account Management
- View balances, transaction history, statements
- Preferences and alerts
2. Payments and Transfers
- Domestic (UAE) and international (SWIFT)
- Beneficiary setup, limits, and approvals
3. Notification System
- Push, SMS, email
- Preference management and event-driven triggers
4. Utilities Payments
- Biller aggregation, linking, presentment, payment. Include non-functional requirements: security, availability, scalability, observability, and cost management.

### Architecture Expectations
Propose a high-level AWS-based solution addressing:
- System/Application Architecture Microservices vs. modular monolith, data flow, service boundaries
- Security & Compliance Identity and access, encryption, secure integrations (SWIFT, utilities)
- Deployment & Availability Multi-AZ setup, DR strategy, CI/CD pipelines, monitoring
- DevOps & SDLC IaC, release strategy, environment management
- Cost-Aware Design Compare key AWS services by operational cost and trade-offs (e g., EKS vs. ECS vs. Lambda; Aurora vs. DynamoDB)
- Buy vs Build Analysis

### Indicate which components should be:
- Built in-house: where customization/control is needed
- Bought/integrated: e.g., SWIFT gateway, biller aggregators, notification APIs Justify each choice based on time-to-market, compliance, and cost.

### Deliverables
Submit a brief proposal (max 8–12 pages) that includes:
- Executive summary of solution
- Key architecture diagrams (Context/Container/Component, service layers, AWS deployment)
- Comparative cost analysis tables for core AWS options
- Buy vs. build recommendations
- Risks and key trade-offs
- Format: PDF + architecture diagrams Optional: Excel sheet for cost comparisons
Expectation
- The content should be clear in communicating the intent of what is being depicted.