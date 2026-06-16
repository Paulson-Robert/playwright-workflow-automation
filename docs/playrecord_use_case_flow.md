# Use Case Specification: PlayRecord Automation Flow Platform

**Document Type:** Use Case Specification  
**Version:** 2.0  
**Mode:** Record → Review → Parameterize → Replay  
**Stack:** Angular (Frontend) · Python FastAPI (Backend) · SQLite (DB) · Ollama/Groq (AI) · Playwright TypeScript (Automation)  
**Phase:** MVP Phase [file:1]

## 1. Executive Summary
PlayRecord is a browser automation platform that converts a recorded Playwright browser session into a reusable, reviewable, parameterized flow. A user records or imports a browser workflow, reviews the extracted steps, confirms dynamic and sensitive placeholders, saves a reusable flow definition, and later replays that flow with fresh runtime values through a controlled execution pipeline. [file:1]

The refined MVP flow emphasizes practical implementation and user trust. Instead of forcing raw script editing for everyone or relying on fully automatic AI conversion, the platform combines structured review, user confirmation, runtime-only secret handling, and live execution visibility. [file:1]

## 2. System Actors
| Actor | Role | Interaction Mode |
|---|---|---|
| Flow Author | Creates flows, records/imports scripts, reviews steps, confirms parameters | Web UI (Angular) [file:1] |
| Flow Runner | Triggers saved flows and monitors execution | Web UI (Angular) [file:1] |
| System (AI Layer) | Suggests placeholders, sensitivity, and optional NLP command parsing | Internal service via Ollama/Groq [file:1] |
| Automation Engine | Executes resolved Playwright scripts | Backend subprocess runner [file:1] |
| Target Application | Web application being automated | Playwright browser session [file:1] |

In MVP, Flow Author and Flow Runner are typically the same operator. [file:1]

## 3. System Boundaries
### In Scope (MVP)
- Flow creation with target URL and guided recording/import. [file:1]
- Raw Playwright script capture and storage. [file:1]
- Parsing to structured steps. [file:1]
- AI-assisted placeholder suggestion. [file:1]
- User review and correction of recorded steps. [file:1]
- Parameter schema generation and editing. [file:1]
- Manual replay with runtime input collection. [file:1]
- Playwright execution through temporary resolved scripts. [file:1]
- Real-time log streaming and run history. [file:1]

### Out of Scope (MVP)
- Scheduled runs. [file:1]
- Multi-user collaboration and access control. [file:1]
- Selector drift auto-healing. [file:1]
- CAPTCHA/MFA bypass. [file:1]
- Mobile automation. [file:1]

## 4. Core Use Cases
### UC-01: Create and Record a New Flow
**Actor:** Flow Author  
**Trigger:** User wants to automate a repetitive browser workflow  
**Preconditions:**
- Target application is reachable. [file:1]
- Playwright recording path or import fallback is available. [file:1]

**Main Flow:**
1. User navigates to PlayRecord and clicks **New Flow**. [file:1]
2. User enters flow name, target URL, and optional description. [file:1]
3. User clicks **Start Recording**. [file:1]
4. System starts a guided recording session or presents a fallback import command/path if direct launch is unavailable in the MVP environment. [file:1]
5. User performs the target process in the recording browser. [file:1]
6. System captures or receives the generated raw TypeScript Playwright script. [file:1]
7. System saves the flow in draft state and starts parsing/analysis. [file:1]

**Postconditions:**
- Raw script is stored against the flow. [file:1]
- Flow enters review-ready analysis stage. [file:1]

**Alternative Flows:**
- A1: User pastes or uploads a generated `.ts` script instead of using the primary recording path. [file:1]
- A2: User abandons recording; system saves draft metadata without a runnable flow. [file:1]

**Edge Cases:**
- E1: Script is invalid or unparsable; system blocks progression and highlights the problem. [file:1]
- E2: Script contains no meaningful actionable steps; system warns before allowing further processing. [file:1]

