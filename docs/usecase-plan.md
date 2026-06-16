**Use Case Plan - PlayRecord (Video-to-Automation Platform)**

**Document Type:** Use Case Specification  
**Version:** 1.0  
**Mode:** Playwright Codegen → LLM Parameterization → Replay  
**Stack:** Angular (Frontend) · Python FastAPI (Backend) · SQLite (DB) · Ollama/Groq (AI) · Playwright TypeScript (Automation)  
**Phase:** MVP Phase

**1\. Executive Summary**

PlayRecord is a process automation platform that converts a live browser recording session into a reusable, parameterized automation flow. A user records a workflow once using Playwright Codegen, the system processes that recording into a structured JSON definition with dynamic parameter placeholders, and from that point forward, the same workflow can be triggered on demand with natural language commands and dynamic data inputs - without re-recording or manual scripting.

The core value proposition: **Record once. Replay forever with different data.**

**2\. System Actors**

| Actor                  | Role                                                                   | Interaction Mode             |
| ---------------------- | ---------------------------------------------------------------------- | ---------------------------- |
| **Flow Author**        | Records the process, reviews extracted steps, defines parameter schema | Web UI (Angular)             |
| **Flow Runner**        | Triggers automation runs with dynamic data via commands                | Web UI - Command Interface   |
| **System (AI Layer)**  | Parameterizes Codegen output, parses NLP commands                      | Internal - Ollama / Groq API |
| **Automation Engine**  | Executes Playwright scripts against target web apps                    | Backend subprocess runner    |
| **Target Application** | The web app being automated (e.g., timesheet portal)                   | Playwright browser session   |

In MVP, Flow Author and Flow Runner are the same person. Multi-user roles are future Phase concern.

**3\. System Boundaries**

**In Scope (MVP):**

- Playwright Codegen-based live recording
- LLM-driven parameterization of recorded scripts
- Storing flows as structured JSON in SQLite
- Pre-run parameter collection (credentials + dynamic data)
- Playwright script execution with injected parameters
- Real-time log streaming via WebSocket
- Run history and status tracking

**Out of Scope (MVP):**

- Video file upload and frame extraction (Mode A - deferred to Phase 3)
- Multi-user accounts and access control
- Scheduled / cron-based automation runs
- Selector drift detection and auto-healing
- Mobile browser automation
- Captcha solving or 2FA bypass

**4\. Core Use Cases**

**UC-01: Record a New Flow via Playwright Codegen**

**Actor:** Flow Author  
**Trigger:** User wants to automate a repetitive browser-based process  
**Preconditions:**

- Playwright is installed on the user's machine
- The target web application is accessible

**Main Flow:**

- User navigates to PlayRecord → clicks **"New Flow"**
- User enters a flow name and optional description
- System displays instructions to launch Playwright Codegen in terminal
- User opens terminal and runs:

npx playwright codegen <https://target-app.com>

- A browser window opens alongside a Codegen inspector panel
- User performs the full process manually (login → navigate → fill → submit → logout)
- Codegen generates a raw TypeScript script in real time
- User copies the generated .ts script content
- User pastes the script into the PlayRecord UI upload panel
- System saves the raw script and triggers the parameterization pipeline (→ UC-02)

**Postconditions:**

- Raw Codegen script is stored in the database against the flow record
- Parameterization pipeline is initiated

**Alternative Flows:**

- **A1:** User uploads a .ts file directly instead of pasting text
- **A2:** User abandons mid-recording - system saves a draft with no steps, allows resuming later

**Edge Cases:**

- **E1:** Pasted content is not valid TypeScript → system shows a parse error with line reference
- **E2:** Script contains no actionable steps (only navigations) → system warns user before saving
- **E3:** Codegen session times out on the target app due to inactivity → user must re-record; system does not partially save broken sessions

**UC-02: Parameterize Raw Script into Flow JSON**

**Actor:** System (AI Layer - Ollama llama3.2 or Groq)  
**Trigger:** Raw Codegen script is saved (auto-triggered after UC-01)  
**Preconditions:**

- Valid TypeScript Playwright script exists in the database
- AI service (Ollama local or Groq API) is reachable

**Main Flow:**

- Backend extracts the raw .ts script text from the database
- System sends the script to the LLM with a structured parameterization prompt:
  - Identify all hardcoded dynamic values (dates, usernames, hours, names)
  - Replace them with {{PARAM_KEY}} placeholders in SCREAMING_SNAKE_CASE
  - Flag credential fields (password inputs) as is_sensitive: true
  - Output a structured JSON array of steps + a parameter schema object
