---
layout: default
title: The hard part of AI agents is not the demo
date: 2026-07-02
excerpt: Demos hide the recovery path. Production needs idempotent tools, explicit state, budgets, a kill switch, and traces.
---

<p class="back"><a href="{{ '/' | relative_url }}">&lt;- Posts</a></p>
<p class="date">2 Jul 2026</p>

<article>
<h1>{{ page.title }}</h1>

A demo agent looks finished because the happy path is short. It calls a tool, prints an answer, and stops. A production run lives long enough to fail in the middle of a side effect, then try again.

That is the gap I keep hitting in real agent systems. The model is not the scarce part. The scarce part is what happens after the first tool call goes wrong.

> Agents need failure recovery before they need autonomy.

Autonomy without recovery just automates the mess. I would rather a run that can stop, retry, or hand back to a human than one that keeps going with half-applied work.

## What production runs need

- Idempotent tools, so a retry does not double-charge or double-write.
- Explicit state, stored outside the model, so a crash does not invent a new history.
- Budgets and timeouts, so a loop cannot spend the night.
- A kill switch a human can hit without asking the agent politely.
- Full traces of what was attempted, not a summary the model wrote after the fact.

This post is layout sample copy for the blog. The first real artifact post is ctxlane.
</article>
