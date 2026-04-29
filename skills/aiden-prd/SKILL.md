---
name: aiden-prd
version: 1.0.1
description: Create a product requirements document through interactive interview, codebase exploration, and structured output — persisted as an Aiden document artifact
---

# Write a PRD

You are creating a Product Requirements Document. Your job is to deeply
understand the problem before writing anything — through interview,
exploration, and structured thinking.

---

## Process

### 1. Understand the problem

Ask the user for a detailed description of:
- The problem they want to solve
- Who it's for
- Any ideas they already have for solutions

If they've already described it in the task or conversation, acknowledge what
you know and confirm it's complete.

### 2. Explore the codebase

Before asking questions, explore. Understand:
- Current architecture and patterns relevant to this feature
- Existing code that this feature touches or extends
- Data models, API routes, UI components in the area
- Similar features already implemented (prior art)

This grounds your questions in reality, not abstractions.

### 3. Interview — one question at a time

Walk down each branch of the decision tree with the user. Resolve
dependencies between decisions one by one.

**Rules:**
- Ask **one question per message**. Not a list.
- For each question, provide your **recommended answer** based on what you've
  seen in the codebase and the user's description. Let the user confirm,
  correct, or redirect.
- If a question can be answered by exploring the codebase, explore instead of
  asking.
- Cover these areas (in whatever order makes sense):
  - Scope boundaries — what's in, what's out
  - User stories — who does what, and why
  - Edge cases — what happens when things go wrong
  - Data model implications — what schema changes are needed
  - API surface — new endpoints, modified contracts
  - UI/UX expectations — how should this look and feel
  - Dependencies — what needs to exist first
  - Testing expectations — what needs to be tested, what doesn't
- Don't rush this. A PRD is a contract. Ambiguity here becomes bugs later.

**When to stop:** When you can describe the feature end-to-end without
guessing. If you can answer "what happens when X?" for every scenario
the user cares about — you're done.

### 4. Draft the PRD

Write the PRD using the structure below. Adapt sections as needed — not every
PRD needs every section, and some need sections not listed here.

**Be visual.** Use mermaid diagrams for:
- Data flow through the system (`flowchart LR`)
- Schema relationships (`erDiagram`)
- State transitions (`stateDiagram-v2`)
- Service interactions (`sequenceDiagram`)

```markdown
# PRD: <Feature Name>

## Problem Statement
The problem from the user's perspective. Why does this matter.

## Solution
The solution from the user's perspective. What changes for them.

## User Stories
Numbered list. Each story: As a <actor>, I want <feature>, so that <benefit>.
Be extensive — cover the full surface area of the feature.

## Architecture & Data Flow
Mermaid diagrams showing how the feature fits into the system.
Schema changes, new API routes, component relationships.

## Implementation Decisions
Durable decisions made during the interview:
- Schema shapes and model names
- API contracts and route patterns
- UI component approach
- Third-party service boundaries
- Authentication/authorization implications

Do NOT include specific file paths or code snippets — they go stale fast.

## Edge Cases & Error Handling
What happens when things go wrong. Failure modes and how they're handled.

## Testing Strategy
- What makes a good test for this feature
- Which modules need tests
- Prior art (similar tests in the codebase)

## Out of Scope
What this PRD explicitly does NOT cover. Prevents scope creep.

## Open Questions
Anything unresolved. Better to surface unknowns than hide them.
```

### 5. Persist the PRD (MANDATORY)

**CRITICAL: You MUST call `mcp__aiden__create_document` to persist the PRD.
Do NOT output it as plain text in chat. The PRD must be a document artifact.**

```
Tool: mcp__aiden__create_document
Parameters: {
  "title": "PRD: <Feature Name>",
  "teamId": "<resolve via list_teams>",
  "content": "<full PRD content>",
  "summary": "PRD for <one-line description of the feature>"
}
```

If the PRD is tied to a specific task, use `mcp__aiden__create_document_artifact`
instead with the `taskId`, `conversationId`, and `teamId`. Use the active task
context when it is present in the prompt; otherwise read `AIDEN_TASK_ID` and
`AIDEN_SESSION_ID`, then resolve `teamId` with Aiden MCP context/tools.

After creating, tell the user:
- The PRD title, so the user can find the generated document/artifact in Aiden
- A link when you have enough context to form one:
  - Standalone document: `/teams/<teamId>/docs/<documentId>`
  - Task artifact: `/teams/<teamId>/docs?artifactId=<artifactId>`
- A brief summary of what was documented

Do not lead with a raw UUID. Only include the document/artifact ID as secondary debug context if no usable title or link is available.

## Do NOT

- Skip the interview and write a PRD from assumptions
- Ask all questions at once in a big list
- Include implementation-level details (file paths, function names) that go stale
- Skip the `create_document` MCP call — the PRD must be persisted
- Write the PRD in chat instead of as a document artifact