- LLM returns parameterized JSON
- System validates the JSON structure against the FlowStep schema
- Steps are saved to flow_steps table; parameter schema saved to flows.param_schema
- Flow status updated from draft → ready
- User is notified (UI updates automatically via polling or WebSocket event)

**Postconditions:**

- flow_steps table populated with ordered, parameterized steps
- param_schema JSON stored with credentials and run_params sections
- Flow is in ready status and available to trigger

**Alternative Flows:**

- **A1:** LLM is unavailable (Ollama not running / Groq rate limited) → system retries 3 times with exponential backoff, then marks flow as parameterization_failed with error message
- **A2:** LLM returns malformed JSON → system attempts JSON repair using regex cleanup; if repair fails, marks as parameterization_failed

**Edge Cases:**

- **E1:** LLM over-parameterizes - replaces static values like button labels with placeholders → system runs a confidence filter; low-confidence replacements are flagged for user review
- **E2:** LLM misses a dynamic value (e.g., leaves a hardcoded date) → user can manually edit steps in UC-03
- **E3:** Script contains a loop (e.g., filling 5 rows in a table) → LLM should collapse the loop into a single parameterized step with a repeat_count or items param; if it cannot, it expands to N individual steps and flags them for user review
- **E4:** Script contains conditional logic (if/else) → system flattens to the observed execution path and adds a note in the step description

**UC-03: Review and Edit Extracted Flow Steps**

**Actor:** Flow Author  
**Trigger:** User opens a flow that has been parameterized  
**Preconditions:**

- Flow exists with status ready or parameterization_failed

**Main Flow:**

- User opens a flow from the Flow Library
- System displays a step-by-step timeline showing:
  - Step number, action type icon, selector hint, value/param key
  - Sensitive fields visually distinguished (lock icon)
- User reviews each step for accuracy
- User can:
  - Edit selector_hint text inline
  - Change action_type via dropdown
  - Toggle is_sensitive flag
  - Change or remove a param_key
  - Reorder steps via drag-and-drop
  - Delete a step
  - Insert a new manual step between existing steps
- User clicks **"Save Changes"**
- System updates the step records and regenerates the parameter schema

**Postconditions:**

- Steps reflect user's corrections
- Parameter schema is regenerated from the updated steps
- Flow status remains ready

**Edge Cases:**

- **E1:** User deletes a step that is the only navigate action → system warns that removing navigation may break the flow
- **E2:** User edits a param_key that was referenced in a previously saved run's history → system warns but allows the change; old run records are not affected
- **E3:** User reorders steps in a way that puts a fill before a navigate → system shows a logical sequence warning (non-blocking)

**UC-04: Define and Edit Parameter Schema**

**Actor:** Flow Author  
**Trigger:** User wants to review, rename, or configure the parameters for a flow  
**Preconditions:**

- Flow exists with status ready

**Main Flow:**

- User navigates to the **Parameters** tab on the flow detail page
- System displays auto-generated parameter schema in two sections:
  - **Credentials** (sensitive fields - rendered as password inputs at run time)
  - **Run Parameters** (dynamic data - dates, hours, counts, names)
- For each parameter, user can edit:
  - Display label (e.g., HOURS_PER_DAY → "Hours Per Day")
  - Input type (string, number, date, password)
  - Required / optional toggle
  - Default value (for non-sensitive params only)
- User clicks **"Save Schema"**
- System stores updated schema

**Postconditions:**

- Parameter schema reflects user's customizations
- Pre-run dialog (UC-05) will render fields using this updated schema

**Edge Cases:**

- **E1:** User sets a credential field to non-sensitive → system shows a strong warning but allows it
- **E2:** User removes a required parameter that is referenced in a step's param_key → system blocks removal with an error: "This parameter is used in Step N"
- **E3:** User adds a default value for a date type param → system validates it is a parseable date string
- **E4:** User marks a param as optional but a step strictly requires it → system warns at run time if the optional param is left blank

**UC-05: Trigger a Flow Run (Manual)**

**Actor:** Flow Runner  
**Trigger:** User clicks "Run Flow" on a flow detail page  
**Preconditions:**

- Flow is in ready status
- Flow has at least one step
- Target web application is accessible from the machine running the automation

**Main Flow:**

- User clicks **"Run Flow"** button
- System opens a **Pre-Run Dialog** modal
- Modal renders two sections from the parameter schema:
  - **Credentials** - password-masked input fields (never stored)
  - **Run Parameters** - typed inputs based on schema (date pickers, number inputs, text fields)