### UC-02: Analyze Script and Suggest Parameters
**Actor:** System (AI Layer)  
**Trigger:** Raw script is stored for a draft flow  
**Preconditions:**
- Valid or partially parseable TypeScript script exists. [file:1]
- AI analysis service is available, or fallback heuristic parsing is enabled. [file:1]

**Main Flow:**
1. Backend parses the raw TypeScript into ordered browser actions. [file:1]
2. System identifies candidate dynamic values, credentials, and static actions. [file:1]
3. AI layer suggests placeholder keys in consistent naming format. [file:1]
4. System prepares a structured step list, suggested parameter schema, and optional parameterized template script. [file:1]
5. Flow status becomes review-ready. [file:1]

**Postconditions:**
- Suggested steps and placeholders are ready for user review. [file:1]
- No flow is marked ready for execution without user confirmation. [file:1]

**Alternative Flows:**
- A1: AI service is unavailable; system falls back to deterministic parsing and marks suggestions as limited-confidence. [file:1]
- A2: Parsing partially succeeds; unsupported steps are flagged for manual review. [file:1]

**Edge Cases:**
- E1: AI over-parameterizes fixed labels or selectors; suggestions are shown as editable, not final. [file:1]
- E2: AI misses dynamic values; user can manually mark them in UC-03. [file:1]

### UC-03: Review and Finalize Flow Steps
**Actor:** Flow Author  
**Trigger:** User opens a review-ready flow  
**Preconditions:**
- Structured steps and parameter suggestions exist. [file:1]

**Main Flow:**
1. User opens the flow from the Flow Library. [file:1]
2. System shows a step timeline with action type, selector hint, captured value, and placeholder/sensitivity suggestion. [file:1]
3. User reviews each step for correctness. [file:1]
4. User may delete, insert, reorder, or edit steps. [file:1]
5. User may accept or reject AI suggestions. [file:1]
6. User may mark additional values as dynamic or sensitive. [file:1]
7. User may view or edit the raw TypeScript in advanced mode if needed. [file:1]
8. User clicks **Save Changes**. [file:1]
9. System regenerates the parameter schema and execution template. [file:1]
10. Flow status becomes ready if validation passes. [file:1]

**Postconditions:**
- Flow contains validated steps and placeholders. [file:1]
- Flow is ready for runtime execution. [file:1]

**Edge Cases:**
- E1: User removes the only navigation step; system warns that flow startup may fail. [file:1]
- E2: User reorders steps into an illogical sequence; system warns but allows explicit confirmation. [file:1]
- E3: User creates duplicate placeholder keys with conflicting types; system blocks save until corrected. [file:1]

### UC-04: Define and Edit Parameter Schema
**Actor:** Flow Author  
**Trigger:** User opens the Parameters tab for a ready or review-ready flow  
**Preconditions:**
- Flow contains placeholder-backed steps. [file:1]

**Main Flow:**
1. User opens the Parameters tab. [file:1]
2. System shows generated schema grouped into **Credentials** and **Run Parameters**. [file:1]
3. User edits display label, type, required flag, and default value where allowed. [file:1]
4. User saves the schema. [file:1]
5. System validates references and persists the updated schema. [file:1]

**Postconditions:**
- Pre-run dialog rendering behavior is updated from schema configuration. [file:1]

**Edge Cases:**
- E1: User removes a parameter still referenced by steps; save is blocked. [file:1]
- E2: User changes field type to an incompatible value; system requires correction. [file:1]

### UC-05: Trigger a Flow Run (Manual)
**Actor:** Flow Runner  
**Trigger:** User clicks **Run Flow** from the library or detail page  
**Preconditions:**
- Flow is in ready state. [file:1]
- Parameter schema exists. [file:1]

**Main Flow:**
1. User clicks **Run Flow**. [file:1]
2. System opens the pre-run dialog. [file:1]
3. Dialog renders credential fields and run parameter fields from schema. [file:1]
4. User enters required values and confirms. [file:1]
5. System validates the submission. [file:1]
6. System creates a RunJob with non-sensitive parameters only. [file:1]
7. Frontend navigates to the Run Console. [file:1]
8. Backend starts Playwright execution asynchronously. [file:1]

