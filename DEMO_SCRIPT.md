# ConsentOS — final <3 minute demo script

## Before recording

1. Open https://consentos-webmcp.vercel.app/ in the supported WebMCP environment.
2. Reset the demo so the score is **54/100** and the page shows **8 avoidable risks**.
3. Confirm the top status says **WebMCP connected · 13 tools**.
4. Keep the Agent activity rail visible when possible.

## 0:00–0:20 — Problem + product

Show the 54/100 dashboard, exposure map and 3D score instrument.

Narration:

“Privacy controls are fragmented across connected apps, tracking settings, retention policies, active sessions, exports and deletion flows. ConsentOS turns those settings into an agent-native privacy control plane, without giving the agent authority to approve destructive actions.”

## 0:20–0:40 — Prove WebMCP

Show the **WebMCP connected · 13 tools** status, then open **Agent tools** for a quick scan.

Narration:

“Instead of hunting for buttons, the page exposes 13 structured WebMCP tools with typed inputs, readable metadata and explicit read-only hints.”

Close the tool modal quickly.

## 0:40–1:35 — One prompt, multiple meaningful calls

Give the agent:

> Make this account as private as possible without breaking essential functionality. Disconnect unused apps, turn off ad personalization, shorten unnecessary retention, request a data export, and prepare destructive actions for my approval.

The ideal sequence is:

1. `get_privacy_state`
2. `toggle_ad_personalization` → `enabled: false`
3. revoke stale non-essential apps with `revoke_app_access`
4. shorten unnecessary retention with `change_data_retention`
5. `request_data_export`
6. `request_delete_data_category` for location history
7. `request_sign_out_session` for the unfamiliar session
8. `get_pending_approvals`

Keep the page visible while the score and Agent activity rail change.

## 1:35–2:10 — Signature authority moment

Move to the visible **Agent can / Human only** card and approval cards.

Narration:

“This is the core design decision. The agent can inspect, improve and prepare consequential work, but approval is not a WebMCP tool. ConsentOS deliberately keeps deletion and session termination under human authority.”

Press **Approve** on the queued destructive actions yourself.

Open the Audit log briefly and show that those decisions are attributed to **human**.

## 2:10–2:35 — Verify the outcome

Have the agent call `get_privacy_state` or `get_privacy_score_details` again.

Show **96/100** and **0 avoidable risks**.

Narration:

“The agent can now verify the result through the same structured page interface. The score is deterministic, not decorative: the demo starts at 54 with eight avoidable risks and ends at 96 with zero.”

## 2:35–2:55 — Close

Show the score instrument, exposure map and activity timeline together.

Narration:

“ConsentOS demonstrates a simple principle for agent-native websites: WebMCP defines capability; the human retains authority.”

End before 3:00.

## Recording rules

- Record actual WebMCP tool execution; do not fake calls with manual clicks.
- Human clicks should be used only for the intentionally human-only approval step and normal demo navigation.
- Keep browser zoom high enough for judges to read tool status, score changes and activity entries.
- If the agent takes an unexpected path, reset rather than cutting together a misleading sequence.
