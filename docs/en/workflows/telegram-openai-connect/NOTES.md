# Why I Built OpenAI Connect Flow in Telegram

The reason is simple:

**I got tired of SSH-ing into VPS just to reconnect OpenAI.**

If every auth reconnect requires opening VPS shell first, daily operations become heavy and inefficient.

So this flow was built to handle reconnect directly from Telegram while keeping built-in command behavior safe.

## Problem Being Solved
Before this flow, reconnect usually looked like:
1. login to VPS,
2. open correct environment/shell,
3. run manual auth flow,
4. collect login URL,
5. paste callback,
6. re-check status.

This is manageable once in a while, but painful when repeated.

## Recent Field Notes (Important)
In real VPS usage, we saw a common blocker:
- browser OAuth loops after pressing Continue,
- `localhost:1455` callback is sometimes unusable on user side.

A working fix path:
1. Enable ChatGPT account option:
   `Enable device code authorization for Codex`
   (in ChatGPT Security Settings).
2. Run:
   `codex login --device-auth`
3. Verify at:
   `https://auth.openai.com/codex/device`
4. After success, OpenClaw can use synced OAuth credentials.

## Additional Field Note: `deactivated_workspace` + Session Stickiness
Another recurring production issue:
- `deactivated_workspace` appears repeatedly on some group/cron sessions.
- root cause is often not gateway outage, but old session stickiness to an invalid OAuth auth profile.

Recovery pattern that worked:
1. set auth profile priority (`auth.order`) to prefer a valid profile,
2. clear stale `authProfileOverride` pin in session store,
3. restart gateway and verify both group and cron sessions.

Operational note:
- direct chat may look healthy because it already moved to a valid profile,
- while group/cron can still fail if they remain sticky to old profile pin.

## Flow Goal
Move high-frequency reconnect tasks into Telegram:
- start auth from chat,
- open login URL,
- submit callback,
- check status,
- continue with model usage.

Live implementation status:
- `/openai` is handled by deterministic native command handler plugin,
- no longer dependent on reasoning-agent routing for connect/status/use/cancel.

## Why Not Replace Built-in Commands?
Because OpenClaw built-ins are still important and should not be broken by helper features.

Required principle:
- additive only,
- no override,
- no command wipe.

So `/openai` is an extension, not a replacement.

## Desired Outcome
If this flow stays mature and safe, OpenAI reconnect becomes:
- more practical,
- faster,
- Telegram-first,
- no constant VPS hop for auth refresh.

## In Short

> **OpenAI reconnect should not be as heavy as SSH-ing to VPS every time auth needs refresh.**

If it can be done safely from Telegram, operations become much smoother.

## Upstream Feature Request
Public feature request for this workflow:
- https://github.com/openclaw/openclaw/issues/61205

Useful as reference if this workflow becomes a native OpenClaw feature in the future.
