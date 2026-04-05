# Command Design — Telegram OpenAI Connect Helper

This document explains `/openai` helper command design to keep UX simple, implementation safe, and behavior deterministic.

---

## Design Goals
- reduce OpenAI reconnect friction,
- keep command additive to native commands,
- keep state and error handling explicit,
- avoid dependency on reasoning agent for command execution.

---

## Design Principles
- **Complement**, not replacement of native commands
- **Additive only**
- **Low-friction UX**
- **Safe by default**
- **State-aware**
- **Minimal sensitive data exposure**

---

## Command Surface
### Primary entry
```text
/openai
```
Default behavior options:
- show helper usage, or
- alias to `/openai connect` for simplest UX.

### Recommended subcommands
```text
/openai connect device-auth
/openai connect url
/openai paste <redirect_url>
/openai status
/openai use <model>
/openai cancel
```

---

## Per-command Design

## 1) `/openai connect device-auth` and `/openai connect url`
### Goal
Start OAuth flow.

### Behavior
- check whether auth is already active,
- `device-auth` mode: return verification URL + one-time code,
- `url` mode: return OAuth callback URL flow,
- store state with mode,
- return short next-step instructions.

### Remote/headless fallback
- run `codex login --device-auth`,
- direct user to `https://auth.openai.com/codex/device`,
- complete verification,
- then sync credentials back into OpenClaw.

### Guardrails
- never expose raw token,
- never create pending state without expiry.

---

## 2) `/openai paste <redirect_url>`
### Goal
Submit callback URL from browser flow.

### Behavior
- only allowed when state is waiting for callback,
- strict callback URL validation,
- mask sensitive params in logs,
- exchange auth code,
- move state to success/failure.

### Live behavior note
- `paste` is only relevant for `connect url`.
- if flow is `device-auth`, return guidance that paste is unnecessary.

### Guardrails
- reject when state is invalid,
- never echo code/token to chat,
- reject non-matching callback URL patterns.

---

## 3) `/openai status`
### Goal
Show current flow/auth status.

### Behavior
- display whether auth is active,
- show pending/failed/expired flow state,
- include next-step hint where useful.

---

## 4) `/openai use <model>`
### Goal
Set model after auth is active.

### Behavior
- reject when auth is not active,
- validate model string,
- set model and return clear confirmation/error.

---

## 5) `/openai cancel`
### Goal
Cancel ongoing flow.

### Behavior
- stop pending worker if running,
- mark state as `cancelled`,
- cleanup temporary pending state.

---

## State Model
Minimum states:
- `idle`
- `oauth_started`
- `awaiting_callback_url`
- `callback_received`
- `auth_completed`
- `auth_failed`
- `cancelled`
- `expired`

Rules:
- explicit transitions,
- explicit triggers,
- TTL on pending states.

---

## UX Copy Guidelines
- short and direct instructions,
- actionable errors with clear next step,
- concise success message with what to do next.

Examples:
- success connect: login URL/device code + next command,
- failed callback: ask user to restart connect,
- status: show connected vs pending clearly.

---

## Error Handling Coverage
Must handle:
- missing args,
- malformed callback,
- expired flow,
- provider exchange/auth failure,
- duplicate connect while pending,
- invalid model,
- user cancellation,
- device code authorization not enabled.

### Known blocker guidance
When browser callback loops and localhost callback is unusable:
1. enable `Enable device code authorization for Codex` in ChatGPT Security Settings,
2. rerun `codex login --device-auth`,
3. continue with device verification.

---

## Security Design
- strict callback validation,
- token/code never echoed to chat,
- sensitive parameters masked in logs,
- no unnecessary data persistence,
- pending state has TTL and safe cleanup.

---

## Compatibility Design
To stay safe with native Telegram commands:
- add helper via `customCommands`,
- do not change `commands.native`,
- do not change `commands.nativeSkills`,
- never replace full command set.

---

## Recommended Roadmap
### Phase 1 (implemented)
- connect device-auth/url,
- paste,
- status,
- use,
- cancel.

### Phase 2 (hardening)
- edge-case handling,
- timeout/state cleanup audit.

### Phase 3 (UX + observability)
- better operational telemetry,
- command-level test harness for each connect mode.

---

## Conclusion
The command design should optimize for:
1. practical user workflow,
2. native command safety,
3. stable state/error behavior.

If these three are preserved, `/openai` becomes a reliable operational helper without making Telegram integration fragile.
