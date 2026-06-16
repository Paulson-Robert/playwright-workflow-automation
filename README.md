# Playwright Workflow Automation

A workflow automation tool that converts a live browser recording session into a reusable, parameterized automation flow.

## How It Works

1. **Record** — Use Playwright Codegen to record a browser workflow once
2. **Parameterize** — AI extracts dynamic values and generates a reusable JSON flow definition
3. **Replay** — Trigger the same workflow anytime with different data via UI or natural language command

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Angular |
| Backend | Python FastAPI |
| Database | SQLite |
| AI Layer | Ollama (local) / Groq API |
| Automation | Playwright TypeScript |

## Project Structure
playwright-workflow-automation/
├── frontend/ # Angular application
├── backend/ # Python FastAPI server
├── automation/ # Playwright TypeScript scripts
├── docs/ # Project documentation

text

## Getting Started

See `docs/usecase-plan.md` for the full use case specification.

## Branches

- `main` — stable, production-ready
- `dev` — active development, all PRs merge here
