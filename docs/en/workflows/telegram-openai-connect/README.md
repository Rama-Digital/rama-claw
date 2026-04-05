# OpenAI Connect Flow in Telegram (Safe for Built-in Commands)

## Goal
This document describes how to add an OpenAI OAuth flow through Telegram without breaking existing built-in commands.

Primary targets:
- built-in commands remain safe,
- new command is additive,
- `/openai` runs deterministically via a native command handler (not reasoning-based).

---

## Core Principles
To keep built-in commands safe, implementation should follow:
- **additive only**
- **no override**
- **no command wipe**

Meaning:
- do not disable native commands,
- do not replace the whole command menu,
- do not use any approach that empties existing commands.

---

## Flow Summary
Main operational flow:
1. User types `/openai connect device-auth` or `/openai connect url`.
2. System starts auth flow based on selected mode.
3. User completes verification.
4. User checks `/openai status`.
5. User optionally chooses a model with `/openai use <model>`.

Fallback flow for remote/headless environments:
1. If localhost callback loops or fails redirect, do not force browser callback.
2. Switch to device auth using `codex login --device-auth`.
3. User opens `https://auth.openai.com/codex/device` and enters the one-time code.
4. After success, OpenClaw can sync OAuth profile from Codex CLI.

---

## Additive Command Example

```json
{
  "channels": {
    "telegram": {
      "customCommands": [
        {
          "command": "openai",
          "description": "OpenAI helper: connect device-auth | connect url | status | cancel"
        }
      ]
    }
  }
}
```

Notes:
- this only adds `/openai` into the Telegram menu,
- built-in commands stay enabled,
- configuration remains additive.

---

## Current Live Behavior
Implemented command set:
- `/openai connect device-auth`
- `/openai connect url`
- `/openai paste <url>` (for `connect url` mode)
- `/openai status`
- `/openai use <model>`
- `/openai cancel`

All commands above are handled deterministically via native command handler plugin.

---

## Recommended State Machine
Use explicit state tracking per user/session.

### States
- `idle`
- `oauth_started`
- `awaiting_callback_url`
- `callback_received`
- `auth_completed`
- `auth_failed`
- `cancelled`
- `expired`

### Why this matters
- callback does not land in the wrong flow,
- restart/timeout handling is clearer,
- debugging becomes easier.

---

## Timeout / Expiry Rules
Recommended:
- pending OAuth should expire in **10–15 minutes**,
- callback submitted after expiry must be rejected,
- expired/cancelled state should be cleaned safely.

---

## Error Cases That Must Be Handled
At minimum:
- callback submitted without active connect flow,
- invalid callback URL,
- expired callback,
- user-cancelled login,
- auth provider failure,
- user already has active auth,
- restart during flow,
- **device code authorization not enabled** in ChatGPT Security Settings.

### Common field blocker
Real-world case to support:
- user clicks Continue, but browser flow loops and localhost callback never lands.

Fix path:
1. Click red link: `Enable device code authorization for Codex`.
2. Open ChatGPT Security Settings.
3. Enable: `Enable device code authorization for Codex`.
4. Rerun: `codex login --device-auth`.
5. Complete device flow at `https://auth.openai.com/codex/device`.

---

## Success Criteria
Flow is successful if:
- OAuth exchange succeeds,
- credentials are stored,
- auth status can be verified,
- user receives confirmation auth is active,
- optional model selection succeeds.

---

## Post-Auth Steps
Recommended sequence:
1. verify auth success,
2. check auth status,
3. optionally set model,
4. send success confirmation,
5. clean temporary state.

---

## Test Checklist (Short)
- `/openai` appears in custom Telegram menu,
- built-in commands remain visible,
- both connect modes work,
- valid callback accepted,
- invalid/expired callback rejected safely,
- device-auth path works when browser callback fails,
- cancel clears pending state,
- restart does not break command menu,
- auth status remains visible.

---

## Rollback
If you want to remove the custom helper:
1. remove `openai` entry from `customCommands`,
2. restart gateway safely,
3. verify built-in commands stay intact,
4. ensure no side effects in Telegram command menu.

---

## Security Notes
- never expose raw tokens in chat,
- mask sensitive callback parameters in logs,
- validate callback URL strictly,
- avoid storing unnecessary data.

---

## Conclusion
This workflow is suitable if you want Telegram-based OAuth reconnect **without sacrificing built-in command safety**.

The keys are:
- additive config,
- explicit state,
- expiry rules,
- strict error handling,
- disciplined testing.