**Postconditions:**
- RunJob is created. [file:1]
- Execution begins. [file:1]

**Alternative Flows:**
- A1: User closes the modal; no run is created. [file:1]

**Edge Cases:**
- E1: Required credentials are missing; submission is blocked. [file:1]
- E2: Input data type is invalid; field-level validation is shown. [file:1]

### UC-06: Execute Automation via Playwright
**Actor:** System (Automation Engine)  
**Trigger:** RunJob is created with pending status  
**Preconditions:**
- Playwright runtime is installed. [file:1]
- Parameterized template script exists. [file:1]

**Main Flow:**
1. Backend retrieves the saved template script and runtime parameter values. [file:1]
2. System resolves placeholders into a per-run temporary TypeScript script. [file:1]
3. Sensitive values remain in memory only and are not persisted. [file:1]
4. System writes the resolved file to a temporary runtime location. [file:1]
5. System updates the RunJob to running. [file:1]
6. System launches the Playwright script in a subprocess. [file:1]
7. stdout/stderr events are captured and sanitized. [file:1]
8. Logs are persisted and broadcast through WebSocket. [file:1]
9. On completion, job status becomes success or failed. [file:1]
10. Temporary files are deleted. [file:1]

**Postconditions:**
- Run status and logs are stored. [file:1]
- Runtime artifacts are cleaned up. [file:1]

**Edge Cases:**
- E1: Script hangs; timeout handler stops the process and marks the run failed. [file:1]
- E2: Selector cannot be found; error is logged and run fails. [file:1]
- E3: Credentials are incorrect; login step fails without logging the secret. [file:1]

### UC-07: Monitor Run in Real Time
**Actor:** Flow Runner  
**Trigger:** User opens the Run Console for an active run  
**Preconditions:**
- RunJob exists. [file:1]
- WebSocket service is available. [file:1]

**Main Flow:**
1. Page loads initial run metadata. [file:1]
2. Client connects to the run log WebSocket channel. [file:1]
3. Logs stream live into the console. [file:1]
4. Status indicator updates from pending to running to final state. [file:1]
5. Final banner shows success or failure outcome. [file:1]

**Postconditions:**
- User sees end-to-end execution visibility. [file:1]

### UC-08: Trigger a Flow via Natural Language Command
**Actor:** Flow Runner  
**Trigger:** User enters a command in a command interface  
**Preconditions:**
- At least one ready flow exists. [file:1]
- NLP service is available. [file:1]

**Main Flow:**
1. User enters a natural language command. [file:1]
2. System maps the command to a flow and resolves candidate parameters. [file:1]
3. If any required inputs are missing, the user is prompted only for those values. [file:1]
4. The system proceeds through the same run creation and execution path as UC-05 onward. [file:1]

**Postconditions:**
- Command-initiated run is recorded with traceable context. [file:1]

**Edge Cases:**
- E1: Confidence is low; user must select the intended flow manually. [file:1]
- E2: Command contains credentials; system must not store or trust them as final values. [file:1]

### UC-09: View Flow Run History
**Actor:** Flow Runner  
**Trigger:** User opens a flow’s history or the global runs list  
**Preconditions:**
- At least one RunJob exists. [file:1]

**Main Flow:**
1. User opens run history. [file:1]
2. System displays runs with status, time, trigger source, duration, and optional command text. [file:1]
3. User opens a completed run to inspect static logs. [file:1]
4. User may choose to rerun using historical non-sensitive parameters as defaults. [file:1]

**Postconditions:**
- Historical execution behavior is visible and reusable. [file:1]

### UC-10: Delete a Flow
**Actor:** Flow Author  
**Trigger:** User clicks **Delete Flow**  
**Preconditions:**
- Flow exists. [file:1]

**Main Flow:**
1. User initiates delete. [file:1]
2. System shows a confirmation message. [file:1]
3. User confirms. [file:1]
4. System soft-deletes the flow while preserving historical run records. [file:1]

