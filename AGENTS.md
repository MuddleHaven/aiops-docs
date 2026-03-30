# AGENTS.md

<!-- markdownlint-disable MD013 MD022 MD032 -->

This file guides agentic coding assistants operating in this repository.
Current workspace: `C:\Users\A200477427\Learnings\AIOps-Docs`.

## 1) Repository Overview
- Repository type: docs-first (no in-repo application runtime yet).
- Main artifacts: architecture/proposal markdown documents and diagrams.
- Key markdown docs:
  - `README.md`
  - `AIOps_Product_Architecture_and_Commercialization.md`
  - `AIOps_Architecture_Diagram_Explanation.md`
  - `AIOps_Project_Proposal.md`
- Key binary assets:
  - `AIOps_Practical_Route_Architecture.png`
  - `Implementable_AIOps_Platform_Route_A_Architecture.png`
  - `AIOps_智能运维平台项目立项书.pdf`

## 2) Rule Sources and Precedence
Follow instructions in this order:
1. System/developer/runtime instructions from the active harness.
2. Repository Cursor/Copilot rules (if present).
3. This `AGENTS.md` file.
4. Existing repository patterns.

Rule-file status checked in this repo:
- `.cursorrules`: not found.
- `.cursor/rules/`: not found.
- `.github/copilot-instructions.md`: not found.

Agent requirement:
- Re-check the 3 rule locations before major edits.
- If any are added later, treat them as higher priority than this file.

## 3) Optimization Goals
- Keep terminology consistent across all long-form docs.
- Prefer focused, incremental edits over broad rewrites.
- Keep markdown content and diagram references synchronized.
- Avoid inventing tooling/process assumptions not present in repo.

## 4) Build/Lint/Test Commands
Important: this repository currently has no formal build system, linter config, or test suite.

### 4.1 Current command reality
- Build command: not applicable.
- Lint command: not enforced by repository config.
- Test command: not applicable.
- Single-test command: not applicable (no tests exist yet).

### 4.2 Optional local documentation checks
Run only when Node.js is available in the environment.

```bash
# Lint all markdown files (optional)
npx markdownlint-cli2 "**/*.md"

# Lint one markdown file (single-file check)
npx markdownlint-cli2 "README.md"
```

### 4.3 Future test templates (if tests are added)
Update this section with actual project commands when a framework is introduced.

```bash
# Pytest: run all tests
pytest

# Pytest: run one test function
pytest tests/test_example.py::test_specific_case
```

## 5) Documentation Style Guidelines

### 5.1 Language and tone
- Default language is Chinese with technical English terms preserved.
- Keep writing professional, specific, and implementation oriented.
- Prefer concrete statements over vague strategic wording.
- Keep terms and definitions stable within each document.

### 5.2 Structure and formatting
- Use one top-level `#` heading per file.
- Use clear heading hierarchy (`##`, `###`) with logical progression.
- Keep numeric section prefixes when a document already uses them.
- Use `---` only when it improves readability.
- Use English names for new files and directories.
- Chinese content is allowed inside Markdown documents when appropriate.

### 5.3 Terminology consistency
- Wrap module/service identifiers in backticks.
- Keep canonical terms consistent across docs:
  - `Incident`
  - `RCA Result`
  - `NormalizedEvent`
  - `IncidentContext`
  - `execution-gateway`
  - `policy-engine`
- Keep status flow wording consistent:
  - `new -> triaged -> diagnosing -> remediating -> resolved/closed`

### 5.4 Lists, tables, and links
- Use bullets for principles, scope, constraints, and key points.
- Use ordered lists for procedures and time-based sequences.
- Use tables for role splits, capability matrices, and KPI definitions.
- Keep table headers explicit and unambiguous.
- Use relative links for local docs/images and verify exact filenames.

## 6) Code Style Guidelines (for future code/scripts)
This repo is docs-only today.
If code/scripts are added, apply these defaults unless project configs override them.

### 6.1 Imports
- Do not use wildcard imports.
- Group imports as: standard library, third-party, local modules.
- Keep one import per line unless language idioms require grouping.
- Remove unused imports in the same change.

### 6.2 Formatting
- Prefer automated formatters once configured.
- Keep functions small and focused.
- Avoid dead code and commented-out legacy blocks.

### 6.3 Types and contracts
- Add explicit types for public functions and interfaces.
- Prefer narrow, precise types over broad untyped structures.
- Validate external input at boundaries.
- Keep schema/type names aligned with AIOps domain terminology.

### 6.4 Naming
- Use descriptive, domain-accurate names.
- Avoid unclear abbreviations (except standard terms like RCA/KPI/SLA/MTTR).
- Python naming: `snake_case` for functions/variables, `PascalCase` for classes.
- TypeScript/JS naming: `camelCase` for functions/variables, `PascalCase` for types/classes.

### 6.5 Error handling and logging
- Fail fast on invalid state; do not silently swallow exceptions.
- Return actionable errors with debugging context.
- Separate user-facing errors from internal diagnostic details.
- Prefer structured logs including incident id/service/action when available.
- For automation actions, record actor, timestamp, and result.

## 7) Change Management
- Do not rename core docs unless explicitly requested.
- Preserve historical rationale in proposal/architecture documents.
- If canonical terms change in one doc, update related docs in the same change.
- Keep diagram references and explanation text mutually consistent.
- Prefer small, reviewable changes grouped by topic.
- When renaming files, update all local references in the same change.

## 8) Agent Completion Checklist
Before finishing a task, verify:
- Only necessary files changed.
- Terminology is consistent with existing AIOps docs.
- Local links and image references resolve.
- Any optional checks you ran are reported with outcomes.
- If no tests exist, state that explicitly.

If repository structure or tooling changes, update this file in the same PR.
