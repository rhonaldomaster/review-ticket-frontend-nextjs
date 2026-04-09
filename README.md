# review-ticket-frontend-nextjs

A Claude Code skill that reviews a Jira ticket against the frontend codebase. It audits the existing Next.js/React implementation, identifies gaps, checks ticket quality, and optionally cross-references Tambora QA test cases.

## Requirements

The following tools must be available in the session:

- **Jira CLI** — `jira` must be installed and authenticated (`jira issue view` must work)
- **Tambora MCP** (`mcp__tambora__*`) — required only if a Tambora suite name is provided
  - `check_connectivity`
  - `list_test_cases`
  - `create_test_run_from_suite`
  - `add_test_run_results`
  - `complete_test_run`

## Usage

```
/review-ticket-frontend-nextjs <ticket-id> [tambora-suite-name] [--qa]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `ticket-id` | Yes | Jira ticket ID (e.g. `MPP-221`) |
| `tambora-suite-name` | No | Full Tambora suite name to cross-reference (e.g. `MPP-150 Memories list view`) |
| `--qa` | No | Enables QA mode — skips deep FE audit, shows testability status per test case, and walks through recording test run results |

### Examples

```bash
# Frontend audit only
/review-ticket-frontend-nextjs MPP-221

# Frontend audit + Tambora coverage analysis
/review-ticket-frontend-nextjs MPP-221 MPP-150 Memories list view

# QA mode — testability check + record test run results
/review-ticket-frontend-nextjs MPP-221 MPP-150 Memories list view --qa
```

## Modes

### Normal mode (default)

Intended for **frontend developers**. Runs a full analysis:

1. Fetches ticket details from Jira
2. Quick frontend readiness check (pages, components, API client, types)
3. Deep codebase audit (pages, components, hooks, API client, types, tests, Storybook)
4. Implementation status per acceptance criterion
5. Gap analysis (missing pages, components, tests, a11y, mobile, Storybook stories)
6. Ticket quality check (ambiguities, missing UI states, open decisions, security)
7. Tambora test case coverage table (if suite name provided) — maps each test case to a frontend coverage status
8. Optional: post findings as a comment to one or more Jira tickets

### QA mode (`--qa`)

Intended for **QA engineers**. Skips the deep frontend audit and focuses on testability:

1. Fetches ticket details from Jira
2. Quick frontend readiness check — produces a Yes / No / Partial table per acceptance criterion
3. Fetches Tambora test cases for the suite and marks each one as Testable / Not testable yet
4. Walks through recording test run results one case at a time:
   - Reuses an existing test run (by code) or creates a new one automatically
   - Asks for status per case: `passed`, `failed`, `skipped`, or `broken`
   - Optionally captures an error message for failed/broken cases
   - Shows a summary before submitting — no accidental writes
   - Submits all results in one batch
   - Optionally marks the test run as completed

## Tambora Test Run Flow (QA mode)

```
Do you have an existing run code? (e.g. TR-MPP-14)
  ├─ Yes → reuse it
  └─ No  → create new run from suite automatically

For each test case:
  [TC-MPP-XXXX] — Title
  Status? (passed / failed / skipped / broken) — Enter to skip
    └─ If failed/broken → optional error message

Show summary → confirm → submit batch → optionally complete run
```

## Output

### Normal mode report structure

- **Summary** — what the ticket asks for
- **Status** — current Jira status and subtasks
- **Frontend Readiness** — quick Yes/No/Partial per acceptance criterion
- **Frontend Implementation Audit** — what exists, with file paths and line numbers
- **Acceptance Criteria Coverage** — Implemented / Partial / Not implemented per criterion
- **Ticket Quality Issues** — ambiguities, missing UI states, open decisions
- **Recommendations** — actionable next steps
- **Tambora Test Case Coverage** — coverage status per test case (if suite provided)

### QA mode report structure

- **Summary** — what the ticket asks for
- **Status** — current Jira status
- **Frontend Readiness** — Yes / No / Partial per acceptance criterion, with warnings for untestable items
- **Tambora Test Cases** — list with Severity and Testable? columns

## Target Codebase

Designed for a **NextJS frontend**:

| Area | Paths |
|------|-------|
| Pages & layouts | `src/app/` |
| Route handlers | `src/app/api/` |
| Shared components | `src/components/` |
| Co-located components | `src/app/**/_components/` |
| Storybook stories | `**/__stories__/` |
| API client | `src/lib/api/` |
| Custom hooks | `src/lib/hooks/` |
| Utils & constants | `src/lib/utils/`, `src/lib/constants/` |
| Types & Zod schemas | `src/types/`, `**/*.types.ts` |
| Auth & middleware | `middleware.ts`, `src/lib/auth/`, `src/components/auth/` |
| Tests | `**/__tests__/`, `**/*.test.tsx` |
