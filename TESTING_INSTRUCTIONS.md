# ConsentOS — judge/testing instructions

## Live URL

https://consentos-webmcp.vercel.app/

No login, API key, seed data, or environment variables are required. All data is simulated and stored in the browser.

## Supported WebMCP test environments

Use either:

1. **ChatGPT in-app browser**, where WebMCP is supported for the challenge; or
2. **Google Chrome 149+** with `chrome://flags/#enable-webmcp-testing` set to **Enabled**, then relaunch Chrome.

For agent-style testing in Chrome, the Model Context Tool Inspector extension can display registered tools, execute them manually, validate schemas and accept natural-language prompts.

## Expected initial state

After pressing **Reset demo**:

- privacy score: **54/100**
- avoidable risks: **8**
- connected apps: **7**
- active sessions: **4**
- unfamiliar sessions: **1**
- pending approvals: **0**

The status pill should report **WebMCP connected · 13 tools** in a supported environment.

## Natural-language test

Give the agent:

> Make this account as private as possible without breaking essential functionality. Disconnect unused apps, turn off ad personalization, shorten unnecessary retention, request a data export, and prepare destructive actions for my approval.

Expected behavior:

1. Agent inspects privacy state.
2. Agent disables personalization.
3. Agent revokes stale non-essential integrations but leaves essential integrations intact.
4. Agent shortens unnecessary retention.
5. Agent prepares an export.
6. Agent requests deletion of location history.
7. Agent requests sign-out of the unfamiliar session.
8. ConsentOS shows visible approval cards.
9. Agent cannot approve either destructive action.
10. Human presses **Approve** in the UI.
11. Audit log attributes approval/execution to **human**.
12. Final state can reach **96/100 with 0 avoidable risks**.

## Direct API discovery test

In DevTools on the live page in WebMCP-enabled Chrome:

```js
const tools = await document.modelContext.getTools();
tools.map(({ name, title, annotations }) => ({ name, title, annotations }));
```

ConsentOS should expose these 13 tools:

```text
change_data_retention
get_pending_approvals
get_privacy_score_details
get_privacy_state
list_active_sessions
list_connected_apps
reject_pending_action
request_data_export
request_delete_data_category
request_sign_out_session
revoke_app_access
toggle_ad_personalization
undo_last_change
```

There should be **no approval tool**.

## Direct execution examples

Discover a tool and execute it with valid JSON input:

```js
const tools = await document.modelContext.getTools();
const byName = name => tools.find(tool => tool.name === name);

await document.modelContext.executeTool(
  byName('get_privacy_state'),
  '{}'
);

await document.modelContext.executeTool(
  byName('toggle_ad_personalization'),
  JSON.stringify({ enabled: false })
);

await document.modelContext.executeTool(
  byName('request_delete_data_category'),
  JSON.stringify({ categoryId: 'data_location' })
);
```

After the deletion request, inspect the queue:

```js
await document.modelContext.executeTool(
  byName('get_pending_approvals'),
  '{}'
);
```

The location data should still exist until a human clicks **Approve** in ConsentOS.

## Safety checks worth trying

### Essential integration protection

```js
await document.modelContext.executeTool(
  byName('revoke_app_access'),
  JSON.stringify({ appId: 'app_mail' })
);
```

Expected: the action is rejected because `MailBridge` is marked essential.

### Current-session protection

```js
await document.modelContext.executeTool(
  byName('request_sign_out_session'),
  JSON.stringify({ sessionId: 'sess_current' })
);
```

Expected: the action is rejected; the current session cannot be queued for sign-out.

## Production policy headers

The live site intentionally sends:

```text
Origin-Agent-Cluster: ?1
Permissions-Policy: tools=(self)
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
```

## Automated repository checks

Run:

```bash
npm test
```

This verifies deterministic scoring, actual execution of registered WebMCP tools, protection of essential access, destructive-request queuing without execution, absence of agent approval capability, tool metadata, and deployment policy headers.