- Pre-populated fields show default values where defined
- User fills all required fields
- User clicks **"Trigger Run"**
- System validates all required fields are filled
- System creates a RunJob record with:
  - status: pending
  - params: non-sensitive params only (credentials excluded from persistence)
  - triggered_by: manual
- System returns a job_id immediately (HTTP 202 Accepted)
- Frontend navigates to Run Console (/runs/:jobId) - UC-07
- Backend background task starts the Playwright execution - UC-06

**Postconditions:**

- RunJob record created in DB
- Playwright execution begins asynchronously
- User is watching the live Run Console

**Alternative Flows:**

- **A1:** User closes the Pre-Run Dialog → run is cancelled, no job record created

**Edge Cases:**

- **E1:** User leaves a required credential field empty → system blocks submission with inline validation error
- **E2:** User enters an invalid date format → system shows field-level error before submission
- **E3:** Two runs of the same flow triggered simultaneously → system allows it (parallel runs are independent job records); a future version may add a concurrency limit
- **E4:** The target app's base URL has changed since recording → run will fail at the first navigate step; error will appear in the Run Console log

**UC-06: Execute Automation via Playwright**

**Actor:** System (Automation Engine)  
**Trigger:** RunJob created with status pending (auto-triggered from UC-05)  
**Preconditions:**

- Node.js and Playwright are installed on the backend server
- RunJob record exists with status: pending
- Generated .ts Playwright script exists for the flow

**Main Flow:**

- Backend retrieves the parameterized script template for the flow
- System injects runtime params (including in-memory credentials) into the script, replacing {{PARAM_KEY}} tokens
- System writes the resolved script to a temporary file at /tmp/run\_{job_id}.ts
- System updates RunJob.status → running, sets started_at
- System spawns a Node.js subprocess:

npx ts-node /tmp/run\_{job_id}.ts

- stdout/stderr of the subprocess is captured line by line
- Each log line is:
  - Saved as a RunLog record in the database
  - Broadcast via WebSocket to any connected clients watching this job_id
- On subprocess exit:
  - Exit code 0 → RunJob.status = success
  - Non-zero exit code → RunJob.status = failed
  - finished_at is set in both cases
- Temporary script file is deleted from /tmp/
- Final status is broadcast via WebSocket

**Postconditions:**

- RunJob status is success or failed
- All log lines persisted in run_logs table
- Temporary script file cleaned up

**Edge Cases:**

- **E1:** Subprocess hangs (target app unresponsive) → a configurable timeout (default: 5 minutes) kills the subprocess and marks job as failed with "Execution timeout" log line
- **E2:** Playwright cannot find an element matching the selector hint → Playwright throws a TimeoutError; this is captured as an error log line and job is marked failed
- **E3:** Target app shows an unexpected modal/dialog (cookie banner, session expired) → step fails; user must update the flow to handle this step
- **E4:** Credentials are wrong → login step fails; job marked failed; credentials are never logged (system filters out sensitive param values from log output)
- **E5:** Backend server restarts mid-run → job remains in running status indefinitely; a startup recovery job should scan for stale running jobs older than the timeout and mark them failed
- **E6:** Multiple tabs or navigation errors during execution → Playwright context is fully cleaned up even on failure (browser instance is always closed in a finally block)

**UC-07: Monitor Run in Real Time (Run Console)**

**Actor:** Flow Runner  
**Trigger:** Navigated to /runs/:jobId after triggering a run  
**Preconditions:**

- RunJob record exists
- WebSocket server is running

**Main Flow:**

- Page loads and fetches initial job metadata via GET /runs/:jobId
- Angular WebSocketService connects to ws://backend/ws/runs/:jobId/logs
- As log messages arrive, they are appended to the console list in real time
- Each log line renders with:
  - Timestamp
  - Color-coded level badge: INFO (blue) / WARNING (yellow) / ERROR (red) / SUCCESS (green)
  - Message text
- Console auto-scrolls to the latest log line
- Status indicator at top updates dynamically: Pending → Running → Success / Failed
- On job completion, WebSocket connection closes gracefully
- User sees a final status banner: "✅ Run Completed Successfully" or "❌ Run Failed at Step N"

**Postconditions:**

- User has full visibility of what happened during execution

**Alternative Flows:**

- **A1:** User navigates to a past run → logs are loaded from DB (no WebSocket), displayed statically
- **A2:** WebSocket disconnects mid-run → system shows a "Reconnecting..." indicator and attempts reconnect with exponential backoff (3 attempts)

**Edge Cases:**

