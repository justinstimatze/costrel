# costrel

A costrel is a small vessel carried out to someone working at a distance,
filled in advance with what they'll need. costrel is a discipline for the
same shape of problem: when a task hits a decision point that calls for a
more expensive model's judgment, package the question densely enough that
the expensive call spends its budget reasoning, not re-reading files a
cheaper model already read.

You decide by hand when to reach for this, every time — nothing here
triggers it for you. The actual product is the request package itself.

## When to reach for it

Two shapes:

- **Stakes.** The cost of a wrong verdict is asymmetric or hard to walk
  back — a security-critical claim, something that ships if you're wrong.
  Brief is narrow and deep: one claim, the citations that ground it, a
  bounded question. No real run through the template has hit this shape
  yet — the example above predates it.
- **Range.** The task doesn't decompose — a fan-out of narrow cheap moves
  can't reconstruct what one mind holding the whole span at once would
  catch. Most common real trigger: a project feels stalled and the ask is
  "what's next," not because a wrong answer is costly, but because your own
  incremental moves produced the plateau and more of the same won't break
  it. Brief is wide and compressed — it holds the whole span rather than
  narrowing to a claim. Confirmed by a real run: a sibling project's
  ten-stage pipeline collapsed when only one stage's model tier changed,
  and the answer depended on holding the whole pipeline in view at once
  (below).

If neither fits, this isn't the tool — answer it yourself, or fan it out
to cheap parallel moves instead. A third candidate shape was considered
after an early run didn't land in either bucket; it turned out to be fully
checkable afterward with a few direct tests, which means it decomposed and
never needed a dispatch at all — see `CHANGELOG.md` for the retraction.

## The brief

Six fields, written as labeled sections, not narrative prose the consultant
has to parse for facts:

1. **Frame** — one line: the actual decision on the table.
2. **State** — artifact-level facts: paths, commits, numbers, named
   candidates. Verify every fact here that's cheaply checkable locally
   *before* you dispatch. The consultant is closed-book and can't check
   anything itself — an unresolved checkable fact doesn't save you the
   check, it moves the cost onto the expensive call, which then has to
   hedge around it instead of answering. For an eval- or scoring-harness-shaped
   consult, state what the scorer actually sees — the final answer, or the
   full pipeline trace. A second real consult found this single omission
   mattered as much as the metric itself (below). **Label each bullet
   verified, inferred, or recalled** — checked directly, read without
   checking, or stated from memory. The consultant can only weigh a claim
   correctly if it knows which kind it's looking at.
3. **Trajectory** — the sequence of moves already tried, each with what it
   actually changed. Mostly free: compression of reading you already did.
4. **Friction point** — where it stalled, as an observed fact, never a
   diagnosis. Diagnosing it yourself and asking the consultant to confirm
   launders your own assumption through their prose instead of getting
   independent judgment.
5. **Exclusions** — what's already been tried and ruled out, and why.
   Without this, the likely failure is the consultant re-proposing
   something already rejected. Leave room for it to name something outside
   the list if it has a real reason to.
6. **Open question** — bounded in scope, open in answer. Too open and the
   consultant spends its own budget re-scoping the problem. Too narrow and
   the answer is just your own prior read back in better prose.

Plus two hard requirements, both found by hitting the gap in a real run
rather than reasoned out in advance:

- End the brief requiring the consultant's **first line** to be
  `Answering as: <model>`. A requested model override can fail silently —
  confirmed for one dispatch path, worth assuming elsewhere until proven
  otherwise. This turns a silent failure into a visible one for the cost of
  one line.
- Instruct it: **if you need information this brief doesn't supply to
  answer with real confidence, say exactly what's missing before you
  answer, rather than guessing past the gap.** A consultant that hedges
  correctly on a real gap is a better signal than one that fills it with a
  plausible guess.

## A real run

The first run of this discipline picked the nearest genuinely unresolved
decision — which mechanism should actually dispatch the consult — rather
than a staged example. Closed-book, no prior context, told explicitly to
flag anything it needed that the brief didn't supply.

