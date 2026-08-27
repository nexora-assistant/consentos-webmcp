# ConsentOS — Devpost submission draft

## Tagline
Agent-managed privacy without agent-controlled consent.

## Public source
https://github.com/nexora-assistant/consentos-webmcp

## Live demo
https://consentos-webmcp.vercel.app/

## Inspiration
Privacy settings are fragmented across connected apps, tracking controls, retention policies, active sessions, exports and deletion workflows. That is frustrating for humans and brittle for visual browser agents. We wanted to explore an agent-native model without solving usability by giving an agent unlimited authority.

## What it does
ConsentOS is a simulated privacy control center that exposes **13 structured WebMCP tools**. An agent can inspect privacy posture, explain the score, disable personalization, revoke stale non-essential apps, shorten retention, prepare exports, and queue sensitive operations.

Destructive actions are deliberately separated from agent authority. The agent can request deletion of a data category or sign-out of a suspicious session, but only a human can approve those actions through the ConsentOS interface. There is intentionally **no WebMCP approval tool**.

The account begins at **54/100 with 8 avoidable risks**. A deterministic privacy-hardening workflow can reach **96/100 with 0 avoidable risks**. Every state change is reflected visibly in the dashboard, Agent activity rail and audit history.

## How we used WebMCP
ConsentOS uses the imperative API through `document.modelContext.registerTool(...)` and manages tool lifecycle with `AbortSignal`.

The tool surface includes:
- read-only discovery of account state, score, apps, sessions and approvals;
- reversible mutations for app access, personalization and retention;
- structured requests for destructive operations;
- rejection and undo.

Each tool has a human-readable title, JSON input schema and explicit `readOnlyHint`. When `document.modelContext.getTools()` is available, ConsentOS verifies how many of its own tools are actually registered and surfaces the count in the product UI.

The most important WebMCP design decision is what is **not** a tool: destructive approval. WebMCP expresses what the agent may do; the human remains a separate authority boundary.

## Human + agent collaboration
A single instruction can ask the agent to improve privacy across several settings. The agent performs safe/reversible work and prepares sensitive changes. ConsentOS then turns the sensitive requests into visible approval cards. The human makes the final destructive decision, and the audit log records that the authorization came from the human.

This creates a collaboration loop rather than either extreme: brittle manual navigation or unrestricted agent autonomy.

## Technical execution
- 13 WebMCP tools
- `document.modelContext`, not deprecated `navigator.modelContext`
- AbortSignal lifecycle cleanup
- post-registration tool-count verification when `getTools()` is available
- deterministic state engine, local persistence and undo
- human-attributed audit trail
- responsive 3D/motion UI with reduced-motion support
- explicit production headers: `Origin-Agent-Cluster: ?1` and `Permissions-Policy: tools=(self)`
- automated GitHub Actions checks for score calibration, tool boundary/metadata and deployment headers
- progressive enhancement when WebMCP is unavailable

## Challenges
The main challenge was balancing automation and authority. Exposing more tools can make an agent more capable, but it does not automatically make the system more trustworthy. We designed and tested a capability boundary where the agent can prepare consequential work without being able to authorize it.

A second challenge was making the demo deterministic. The privacy score is derived from actual simulated state rather than being a random or scripted animation, so the before/after result is reproducible.

## Accomplishments
- a clear human-only authorization boundary with no agent approval tool
- meaningful multi-tool WebMCP workflow rather than a single CRUD action
- deterministic **54/8 → 96/0** privacy-hardening path
- visible agent activity and human-attributed audit records
- responsive 3D/motion interface
- production WebMCP policy headers
- automated CI proving the key safety and scoring invariants

## What we learned
Agent-native websites benefit from explicit, typed capabilities, but consequential operations need an authorization model separate from capability discovery. WebMCP can make agents more reliable while also making the limits of their authority easier to reason about and test.

## What's next
A production ConsentOS would connect to authenticated privacy APIs from real services, enforce approvals server-side, support reusable privacy policies, cryptographically signed audit records, organization-level controls and scoped provider permissions.

## Demo prompt
> Make this account as private as possible without breaking essential functionality. Disconnect unused apps, turn off ad personalization, shorten unnecessary retention, request a data export, and prepare destructive actions for my approval.
