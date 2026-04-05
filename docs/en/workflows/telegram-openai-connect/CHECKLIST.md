# Implementation Checklist — Telegram OpenAI Connect Flow

Use this checklist to keep `/openai` implementation safe, additive, and production-ready.

---

## 1. Pre-Implementation
- [ ] Confirm the objective: reconnect OpenAI without SSH-ing into VPS every time.
- [ ] Keep helper command as a **complement**, not a native-command replacement.
- [ ] Read guardrails before touching config.
- [ ] Verify native Telegram commands are currently visible.
- [ ] Verify target environment uses native Telegram command menu.

---

## 2. Config Safety
- [ ] **Do not** set `commands.native=false`.
- [ ] **Do not** set `channels.telegram.commands.native=false`.
- [ ] **Do not** set `commands.nativeSkills=false` unless explicitly requested.
- [ ] Use **additive-only** changes.
- [ ] Add helper command via `channels.telegram.customCommands`.
- [ ] Ensure no config path can wipe commands (`setMyCommands([])`).

---

## 3. Command Menu
- [ ] `/openai` appears in Telegram custom menu.
- [ ] Built-in commands remain visible.
- [ ] No native command disappears after patch.
- [ ] Description is clear enough for users.

Target commands:
- `/openai connect device-auth`
- `/openai connect url`
- `/openai paste <url>`
- `/openai status`
- `/openai use <model>`
- `/openai cancel`

---

## 4. Flow Validation
### 4.1 Connect
- [ ] `connect device-auth` starts device auth flow.
- [ ] `connect url` starts browser callback flow.
- [ ] OAuth URL is generated and delivered.
- [ ] User gets clear login instruction.
- [ ] Device-auth fallback is available when localhost callback fails.

### 4.2 Callback
- [ ] Valid callback URL is accepted.
- [ ] Invalid callback URL is rejected safely.
- [ ] Expired callback URL is rejected with retry instruction.
- [ ] Raw token is never posted in chat.

### 4.3 Status
- [ ] `/openai status` returns correct auth state.
- [ ] User can clearly see connected/failed/pending state.

### 4.4 Use Model
- [ ] `/openai use <model>` only works after auth is active.
- [ ] Invalid model selection is handled safely.
- [ ] Model set result can be verified.

---

## 5. State Handling
- [ ] State is stored per user/session.
- [ ] Minimum states exist: `idle`, `oauth_started`, `awaiting_callback_url`, `auth_completed`, `auth_failed`, `cancelled`, `expired`.
- [ ] State is cleaned on success.
- [ ] State is cleaned on cancel.
- [ ] State is cleaned on timeout/expiry.

---

## 6. Timeout / Expiry
- [ ] Pending OAuth has explicit TTL (recommended 10–15 min).
- [ ] Callback after expiry is not treated as valid auth.
- [ ] User is guided to restart flow.

---

## 7. Error Handling
- [ ] Callback without active connect flow is rejected safely.
- [ ] Malformed callback is rejected safely.
- [ ] Provider auth failure moves flow to `auth_failed`.
- [ ] User cancel moves flow to `cancelled`.
- [ ] Mid-flow restart keeps state consistent or invalidates safely.
- [ ] Missing `Enable device code authorization for Codex` is handled with clear instructions.

---

## 8. Security
- [ ] Raw tokens never appear in chat.
- [ ] Sensitive callback params are masked in logs.
- [ ] No unnecessary data is stored.
- [ ] Sensitive logs are not leaked to users.
- [ ] Callback URL is strictly validated before exchange.

---

## 9. Testing
### Functional
- [ ] `/openai` starts flow.
- [ ] `/openai status` works.
- [ ] `/openai use <model>` works after auth.
- [ ] Valid callback succeeds.
- [ ] Invalid callback fails safely.
- [ ] Device-auth succeeds when browser callback is unusable.

### Regression
- [ ] Native Telegram commands remain present.
- [ ] Helper command does not wipe existing menu.
- [ ] Gateway restart does not break command menu.
- [ ] Gateway restart does not leave ambiguous state.

### UX
- [ ] Instructions are short and clear.
- [ ] Errors are actionable.
- [ ] Success message includes next step.

---

## 10. Rollback
- [ ] Remove `openai` from `channels.telegram.customCommands`.
- [ ] Restart gateway safely.
- [ ] Verify native command set remains intact.
- [ ] Verify `/openai` disappears from custom menu.
- [ ] Verify no side effects.

---

## 11. Done Criteria
Implementation is done when:
- [ ] `/openai` helper is active,
- [ ] native commands remain safe,
- [ ] connect flow is usable,
- [ ] callback flow is safe,
- [ ] auth status is visible,
- [ ] state machine is stable,
- [ ] timeout + rollback tested,
- [ ] no sensitive data leaked in chat/logs.
