---
name: costrel
description: Discipline for handing a bounded sub-task to a more expensive "consultant" model mid-task — manually triggered, not a router or auto-escalation. Use when you hit a decision point where either the cost of a wrong verdict is asymmetric or hard to walk back (a security-critical claim, an auth-bypass verification, something that ships if you're wrong), or the task doesn't decompose and needs one mind holding the whole span at once (a stalled or plateaued project, "what's next" after your own incremental moves stopped working). Not for routine work a cheap model already handles fine. Builds a dense six-field brief so the expensive call spends its budget reasoning instead of re-reading files. Triggers on requests like "get a second opinion from a bigger model", "consult opus/fable on this", "this needs an expensive model's judgment before we commit", "I want to hand this off before deciding", or when a project feels stuck and more of the same isn't going to break it.
---

# costrel

A costrel is a small vessel carried out to someone working at a distance,
filled with what they need in advance. This is the same shape: package a
bounded question densely enough that a closed-book consultant model can
answer it without needing to fetch anything.

## Decide whether this is a costrel moment

Two trigger shapes. Don't fire on a vibe that something is "probably worth
a consult" — fire on one of these:

- **Stakes.** The cost of a wrong verdict is asymmetric or hard to walk back.
  Brief is narrow and deep: one claim, the citations that ground it, a
  bounded question. Example: verifying an auth-bypass fix is actually closed,
  handed to a consultant rather than eyeballed inline, because a wrong call
  ships a security regression. No real run through the template has hit this
  shape yet — the example above predates the template.
- **Range.** The task doesn't decompose — a fan-out of narrow cheap moves
  can't reconstruct what one mind holding the whole span at once would catch.
  Brief is wide and compressed — it holds the whole span rather than
  narrowing to a claim. Most common real trigger: a project feels stalled or
  plateaued and the ask is "what's next" — not because a wrong answer is
  costly, but because your own local, incremental moves are what produced
  the plateau, and more of the same won't break it. The stall itself is
  diagnostic; preserve exactly what stalled and where the resistance was,
  don't smooth it into a status summary. Confirmed by a real run: a sibling
  project's ten-stage pipeline collapsed when only one stage's model tier
  changed, and the answer depended on holding the whole pipeline in view at
  once rather than checking any single stage (`CHANGELOG.md`).

If neither fits, don't reach for this — answer it yourself or fan it out to
cheap parallel moves instead. A third candidate shape was considered after
an early run didn't land in either bucket; on reflection that run was fully
checkable afterward with a few direct tests, which means it decomposed and
never needed a dispatch in the first place — see `CHANGELOG.md` for the
retraction rather than carrying a shape here that turned out not to exist.

## On trigger

Don't ask the user to supply all six fields up front, and don't wait for a
command-line argument that spells out the whole brief — assemble Frame,
State, Trajectory, Friction point, and Exclusions from what's already in the
conversation, since a real costrel moment fires mid-task, when most of that
is already sitting in context. Ask the user only for what genuinely isn't
derivable, which is usually just the Open Question itself — only they know
what they actually want to ask. If an argument was given (`/costrel <text>`),
treat it as a draft Open Question to refine, not the whole brief. If too
little context exists to fill State or Trajectory with real facts, say so
and ask directly rather than guessing — a thin brief built on assumptions
defeats the point as surely as an unverified one does.

Show the assembled brief before dispatching it, and wait for a go-ahead.
A consult is a real, non-trivial cost against an expensive model. Firing it
on your own judgment about a stakes or range moment is one thing; spending
the user's money without letting them see the actual brief first is
another. "Manual trigger" means the human approves what's about to be sent — a
higher bar than a human merely being present when the skill fired.

## The brief (six fields, plus two hard requirements)

Write these as labeled sections, not narrative prose the consultant has to
parse for facts.

1. **Frame** — one line: the actual decision on the table, not a restatement
   of the situation.
2. **State** — artifact-level: paths, commits, numbers, named candidates.
   **Verify every fact here that's cheaply checkable locally before you
   dispatch** — a flag's real behavior, a schema's actual fields, an enum's
   actual values. The consultant is closed-book and cannot check anything
   itself; an unresolved checkable fact doesn't save you the check, it moves
   the cost onto the expensive call, which then has to hedge around it
   instead of answering. This was the single largest gap in the first real
   pressure test — three facts left unverified in State became three things
   the consultant had to flag and partially guess around. For a consult
   shaped around an eval or scoring harness specifically, state what the
   scorer actually observes — the final rendered answer, or the full
   pipeline trace — since that single fact decides between two different
   failure mechanisms with two different fixes. A second real consult (see
   `CHANGELOG.md`) omitted it, the consultant flagged it as unknown, and it
   changed which mechanism it could name with confidence. **Label each bullet
   verified, inferred, or recalled** — a fact you checked directly, a
   read on something you didn't check, or something you're stating from
   memory rather than looking up. The consultant can only weigh a claim
   correctly if it knows which kind it's looking at; an unlabeled brief
   reads as if every line were checked.
3. **Trajectory** — the sequence of moves already tried, each with what it
   actually changed. Mostly free: compression of reading you already did for
   your own work, not new research.
4. **Friction point** — where it stalled, as an observed fact ("X stopped
   producing new Y"), never as a diagnosis. Diagnosing it yourself and asking
   the consultant to confirm launders your own assumption through their
   better prose instead of getting independent judgment.
5. **Exclusions** — what's already been tried and ruled out, and why. Without
   this, the likely failure is the consultant re-proposing something already
   rejected — paying consultant prices to regenerate a path already walked.
   Leave room for the consultant to name something outside the list if it has
   a real reason to (this worked correctly in the first pressure test — don't
   close it off entirely).
6. **Open question** — bounded in scope, open in answer. Too open and the
   consultant spends its own budget re-scoping the problem before it can
   reason about it. Too narrow or leading and the answer is just your own
   prior read back in better prose.

**Two hard requirements on top of the six fields**, both discovered by
hitting the gap in a real consult rather than reasoned out in advance:

- End the brief with an instruction that the consultant's **first line must
  be `Answering as: <model>`**. A requested model override can fail silently
  and fall back to a different model with no error — confirmed for the
  `fork` subagent type, and the safest assumption elsewhere until proven
  otherwise. This turns a silent failure into a visible one for the cost of
  one line.
- Instruct the consultant: **if you need information this brief doesn't
  supply to answer with real confidence, say exactly what's missing before
  you answer, rather than guessing past the gap.** This is what makes a
  genuinely closed-book run legible — a consultant that hedges correctly on a
  real gap is giving you a better signal than one that fills it with a
  plausible guess.

## Dispatching it

Use the `costrel-consultant` agent type (`~/.claude/agents/costrel-consultant.md`)
via a non-fork `Agent()` call, picking the actual model with the **per-invocation
`model` parameter** — not the agent-type file's own `model:` field, which is
just a static fallback and only accepts `sonnet`/`opus`/`haiku`/`inherit`
anyway. The per-invocation parameter wins first in the resolution order and
is what actually selects the consultant tier:

```
Agent(subagent_type: "costrel-consultant", model: "opus", prompt: <the brief>)
```

Confirmed by direct use, not assumed:
- Non-fork `Agent()` calls are async — they return immediately and notify on
  completion. The orchestrating turn is never blocked waiting on the
  consultant. `fork` cannot serve as the consultant regardless: it always
  inherits the parent session's own model, ignoring any override.
- The subagent starts fresh (no inherited conversation), but **does** inherit
  CLAUDE.md files — documented, and there's no agent-type field to opt out —
  plus whatever a session's plugins inject (undocumented, but a spawned
  consultant has independently surfaced injected instructions from its own
  context on more than one occasion). Some of that injected material is a
  direct instruction a zero-tool agent cannot satisfy. There's no structural
  fix available — the platform doesn't offer isolation — so
  `costrel-consultant.md`'s own system prompt carries an explicit line
  telling the consultant that inherited instructions assuming tools, files,
  or a coding task don't apply to it. "The brief and nothing else" was never
  literally true; treat that line as the actual enforcement, not the prompt
  alone.

Verified by direct test: `tools: []` in the agent-type file grants truly
zero tools — a spawned instance reported none and confirmed it couldn't
read a file, run a command, or fetch a URL. `~/.claude/agents/` (and
`.claude/agents/`) is watched and a new or edited file is picked up within
seconds, no restart needed — the one exception is the *first* file dropped
into a directory that didn't exist yet at session start, which needs a
session boundary once, the first time only.

## Reporting back

If a consult's outcome is worth sharing — with whoever built this
discipline, or logging it anywhere the taxonomy might later depend on —
paste the brief verbatim alongside what happened, not a summary of it. A
summary loses the exact shape of what was actually asked, which is the
thing a later reader needs to check which trigger shape the consult really
was. The first external report this discipline received couldn't be
classified from its summary alone; only the verbatim brief, requested
afterward, settled it.

## Full history and open questions

This repo's `CHANGELOG.md` has the full design history, prior art
comparisons (opusplan, Anthropic's Research architecture, RouteLLM/FrugalGPT,
crystal, freshet, hybrid-loops, check-plan), and the complete pressure-test
writeup this skill was built from. Read it before changing the taxonomy or
the brief template.