**Postconditions:**
- Flow is removed from active listing. [file:1]
- Historical audit context remains available. [file:1]

## 5. Non-Functional Requirements
### 5.1 Performance
- Analysis and parameter suggestion should complete within practical MVP timing for scripts up to roughly 100 steps. [file:1]
- Real-time log propagation should feel immediate to the active user. [file:1]

### 5.2 Security
- Secrets must never be persisted. [file:1]
- Sensitive data must be sanitized from logs. [file:1]
- Temporary resolved scripts must be cleaned up after execution. [file:1]

### 5.3 Reliability
- Browser and subprocess cleanup must occur even on failure. [file:1]
- Stale running jobs must be recoverable or auto-failed after restart/timeout conditions. [file:1]

### 5.4 Usability
- Runtime input collection must clearly separate credentials from other parameters. [file:1]
- Failed runs must provide enough context to identify the failure point. [file:1]
- Review UI must remain usable even for users who do not understand raw Playwright code. [file:1]

## 6. Data Models (Summary)
| Table | Key Fields | Purpose |
|---|---|---|
| flows | id, name, url, status, raw_script, template_script, flow_json, param_schema | Stores reusable flow definition and source artifacts [file:1] |
| flow_steps | id, flow_id, order, action_type, selector_hint, value, param_key, is_sensitive | Ordered flow steps for editing and rendering [file:1] |
| run_jobs | id, flow_id, status, params, triggered_by, command_text, started_at, finished_at | Execution history record without stored secrets [file:1] |
| run_logs | id, job_id, level, message, timestamp | Per-line runtime log output [file:1] |

## 7. API Endpoints (Summary)
### Flows
| Method | Endpoint | Description |
|---|---|---|
| POST | /flows | Create new flow [file:1] |
| GET | /flows | List flows [file:1] |
| GET | /flows/{id} | Get flow detail [file:1] |
| PUT | /flows/{id} | Update flow metadata [file:1] |
| DELETE | /flows/{id} | Soft-delete flow [file:1] |
| POST | /flows/{id}/script | Upload or attach raw script [file:1] |
| POST | /flows/{id}/analyze | Generate structured steps and suggestions [file:1] |
| PUT | /flows/{id}/steps/{step_id} | Edit step [file:1] |
| PUT | /flows/{id}/schema | Update parameter schema [file:1] |

### Runs
| Method | Endpoint | Description |
|---|---|---|
| POST | /runs | Trigger flow run [file:1] |
| GET | /runs | List runs [file:1] |
| GET | /runs/{job_id} | Get run detail and logs [file:1] |
| POST | /runs/command | Trigger via natural language command [file:1] |

### WebSocket
| Endpoint | Description |
|---|---|
| WS /ws/runs/{job_id}/logs | Stream live run logs [file:1] |

## 8. MVP Acceptance Criteria
- A user can create a new flow and capture or import a recorded Playwright script. [file:1]
- The system can analyze the script and generate editable parameter suggestions. [file:1]
- A user can review and finalize steps before marking the flow ready. [file:1]
- A ready flow can be triggered with runtime parameters and credentials. [file:1]
- Execution uses a temporary resolved Playwright script and preserves no secrets. [file:1]
- Live logs and final run outcomes are visible to the user. [file:1]
- Historical runs can be reviewed after completion. [file:1]

## 9. Known Risks and Mitigations
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| AI misses or over-suggests placeholders | Medium | High | Mandatory user review before readiness [file:1] |
| Recorded selectors become unstable after target UI changes | High | High | User step editing now; self-healing deferred [file:1] |
| Runtime capture/launch differs across environments | Medium | Medium | Support guided fallback import path for MVP [file:1] |
| Free-tier AI inference is slow | Medium | Medium | Use Ollama primarily and optional Groq fallback [file:1] |
| Secrets leak into logs | Low | High | Central log sanitization and secret exclusion rules [file:1] |
| Temporary execution files remain after failures | Low | Medium | Enforce cleanup and startup recovery scans [file:1] |
