# Business Requirements Document (BRD): PlayRecord Automation Flow Platform

## 1. Executive Summary
This project develops PlayRecord, a browser workflow automation platform that enables users to record a web-based business process once, convert it into a reusable parameterized flow, and replay it safely with new inputs on demand. The platform combines Playwright Codegen-based recording, AI-assisted parameter suggestion, structured flow modeling, and controlled replay to reduce repetitive manual work for internal business operations teams and technically inclined users. [file:1][file:12]

The MVP is designed to be practical, cost-conscious, and achievable using free-tier and open-source technologies. The system will prioritize reliable recording, guided review, secure parameter handling, and observable execution over advanced enterprise capabilities such as multi-tenant access control, scheduler orchestration, or self-healing selectors. [file:1][file:12]

## 2. The Business Problem & Opportunity
Teams repeatedly perform browser-based workflows such as timesheet entry, portal updates, repetitive form submissions, and internal system operations that are rule-based but still executed manually. This creates avoidable effort, introduces inconsistencies, and makes business continuity dependent on individual operator memory. [file:1]

Existing automation options often fail in one of two ways:
- Traditional RPA-style tools can be expensive, opaque, or overbuilt for small teams and MVP-stage internal products. [file:1]
- Raw Playwright scripting is powerful, but requires engineering skill, code editing, and repeat maintenance that non-developer users cannot easily manage. [file:1]
- AI-only automation without review creates a trust problem because users need visibility into what was captured, which values are dynamic, and what will run at execution time. [file:1][file:12]

The opportunity is to create a “record once, review safely, replay with new data” platform that sits between low-level scripting and heavyweight RPA suites. The product can become useful quickly for internal workflows while remaining extendable toward broader automation use cases later. [file:1]

## 3. Proposed Solution
PlayRecord will provide a guided flow lifecycle built around four stages: capture, review, parameterize, and replay. [file:1]

- Capture: A user creates a new flow from the UI by entering a flow name and target URL, then starts a Playwright Codegen recording session or uses a guided fallback import path for the generated TypeScript script. [file:1]
- Review: The platform parses the recorded script and presents a structured step-by-step review interface with optional raw TypeScript visibility for advanced users. [file:1]
- Parameterize: The system suggests dynamic and sensitive fields using AI assistance, while the user confirms, edits, or adds placeholders before the flow becomes reusable. [file:1]
- Replay: The saved flow is triggered from a flow library, prompts for required runtime values, resolves placeholders into a temporary Playwright script, executes the automation, and streams logs in real time. [file:1]

This approach retains the document’s strong execution architecture while improving the recording and review experience so the system is both implementable and likable for users. [file:1]

## 4. Core Features & Differentiators
- Guided recording workflow: Users create flows from a web UI instead of starting from code or standalone scripts. [file:1]
- Structured review before activation: Recorded steps are reviewed and corrected before being marked runnable, reducing blind trust in automation. [file:1]
- AI-assisted placeholder suggestion: The system identifies likely dynamic and sensitive values, but the final confirmation remains user-controlled. [file:1]
- Dual artifact model: The system stores a canonical structured flow definition and a parameterized execution-ready TypeScript template, improving both usability and runtime fidelity. [file:1]
- Secure runtime injection: Sensitive values are collected only at run time, excluded from persistence, and sanitized from logs. [file:1]
- Replay observability: Live run console, status updates, and retained execution history improve operational trust and debuggability. [file:1]
- Low-cost MVP architecture: The product is intentionally designed around free-tier or open-source technologies and local/self-hostable services. [file:1][file:12]

## 5. Technology Stack & Infrastructure
To ensure a scalable, cost-effective, and maintainable MVP, the platform architecture will use the following stack: [file:1][file:12]

- UI (Frontend) Layer: Angular application for flow creation, review, schema management, flow library, run console, and history views. [file:1]
- API (Backend) Layer: Python FastAPI service for flow lifecycle APIs, run orchestration, AI integration, validation, and WebSocket log streaming. [file:1]
- Automation Layer: Playwright with TypeScript for code generation and runtime browser automation. [file:1]
- Data Layer: SQLite for MVP persistence of flows, steps, schemas, run jobs, and logs. [file:1]
- AI Layer: Ollama as the primary open-source local inference option, with Groq as an optional free-tier acceleration fallback where available. [file:1]
- Realtime Layer: FastAPI WebSocket endpoints for run log streaming. [file:1]
- Hosting & Infrastructure: Local/self-hosted development environments, GitHub-based workflows, Docker-compatible deployment, and only free/open-source-friendly environments for MVP. Paid managed cloud services are out of scope for this phase. [file:1][file:12]
- Auxiliary Tooling: Open-source utilities for TypeScript parsing, script templating, background execution, and log sanitization. [file:1]

## 6. Project Scope (Phase 1)
### In-Scope
- Single-user or same-operator MVP usage where Flow Author and Flow Runner are effectively the same person. [file:1]
- New flow creation from UI with flow name, target URL, and recording initiation or import fallback. [file:1]
- Storage of raw recorded Playwright TypeScript scripts. [file:1]
- Script parsing into structured executable steps. [file:1]
- AI-assisted suggestion of dynamic and sensitive parameters. [file:1]
- User review and correction of steps, selectors, placeholder keys, sensitivity flags, and parameter definitions. [file:1]
- Saving reusable flows as structured JSON plus execution template script. [file:1]
- Manual flow triggering via flow library or flow detail page. [file:1]
- Pre-run collection of runtime values for credentials and dynamic parameters. [file:1]
- Playwright execution through backend-generated temporary scripts. [file:1]
- Real-time run console with logs and final run status. [file:1]
- Run history with log replay and rerun support using non-sensitive historical parameters. [file:1]

### Out-of-Scope
- Multi-user account administration, RBAC, and workspace sharing. [file:1]
- Scheduled or cron-based automation. [file:1]
- Selector drift detection and self-healing. [file:1]
- CAPTCHA solving, MFA bypass, or anti-bot evasion. [file:1]
- Mobile browser automation. [file:1]
- Fully autonomous AI-driven flow finalization without user review. [file:1]
- Paid cloud-only architecture or proprietary runtime dependencies. [file:12]

## 7. Non-Functional Requirements (NFRs)
- Security: Sensitive parameters must never be persisted to the database and must be removed or masked from any saved or streamed log content. [file:1]
- Reliability: All Playwright executions must run with cleanup protections and timeout handling so browser resources and temporary files are always released. [file:1]
- Performance: Flow parameterization for scripts up to 100 steps should complete within 30 seconds under normal MVP conditions, and run log latency should remain near real time for active sessions. [file:1]
- Usability: The product must separate credentials from non-sensitive parameters in the pre-run experience and make review of recorded steps understandable without requiring code knowledge. [file:1]
- Maintainability: The solution must remain workable using free/open-source tooling, simple local deployment, and modular service boundaries suitable for later scaling. [file:1][file:12]
- Auditability: Run statuses, non-sensitive runtime parameters, step outcomes, and log events must be traceable for debugging and support. [file:1]

## 8. High-Level Success Criteria
- A user can successfully record or import a Playwright Codegen flow and save it as a reusable flow definition. [file:1]
- A user can review and correct steps, selectors, and placeholders before activating the flow. [file:1]
- The system can prompt for runtime values and execute the same flow repeatedly with different input data. [file:1]
- No credential values are stored in the database or exposed in logs. [file:1]
- Run failures provide enough visibility for the user to identify the failed step and reason about remediation. [file:1]
- The MVP is deployable and operable using free-tier or open-source-friendly infrastructure and tooling. [file:1][file:12]
