# ConsentOS

**Agent-managed privacy without agent-controlled consent.**

[![ConsentOS checks](https://github.com/nexora-assistant/consentos-webmcp/actions/workflows/test.yml/badge.svg)](https://github.com/nexora-assistant/consentos-webmcp/actions/workflows/test.yml)

**Live demo:** https://consentos-webmcp.vercel.app/

ConsentOS is a WebMCP Challenge prototype that turns fragmented privacy settings into an explicit agent-native capability surface while keeping destructive authorization human-only. All account data in the demo is fictional and stored locally in the browser.

## Why WebMCP matters here

A visual agent can try to hunt through connected-app pages, retention dropdowns, session controls, export flows and deletion dialogs. That is brittle and obscures what authority the agent actually has. ConsentOS instead exposes typed page tools with structured schemas and readable metadata.

The product is intentionally asymmetric:

- **Agent can:** inspect state, explain score, revoke non-essential apps, disable personalization, shorten retention, prepare exports, queue deletion and queue session sign-out.
- **Human only:** approve destructive deletion or session termination through the visible ConsentOS interface.

There is deliberately **no WebMCP approval tool**.

## WebMCP implementation

ConsentOS uses the current imperative API:

```js
await document.modelContext.registerTool(tool, { signal: controller.signal });
```

It uses `document.modelContext`, not deprecated `navigator.modelContext`. Tool registration is cleaned up with `AbortSignal`. When `getTools()` is available, ConsentOS verifies how many of its own tools are actually registered and surfaces that count in the UI.

### Registered tools — 13

| Tool | Mode | Purpose |
|---|---|---|
| `get_privacy_state` | Read | Full account state and score |
| `get_privacy_score_details` | Read | Deterministic scoring factors |
| `list_connected_apps` | Read | Linked apps, scopes and risk |
| `revoke_app_access` | Reversible write | Revoke non-essential app access |
| `toggle_ad_personalization` | Reversible write | Reduce targeting |
| `change_data_retention` | Reversible write | Shorten retention |
| `request_data_export` | Write | Prepare an export |
| `request_delete_data_category` | Gated request | Queue deletion for human approval |
| `list_active_sessions` | Read | Active/familiar sessions |
| `request_sign_out_session` | Gated request | Queue session sign-out |
| `get_pending_approvals` | Read | Inspect approval queue |
| `reject_pending_action` | Write | Cancel a queued action |
| `undo_last_change` | Reversible write | Restore previous state |

Every tool has a human-readable `title`, JSON input schema, and explicit `readOnlyHint` annotation.

## Signature demo flow

Start at **54/100 with 8 avoidable risks** and give the agent this instruction:

> Make this account as private as possible without breaking essential functionality. Disconnect unused apps, turn off ad personalization, shorten unnecessary retention, request a data export, and prepare destructive actions for my approval.

The agent inspects state and performs safe/reversible actions. Deletion and suspicious-session termination become visible approval cards. The human approves them, the audit log attributes that decision to the human, and the account reaches **96/100 with 0 avoidable risks**.

ConsentOS intentionally never claims perfect privacy.

## Trust and deployment hardening

Production sends explicit browser-policy headers:

```text
Origin-Agent-Cluster: ?1
Permissions-Policy: tools=(self)
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
```

The app progressively degrades in ordinary browsers: the privacy workspace still works even when WebMCP is unavailable.

## 3D + motion interface

- holographic score instrument with orbiting rings
- animated ambient light fields and grid
- glass surfaces with layered depth
- pointer-driven 3D card tilt on desktop
- subtle score float and status glow
- responsive mobile layout
- `prefers-reduced-motion` support
- visible **Agent can / Human only** authority card

## Automated checks

```bash
npm test
```

The test suite proves:

1. seeded score/risk state is exactly **54 / 8** and full hardening reaches **96 / 0**;
2. exactly **13 WebMCP tools** register and no agent approval capability exists;
3. every tool exposes expected metadata/annotations;
4. the production WebMCP origin and permissions headers are present in `vercel.json`.

GitHub Actions runs the same checks on every push and pull request to `main`.

## Project structure

- `index.html` — application shell
- `styles.css` + `polish.css` — 3D/motion visual system
- `src/state.js` — deterministic state, scoring, undo and approval enforcement
- `src/webmcp.js` — 13 WebMCP tools and lifecycle
- `src/ui.js` — responsive interface, interactions and authority visualization
- `src/main.js` — startup and WebMCP status reporting
- `tests/` — deterministic state, WebMCP contract and deployment tests
- `DEMO_SCRIPT.md` — <3 minute demo plan
- `DEVPOST_SUBMISSION.md` — submission copy

## Local run

No build step is required:

```bash
python3 -m http.server 4173
```

Open `http://localhost:4173`. Agent interaction requires a WebMCP-capable browser/environment exposing `document.modelContext`.

## Limitations

ConsentOS is a safe hackathon simulation. It does not touch real accounts, third-party services, sessions, or personal data. A production implementation would require authenticated provider APIs, server-side authorization, secure approval enforcement, signed audit records, and formal privacy/security review.

## License

MIT — see `LICENSE`.
