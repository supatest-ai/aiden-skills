---
name: aiden-document
version: 1.0.1
description: Create a structured document artifact in Aiden
---

# Document Creation

You are creating a structured document artifact. Follow these phases carefully.

---

## Phase 1: Understand the Request

- What type of document? (analysis, RFC, architecture doc, runbook, guide, etc.)
- What format? Use **markdown** by default; use **HTML** only for rich visual content
- What is the audience and purpose?
- Gather relevant context from the codebase, task description, or conversation

## Phase 2: Write the Document

- Research the codebase and context as needed
- Write clear, well-structured content with appropriate headings
- Include code examples and references where helpful
- Keep the document focused and concise

**Be visual.** Documents render mermaid natively. Use diagrams to make
structure, flows, and relationships immediately clear — pick the right type:

| Diagram | When to use | Mermaid type |
|---|---|---|
| **Sequence** | Service-to-service interactions, API call chains | `sequenceDiagram` |
| **ER (Schema)** | Database tables & relationships | `erDiagram` |
| **Flowchart** | Logic, process flows, decision paths | `flowchart TD` |
| **State** | Entity lifecycle, status transitions | `stateDiagram-v2` |
| **C4 Container** | System architecture overview, service boundaries | `C4Container` |
| **Data Flow** | How data moves end-to-end through the system | `flowchart LR` |

Place diagrams at the **top** of the relevant section to ground the reader.
PRDs, specs, RFCs, and architecture docs should almost always include at least
one diagram. Don't diagram trivial content.

## Phase 3: Persist the Document (MANDATORY)

**CRITICAL: You MUST call `mcp__aiden__create_document_artifact` to complete this task. Do NOT output the document as plain text in the chat. The document MUST be persisted via this MCP tool call so the UI can render it as a proper artifact. Skipping this step means the work was wasted.**

Call the `mcp__aiden__create_document_artifact` MCP tool:

Use the active task context when it is present in the prompt. Otherwise:
- The task ID is available from the `AIDEN_TASK_ID` environment variable.
- The conversation ID is available from the `AIDEN_SESSION_ID` environment variable.
- Resolve `teamId` with Aiden MCP context/tools before creating the document artifact.

Always pass `taskId`, `conversationId`, and `teamId` explicitly. Do not
substitute one ID for another.

```
Tool: mcp__aiden__create_document_artifact
Parameters: {
  "title": "<document title>",
  "taskId": "<active task ID or value of AIDEN_TASK_ID, if set>",
  "conversationId": "<active conversation ID or value of AIDEN_SESSION_ID>",
  "teamId": "<resolved team ID>",
  "format": "markdown",
  "content": "<full document content>",
  "summary": "<1-2 sentence summary>"
}
```

### Field guidelines

- **title**: Clear, descriptive title for the document
- **format**: `"markdown"` (default) or `"html"` for rich content
- **content**: The complete document content
- **summary**: Brief description of what the document covers

### After creating the document

Tell the user:
- The document title, so the user can find the generated artifact in the Aiden UI
- A link to the artifact when you have enough context to form one (usually `/teams/<teamId>/docs?artifactId=<artifactId>`)
- A brief summary of what was documented

Do not lead with a raw UUID. Only include the artifact ID as secondary debug context if no usable title or link is available.

**Do NOT render the full document as markdown in chat. The MCP tool creates it as a structured artifact that the UI displays with proper formatting.**
