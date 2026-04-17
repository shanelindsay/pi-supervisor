---
name: pi-supervisor
description: Use pi-supervisor to supervise the current Pi session with a short goal. Explains how /supervise works, how SUPERVISOR.md fits in, and how to write strong single-goal or multi-step goals.
---

# Pi-Supervisor Skill

Use this package when you want a supervisor to watch the current Pi session and
steer it toward a clear goal.

## Core idea

- `SUPERVISOR.md` tells the supervisor **how** to supervise
- the goal tells the supervisor **what this run is trying to achieve**
- the goal is appended again on each supervisor check, so it should be short,
  high-signal, and able to stand on its own
- the goal can be a single line or a longer block of text using paragraphs,
  bullets, numbered lists, or lightweight markdown-style headings

Most of the time, start with a short inline goal. For more complex cases, ask
the agent to start supervision with a longer structured goal.

## Using pi-supervisor

### Start with inline text

Start supervision with inline text:

```text
/supervise Fix the hook runtime and keep the change minimal
```

### Start through the agent

You can also ask the agent to start supervision itself when the goal needs a bit
more structure.

For example, ask it to start supervision with a goal like:

```text
Goal:
Fix the hook runtime.

Priorities:
- Fix startup reliability
- Stop stale widget state
- Prove one clean supervision cycle

Constraints:
- Keep the change minimal
- Do not add new workflow machinery
```

### Other useful commands

Other useful commands:

```text
/supervise status
/supervise stop
/supervise model
/supervise widget
/supervise sensitivity medium
```

## Understanding SUPERVISOR.md

`SUPERVISOR.md` is the supervision behaviour prompt.

There is always one effective supervisor prompt for a run. The package resolves
it in this order:

1. `.pi/SUPERVISOR.md` in the current project
2. `~/.pi/agent/SUPERVISOR.md` as a personal global default
3. the built-in default supervisor prompt shipped with the package

So `SUPERVISOR.md` is not an extra prompt layered on top of some other default.
If a project or global `SUPERVISOR.md` exists, that becomes the supervisor
prompt. Otherwise the built-in default is used.

Use it to shape how the supervisor reasons, for example:

- what quality bar matters most
- how strict the supervisor should be about completion
- which kinds of drift deserve steering

The run goal is separate. The goal says what this run is trying to achieve.

The goal can be a single sentence or a longer block of text. Markdown-style
structure is fine, but it is still just goal text. What matters is that it
gives the supervisor enough context to judge progress.

### When to customise `SUPERVISOR.md`

Good reasons to customise it:

- you want a stricter or looser bar for calling work done
- you want the supervisor to care more about tests, simplicity, or minimal diffs
- you want to change how proactive steering feels
- you want to define what kinds of drift matter most in this project

Keep run-specific instructions in the goal instead.

## What context the supervisor sees

On each supervisor check, the supervisor does not read the whole session from
scratch. It gets a focused context bundle made of:

- the effective supervisor prompt from `SUPERVISOR.md` or the built-in default
- the active goal text
- a recent conversation window
- the most recent compaction or branch summary, if Pi has created one
- recent supervisor interventions
- whether the agent is idle or still working

### Recent conversation window

In the current implementation, the supervisor sees only the last few
user/assistant messages from the session branch:

- `low` sensitivity: last **6** messages
- `medium` sensitivity: last **12** messages
- `high` sensitivity: last **20** messages

This is a recent message window, not the full transcript.

### Compaction summary

If Pi has already created a `compaction` or `branch_summary` entry for older
history, the supervisor also gets the most recent summary text from that entry.

That summary gives the supervisor compressed older context that sits outside the
recent message window.

### What this means at the start

On the first supervision loop, the amount of prior conversation context is
variable.

- if supervision starts after the session already has some history, the
  supervisor can see the recent slice of that history
- if supervision starts early, there may be very little useful prior context
- if no compaction or branch summary exists yet, there is no older-history
  summary to include

That is why the goal should usually stand on its own. It is not just a label.
It often carries much of the initial context the supervisor needs.

## How to write a goal

Keep the goal readable and concrete.

Think of the goal as a short run brief.

The active goal is appended again on each supervisor check, so high-signal and
compact works better than long project-document style input.

In general, keep it short unless extra detail is truly necessary.

Good goals usually make these things easy to see:

- the objective
- any important constraints
- what a successful result should accomplish

The goal does not need a rigid schema.

Most of the time, inline text is enough.

If the agent starts supervision itself through the tool, it can supply the goal
text on the fly. For more complex goals, paragraphs, bullet points, numbered
lists, and markdown-style headings are all fine.

### Common goal forms

#### 1. Inline goal

```text
Fix the hook runtime, keep the change small, and leave the supervisor loop working cleanly.
```

#### 2. Longer structured goal text

```text
Fix the hook runtime while keeping the change minimal and leaving the supervisor loop working cleanly.

Priorities:
- Fix startup reliability.
- Stop stale widget state.
- Prove one clean supervision cycle.

Constraints:
- Keep the change minimal.
- Do not add new workflow machinery.
```

For multi-step work, plain paragraphs, bullets, numbered lists, and lightweight
markdown-style structure are enough. The runtime does not
need a formal schema to use the goal well.

## Goal opening line and truncation

The widget shows the start of the active goal.

In the current implementation, the goal text itself is shortened to 48
characters before it is placed into the first widget line. If the whole widget
line is still too wide for the terminal, the displayed line may be shortened
again by the TUI.

That means the opening line matters.

- for inline goals, put the clearest version of the goal first
- for longer structured goals, make the first line a short one-line goal
- if the goal has more detail, put it after that first line

Suggested pattern for longer goals:

```text
Fix the hook runtime while keeping the change minimal and leaving the supervisor loop working cleanly.

Priorities:
- ...
```
