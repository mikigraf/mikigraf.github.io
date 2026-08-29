---
layout: post
title: "The hard part of AI agents is not the demo"
description: "Most agent demos optimize for surprise. Production systems optimize for recoverability, observability, and boring failure modes."
date: 2026-07-02
tags:
  - ai-agents
  - production-ai
  - reliability
---

A demo agent looks finished because the happy path is short. It calls a tool, prints an answer, and stops. A production run lives long enough to fail in the middle of a side effect, then try again.

The naive loop looks like this:

```python
while not done:
    action = model.decide(history)
    result = tools.run(action)  # may have already succeeded
    history.append(result)
```

If `tools.run` already created a ticket, a retry creates a second one. The model does not remember the HTTP 201. It remembers a timeout.

> Agents need failure recovery before they need autonomy.

Autonomy without recovery just automates the mess. I would rather a run that can stop, retry, or hand back to a human than one that keeps going with half-applied work.

## What production runs need

- Idempotent tools, so a retry does not double-charge or double-write.
- Explicit state, stored outside the model, so a crash does not invent a new history.
- Budgets and timeouts, so a loop cannot spend the night.
- A kill switch a human can hit without asking the agent politely.
- Full traces of what was attempted, not a summary the model wrote after the fact.

State belongs in a file the model does not own:

```json
{
  "run_id": "run_08f3",
  "step": 4,
  "budget_usd": 2.50,
  "spent_usd": 0.81,
  "kill": false,
  "effects": {
    "create_ticket:idem_9c2": {
      "status": "committed",
      "ticket_id": "JIRA-1841"
    }
  }
}
```

The tool then keys off that id, not the model's confidence:

```python
def create_ticket(ctx, title: str, idem: str) -> dict:
    prior = ctx.effects.get(f"create_ticket:{idem}")
    if prior and prior["status"] == "committed":
        return prior  # retry is a no-op

    ticket = jira.create(title, unique=idem)
    ctx.commit(f"create_ticket:{idem}", {
        "status": "committed",
        "ticket_id": ticket.id,
    })
    return ctx.effects[f"create_ticket:{idem}"]
```

If `ctx.kill` is set, the loop exits before the next tool call. The trace is the `effects` map plus the raw tool I/O, not a paragraph the model wrote at the end.
