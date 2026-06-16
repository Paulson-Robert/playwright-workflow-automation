# Product Requirements Document (PRD): PlayRecord Automation Flow Platform

## 1. Product Overview
PlayRecord is an MVP platform for recording browser workflows, converting them into reusable parameterized automation flows, and replaying them safely with new input values. The product targets repetitive browser-based internal processes where users need the power of Playwright without depending on hand-authored scripts for every run. [file:1]

The product promise is: record a business workflow once, review it with confidence, and replay it reliably with different data. The MVP will optimize for trust, guided usability, and low-cost implementation rather than advanced enterprise orchestration. [file:1]

## 2. Product Goals
- Reduce repeated manual execution of browser workflows. [file:1]
- Convert one-time recorded actions into reusable flow assets. [file:1]
- Give users control over dynamic and sensitive values before a flow is saved. [file:1]
- Provide a safe replay model with runtime value injection, live logs, and auditable run history. [file:1]
- Deliver the MVP entirely with free-tier or open-source-compatible technologies. [file:1][file:12]

## 3. Target Users
- Flow Author: Records workflows, reviews extracted steps, confirms placeholders, and defines parameter schema. [file:1]
- Flow Runner: Chooses a ready flow, enters runtime values, triggers execution, and monitors logs. In MVP, this is usually the same person as the Flow Author. [file:1]
- Technical Admin/Builder: Maintains local deployment, AI dependencies, and Playwright runtime configuration in development or internal environments. [file:1]

## 4. User Problems
- Repetitive portal tasks consume time and attention even when the steps are deterministic. [file:1]
- Writing or maintaining Playwright scripts manually is too technical for many potential operators. [file:1]
- Users do not trust fully automatic AI parameterization unless they can inspect and correct what was captured. [file:1]
- Rerunning flows with new credentials, dates, or business inputs is cumbersome without a structured runtime input layer. [file:1]
- Debugging failed browser automation is difficult without execution logs and step-level context. [file:1]

## 5. Product Principles
- Review before replay: no flow becomes runnable without user-visible review. [file:1]
- AI assists, user decides: AI suggests placeholders and sensitivity, but users confirm them. [file:1]
- JSON for product logic, TypeScript for execution: the platform uses structured flow data for usability and a template script for runtime fidelity. [file:1]
- Sensitive data is ephemeral: credentials are runtime-only and excluded from persistence. [file:1]
- MVP over ambition: the first version solves recording, review, parameterization, trigger, and observation well before broader automation features are added. [file:1]

## 6. Functional Requirements
### 6.1 Flow Creation
- The UI must allow creation of a new flow with name, target URL, and optional description. [file:1]
- The system must support a primary recording path and a fallback import path for raw Playwright TypeScript scripts. [file:1]
- The raw recorded script must be stored against the flow draft. [file:1]

### 6.2 Recording and Ingestion
- The system must support Playwright Codegen-based recording for browser actions. [file:1]
- The system must validate that the uploaded or captured script is valid TypeScript or at minimum parseable into supported steps. [file:1]
- The system must reject empty or non-actionable scripts with an understandable validation message. [file:1]

### 6.3 Review and Parameter Suggestion
- The system must parse the raw script into ordered steps. [file:1]
- The system must generate AI-assisted suggestions for dynamic values, sensitive fields, and placeholder names. [file:1]
- The system must show a structured review UI containing step number, action type, selector hint, and either static value or placeholder metadata. [file:1]
- The system must allow users to edit, delete, insert, and reorder steps before finalizing the flow. [file:1]
- The system must allow advanced raw script viewing and limited editing for technical users. [file:1]

### 6.4 Parameter Schema Management
- The system must generate a parameter schema from confirmed placeholders. [file:1]
- The schema must distinguish credentials from non-sensitive run parameters. [file:1]
- The user must be able to edit display labels, input types, required flags, and default values for non-sensitive parameters. [file:1]
- The system must prevent invalid schema changes that break referenced step parameters. [file:1]

### 6.5 Flow Storage
- The platform must persist the raw recorded script, parameterized template script, structured flow JSON, and parameter schema. [file:1]
- The flow must transition through draft, review-ready, ready, and failure states as needed. [file:1]

### 6.6 Manual Trigger and Replay
- The user must be able to trigger a ready flow from the flow library or flow detail screen. [file:1]
- The pre-run dialog must render runtime fields from the stored parameter schema. [file:1]
- The system must validate required inputs before triggering execution. [file:1]
- The system must create a RunJob and start execution asynchronously. [file:1]

### 6.7 Runtime Execution
- The backend must resolve placeholders into a per-run temporary script without modifying the canonical stored flow definition. [file:1]
- The system must execute the resolved Playwright script using the backend automation environment. [file:1]
- The system must stream stdout/stderr-derived logs to the client in near real time. [file:1]
- The system must mark each run as pending, running, success, or failed. [file:1]

### 6.8 Run Console and History
- The user must be able to watch an active run in a live console. [file:1]
- The system must retain completed run metadata and logs for later review. [file:1]
- The user must be able to rerun historical flows using previously saved non-sensitive values as defaults. [file:1]

### 6.9 Optional NLP Trigger
- The product may support natural-language-triggered run initiation where an AI service maps a command to a known flow and resolves some parameters. [file:1]
- If confidence is low or required values are missing, the system must fall back to explicit user confirmation and parameter entry. [file:1]
- This capability is valuable but may be deprioritized behind the core record-review-replay path if MVP scope needs tightening. [file:1]

## 7. Non-Functional Requirements
- The app must remain usable on standard desktop browsers in internal environments. [file:1]
- Logs must not expose secrets or sensitive values. [file:1]
- Long-running or hung executions must timeout and fail cleanly. [file:1]
- The frontend and backend should remain modular enough to swap SQLite for PostgreSQL and local AI inference for other providers in later phases. [file:1]
- The MVP stack must remain within free-tier/open-source constraints. [file:1][file:12]

## 8. MVP Screens / Modules
- Flow Library [file:1]
- New Flow / Start Recording [file:1]
- Recording Import / Capture Status [file:1]
- Flow Review Editor [file:1]
- Parameter Schema Editor [file:1]
- Flow Detail [file:1]
- Pre-Run Dialog [file:1]
- Run Console [file:1]
- Run History [file:1]

## 9. Acceptance Criteria
- A user can create a new flow and associate it with a target URL. [file:1]
- A raw Playwright script can be captured or imported and validated successfully. [file:1]
- The system can parse the script into steps and suggest placeholders for review. [file:1]
- The user can modify steps and schema before saving the flow. [file:1]
- A saved flow can be executed repeatedly with different runtime inputs. [file:1]
- Sensitive values are never written to persistent storage or visible in logs. [file:1]
- Failed runs clearly indicate the failure point and preserve diagnostic logs. [file:1]
- The MVP can run using Angular, Python, Playwright, SQLite, and open-source/free-tier-friendly supporting tools. [file:1]

## 10. Future Enhancements
- Multi-user workspaces and RBAC. [file:1]
- Scheduled flow runs and queues. [file:1]
- Selector self-healing. [file:1]
- Advanced branching and reusable subflows. [file:1]
- Hosted deployment recipes and enterprise-grade persistence. [file:1]