- **E1:** User opens the Run Console after the job has already completed → system detects status != running, skips WebSocket, loads persisted logs from DB
- **E2:** Very long runs produce hundreds of log lines → UI uses virtual scrolling to avoid DOM performance degradation
- **E3:** Sensitive param values (passwords) appear in Playwright's native error output → backend log sanitizer strips known sensitive param keys from all log text before saving or broadcasting

**UC-08: Trigger a Flow via Natural Language Command**

**Actor:** Flow Runner  
**Trigger:** User types a natural language command in the Command Interface  
**Preconditions:**

- At least one flow exists with status ready
- NLP service (Groq API or Ollama) is reachable

**Main Flow:**

- User opens the **Command Interface** panel
- User types a command, e.g.:

"Fill my timesheet with 8 hrs per day from Mon to Fri (Jun 1 - Jun 5)"

- System sends the command + list of available flow names/descriptions to the LLM
- LLM returns a structured RunRequest:

{  
"flow_id": "abc-123",  
"confidence": "high",  
"resolved_params": {  
"HOURS_PER_DAY": 8,  
"WEEK_START_DATE": "2025-06-01",  
"WEEK_END_DATE": "2025-06-05"  
},  
"missing_params": \["USERNAME", "PASSWORD"\]  
}

- If missing_params is non-empty, system opens a focused modal to collect only the missing fields
- User fills missing fields → clicks **"Confirm & Run"**
- System merges resolved params + user-provided missing params
- Proceeds to UC-05 Step 8 onwards (creates RunJob, navigates to Run Console)

**Postconditions:**

- Run is triggered with all params resolved
- Command text is stored in RunJob.command_text for history

**Edge Cases:**

- **E1:** LLM maps the command to the wrong flow → system shows the matched flow name with a "Not the right flow?" option to manually select a different one
- **E2:** LLM cannot confidently map to any flow (confidence: low) → system shows all available flows and asks user to select manually
- **E3:** Command contains ambiguous dates ("next Monday") → LLM resolves relative to current date; system displays the resolved date for user confirmation before running
- **E4:** Command includes credential values ("login as john with password abc123") → system should instruct the LLM via system prompt to never extract or store credential values from commands; credentials always come from the secure pre-run modal only
- **E5:** NLP service is unavailable → system falls back to showing the manual Pre-Run Dialog for the user to select a flow and fill params manually

**UC-09: View Flow Run History**

**Actor:** Flow Runner  
**Trigger:** User navigates to run history for a flow or global run history  
**Preconditions:** At least one RunJob record exists

**Main Flow:**

- User opens the **Run History** tab on a flow detail page, or the global Runs list
- System displays a paginated list of runs showing:
  - Run timestamp
  - Triggered by (manual / NLP command)
  - Command text (if triggered by NLP)
  - Status badge (success / failed / running)
  - Duration
- User clicks a run → navigates to the static Run Console view (logs loaded from DB)
- Failed runs show the step number where failure occurred (derived from last error log)

**Edge Cases:**

- **E1:** User attempts to re-run a historical run → system opens Pre-Run Dialog pre-filled with non-sensitive params from the historical run's params field; credentials must always be re-entered
- **E2:** Deleted flow's run history → run records are retained in DB with a \[Deleted Flow\] label for audit purposes

**UC-10: Delete a Flow**

**Actor:** Flow Author  
**Trigger:** User clicks "Delete Flow" on a flow detail page  
**Preconditions:** Flow exists

**Main Flow:**

- User clicks **"Delete Flow"**
- System shows a confirmation dialog: "This will delete the flow and all its steps. Run history will be retained."
- User confirms
- System soft-deletes the flow (sets deleted_at timestamp, does not hard-delete)
- Flow disappears from the Flow Library
- Associated flow_steps are deleted
- Associated run_jobs and run_logs are retained with flow_id reference intact

**Edge Cases:**

- **E1:** User tries to delete a flow with an active running job → system blocks deletion: "A run is currently in progress. Please wait for it to complete."
- **E2:** User wants to permanently delete including run history → available only via an explicit "Permanently Delete" option in settings (not the default delete action)

**5\. Non-Functional Requirements**

**5.1 Performance**

- Flow parameterization (UC-02) should complete within 30 seconds for scripts up to 100 steps
- Run Console WebSocket log latency should be under 500ms from subprocess stdout to browser display
- Flow Library page should load within 2 seconds for up to 100 flows

**5.2 Security**

- Credential params (is_sensitive: true) must never be persisted to the database at any point
- Credential values must be filtered from all log output before saving or broadcasting
- Temporary Playwright script files in /tmp/ must be deleted immediately after execution, whether the run succeeds or fails
- LLM prompts must explicitly instruct the model never to extract credential values from natural language commands