It recommended a specific mechanism over two others, correctly ruled out a
third on cost grounds (a live peer session carries its own full tool roster
and system prompt at the expensive rate — "budget meant for reasoning goes
to scaffold"), and named three facts the brief had left unverified instead
of guessing past them. Two of those three turned out checkable in minutes
by direct observation once named. That gap — checkable facts reaching the
expensive call unresolved — is what produced both hard requirements above.

The second run was external: a sibling project, `freshet`, dispatched a
real consult against a stalled quality metric ("close the Sonnet-weaver
quality gap") and reported back over the agent-to-agent messaging rail this
project already uses. The consultant derived two real confounds — a
sampling setting and a control-flow gate riding the same model setting as
the thing being measured — straight from the State section, no wasted
budget re-deriving anything. It also caught the one fact `freshet` had left
out: whether the scorer saw the final answer or the full pipeline trace.
That's the state-checklist addition above, and the first real evidence this
discipline has generated outside its own project. A later request for the
actual brief text confirmed it as genuinely range-shaped: a ten-stage
pipeline, one open question spanning all of it, resolved by noticing that
two stages downstream of the change shared the same model setting as the
metric itself.

A third run turned the discipline on itself: a consult asking what should
change in costrel's own design, dispatched to Fable with Anthropic's
current prompting guidance for that model fetched and read first rather
than assumed. It found that `costrel-consultant.md`'s own three sentences
were never the whole prompt — every subagent inherits the parent's
CLAUDE.md and this host's plugin injections, some of them instructions a
zero-tool agent can't satisfy — and that finding, checked against Claude
Code's actual documentation rather than trusted outright, is behind the
current `costrel-consultant.md` and the two extra brief-template
requirements above.

## Install

Ships as a Claude Code skill plus a dedicated closed-book agent type:

```
skills/costrel/SKILL.md            trigger checklist + brief template
agents/costrel-consultant.md       tools: [], model pinned per-call
```

**Symlink, for one machine you already have this repo on:**

```bash
ln -s /path/to/costrel/skills/costrel ~/.claude/skills/costrel
ln -s /path/to/costrel/agents/costrel-consultant.md ~/.claude/agents/costrel-consultant.md
```

Edits to the repo take effect immediately, everywhere on that machine — the
symlink makes it one copy, so there's nothing to keep in sync.

**Plugin marketplace, for every other machine:**

```
/plugin marketplace add justinstimatze/costrel
/plugin install costrel@costrel
```

`/plugin list` or `/plugin details costrel` confirms it's installed, no
cloning or manual symlinks required — a real status check instead of an ad
hoc trigger test. `plugin.json` deliberately carries no `version` field: while
this is still pre-release and iterating, the git commit SHA drives updates
instead of a number someone has to remember to bump on every push (a stale,
un-bumped version is a real documented trap — Claude Code sees the same
string and serves the cached copy, silently, even after new commits land).
Run `/plugin marketplace update costrel` to pull the latest commit; Claude
Code may also auto-check a few minutes after session start, but that default
is only confirmed for Anthropic's own marketplaces, not third-party ones —
don't rely on it here.

One more thing worth knowing if you're using *both* paths: a standalone
`~/.claude/agents/costrel-consultant.md` always overrides a plugin's bundled
copy of the same name. On this machine, the symlink from the last section
will win over the plugin install every time — which is fine, since both
point at the same repo, but it means the plugin path itself is never
actually exercised here. Testing it for real means removing the standalone
symlinks first.

## Dispatching it

Dispatch with the model chosen at the call site, not in the agent-type
file — the per-invocation model parameter takes priority over any static
default, and that's the mechanism that actually picks the consultant tier:

```
Agent(subagent_type: "costrel-consultant", model: "opus", prompt: <brief>)
```

Verified by direct use: the call is async — it returns immediately and
notifies on completion, never blocking the orchestrating turn. Verified by
direct test: the agent type reports zero tools and cannot read a file, run
a command, or fetch a URL — no tool exists for the consultant to reach for.
Structural isolation stops there, though: CLAUDE.md files reach the
consultant regardless ([documented](https://code.claude.com/docs/en/sub-agents),
no opt-out), and so does whatever a
session's plugins inject, some of it direct instructions a zero-tool agent
can't satisfy. `costrel-consultant.md`'s own system prompt is what tells it
to disregard inherited instructions that assume tools, files, or a coding
task — that line, not the platform, is doing the enforcing.

## Reporting back

If a consult is worth writing up anywhere, paste the brief verbatim
alongside the outcome, not a summary. A summary loses the exact shape of
what was asked — the thing a later reader needs to check which trigger
shape a consult actually was. The first external report this discipline
received couldn't be classified from its summary alone; only the verbatim
brief, requested afterward, settled it.

## What it's bad at, and what's not yet true

- Not a router. Nothing auto-triggers this; you decide by hand every time,
  and that's deliberate until the taxonomy above is trusted enough to hook.
- Not literal isolation. The consultant starts with no inherited
  conversation, but CLAUDE.md files and a session's plugin injections reach
  it regardless — documented for CLAUDE.md, with no agent-type field to opt
  out. "The brief and nothing else" was never literally true; what actually
  holds the line is one instruction in the consultant's own system prompt
  telling it to disregard anything inherited that doesn't apply to a
  closed-book agent.
- One external dogfood run so far, from a sibling project rather than a
  user of this repo directly. Real signal, but a sample of one.
- The taxonomy has one shape confirmed by a real, template-run consult
  (range) and one still resting on a pre-template example only (stakes).
  A third shape considered earlier was retracted: the run behind it was
  fully checkable afterward, which just means the taxonomy correctly
  flagged a decomposable question (`CHANGELOG.md`).

## Where it sits

- **[hybrid-loops](https://github.com/justinstimatze/hybrid)** — a cheap
  lens and a reasoner in a persistent loop, mutually generating each
  other's input over time. costrel is a single live handoff to a
  *different, more expensive* model, not a persistent typed-store cycle.
- **[check-plan](https://github.com/justinstimatze/plancheck)** —
  bidirectional verification of a plan through simulated execution.
  costrel's own operating thesis (trajectories over summaries) is borrowed
  from the same logic; the Trajectory field exists because a plan without
  the reads behind it is a guess.
- **[crystal](https://github.com/justinstimatze/crystal)** — moves proven
  work *down* to cheap tiers after repeated stable output. costrel moves a
  single bounded ask *up*, once, on an observed trigger — the opposite
  direction, owing the same discipline: don't fire on a vibe.
- **[opusplan](https://code.claude.com/docs/en/model-config#opusplan-model-setting)**
  (Claude Code) — an expensive model plans, a cheap one executes, split by
  *phase* in one conversation. costrel splits by claim, potentially many
  times within one turn, and the cheap model stays the orchestrator
  throughout.
- **[Anthropic's multi-agent Research architecture](https://www.anthropic.com/engineering/built-multi-agent-research-system)**
  — closest structural cousin, inverted: the *expensive* model orchestrates
  and dispatches *cheap* subagents to search in parallel, solving a breadth
  problem. costrel runs the roles the other way, solving a stakes problem
  instead.
- **[RouteLLM](https://arxiv.org/abs/2406.18665) /
  [FrugalGPT](https://arxiv.org/abs/2305.05176)-style cascades** — decide up
  front which single model handles the whole query, escalate on low
  confidence. No brief-prep step exists in that literature — on escalation,
  the expensive model re-reads the raw query from scratch. costrel's whole
  premise is the opposite: front-load the read cost onto the cheap tier so
  the expensive call never re-opens a file.
- **[Closed-book QA](https://arxiv.org/abs/2002.08910)** — Roberts, Raffel,
  and Shazeer's framing, and where costrel's own term comes from: a model
  answers with zero external retrieval, to measure what's stored in its
  parameters. The goal is different — their closed-book tests memorized
  world-knowledge as a benchmark; costrel's is a budget discipline, denying
  tools and files so the call spends its budget reasoning over a supplied
  brief instead of fetching.
- **[Speculative decoding](https://arxiv.org/abs/2211.17192)** — the closest
  real cousin to a cheap-drafts/expensive-verifies shape: a small model
  proposes tokens, a large one verifies them in parallel, on every inference
  step. costrel runs the same asymmetry at task granularity instead of
  token granularity, manually and rarely instead of automatically, and asks
  for a judgment rather than a check against a fixed target.
- **[LLM-as-judge](https://arxiv.org/abs/2306.05685)** — a strong model
  scores another model's already-finished output against a rubric, inside
  an automated eval pipeline. costrel's consultant is dispatched mid-task,
  before the orchestrator's next move even exists to be graded — closer to
  a second opinion than a grade.
- **[SBAR](https://www.ihi.org/library/tools/sbar-tool-situation-background-assessment-recommendation)**
  and **[ICS-201](https://training.fema.gov/emiweb/is/icsresource/assets/ics%20forms/ics%20form%20201,%20incident%20briefing%20(v3).pdf)**
  — clinical and incident-command handoff protocols, and the closest match
  to costrel's own two-shape split found anywhere, in or out of ML. SBAR is
  four fields, narrow, front-loaded for one urgent call where a wrong read
  costs a patient — costrel's stakes shape. ICS-201 briefs an incoming
  commander on the whole state of an ongoing incident so they don't
  reconstruct it from scratch — costrel's range shape. Both predate LLMs by
  decades and exist for the same reason costrel does: whoever's arriving
  shouldn't have to re-derive what the sender already knows. The
  [five-paragraph order](https://en.wikipedia.org/wiki/Five_paragraph_order)
  (SMEAC) is worth naming as the contrast rather than a third match — a
  comparably dense brief, but issued *down* a hierarchy as an order, where
  costrel's goes *up*, asking for a verdict rather than delivering one.

## Status

Design converged, three real consults, one confirmed range example, zero
stakes runs. Next real step: a stakes-shaped consult run from this repo
directly — the one shape with no real evidence behind it at all. After
that, revisit whether manual-only triggering should ever graduate to an
automatic gate. Full history is in `CHANGELOG.md`.

## License

MIT.