**5.3 Reliability**

- All Playwright subprocess executions must run inside a try/finally block ensuring browser cleanup
- Jobs stuck in running status beyond the timeout threshold must be auto-resolved to failed on server startup
- WebSocket disconnections on the client must trigger automatic reconnect with 3 attempts before showing a manual refresh prompt

**5.4 Usability**

- The pre-run parameter dialog must clearly separate credential fields from run parameter fields
- Sensitive fields must render as password inputs with a show/hide toggle
- Error messages from failed runs must reference the specific step number, not just "run failed"

**6\. Data Models (Summary)**

| Table      | Key Fields                                                                                              | Purpose                                            |
| ---------- | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| flows      | id, name, status, raw_script, param_schema (JSON)                                                       | Stores flow definition and parameterization output |
| flow_steps | id, flow_id, order, action_type, selector_hint, value, param_key, is_sensitive                          | Individual ordered steps of a flow                 |
| run_jobs   | id, flow_id, status, params (JSON, no credentials), triggered_by, command_text, started_at, finished_at | Execution history record                           |
| run_logs   | id, job_id, level, message, timestamp                                                                   | Per-line log output from Playwright subprocess     |

**7\. API Endpoints (Summary)**

**Flows**

| Method | Endpoint                    | Description                                           |
| ------ | --------------------------- | ----------------------------------------------------- |
| POST   | /flows                      | Create a new flow                                     |
| GET    | /flows                      | List all flows                                        |
| GET    | /flows/{id}                 | Get flow with steps and schema                        |
| PUT    | /flows/{id}                 | Update flow metadata                                  |
| DELETE | /flows/{id}                 | Soft-delete a flow                                    |
| POST   | /flows/{id}/script          | Upload raw Codegen script → triggers parameterization |
| PUT    | /flows/{id}/steps/{step_id} | Edit a step                                           |
| DELETE | /flows/{id}/steps/{step_id} | Delete a step                                         |
| PUT    | /flows/{id}/schema          | Update parameter schema                               |

**Runs**

| Method | Endpoint       | Description                                |
| ------ | -------------- | ------------------------------------------ |
| POST   | /runs          | Trigger a run (returns job_id immediately) |
| GET    | /runs          | List all run jobs                          |
| GET    | /runs/{job_id} | Get run detail + logs                      |
| POST   | /runs/command  | Trigger via NLP command (parse + run)      |

**WebSocket**

| Endpoint                  | Description                        |
| ------------------------- | ---------------------------------- |
| WS /ws/runs/{job_id}/logs | Stream live logs for a running job |

**8\. MVP Acceptance Criteria**

The MVP is considered complete when all of the following are true:

- \[ \] A user can record a Playwright Codegen session and paste the script into the UI
- \[ \] The system automatically parameterizes the script and produces a valid flow JSON with steps and param schema
- \[ \] A user can review, edit, reorder, and delete steps from the UI
- \[ \] A user can trigger a run by filling a pre-run dialog with credentials and dynamic params
- \[ \] Playwright executes the flow against the real target application with injected parameters
- \[ \] The Run Console shows live log output with correct color-coded levels
- \[ \] A user can type a natural language command and the system resolves it to a flow + params
- \[ \] Run history is viewable with per-run log replay
- \[ \] No credential values are ever written to the database or appear in log output
- \[ \] Failed runs display the step number and error message clearly

**9\. Known Risks and Mitigations**

| Risk                                                    | Likelihood          | Impact | Mitigation                                                               |
| ------------------------------------------------------- | ------------------- | ------ | ------------------------------------------------------------------------ |
| LLM parameterization misses dynamic values              | Medium              | High   | User step review (UC-03) acts as a manual correction layer               |
| Playwright selector fails on target app after UI update | High                | High   | Deferred to Phase 4: selector drift detection and auto-healing           |
| Ollama local model too slow for acceptable UX           | Medium              | Medium | Offer Groq API as a fallback for the parameterization step               |
| Target app has bot detection / rate limiting            | Low (internal apps) | High   | Out of scope for MVP; document as a known limitation                     |
| Temporary script files not cleaned up on crash          | Low                 | Medium | Server startup cleanup job scans and deletes stale /tmp/run\_\*.ts files |
| Concurrent runs overwhelming the backend                | Low                 | Medium | Add a max concurrent runs limit (configurable, default: 3) in Phase 2    |

_End of Document - Version 1.0_
