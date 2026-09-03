# Changelog

Dated entries, not versioned releases — there's no binary here, just a
skill, an agent type, and the discipline behind them.

## 2026-09-03 — design converged, first pressure test, repo opened

**Naming.** Checked npm, PyPI, crates.io, and GitHub for `costrel`: clean —
no package on any registry, and the three GitHub hits (a Project Gutenberg
title and one unrelated "cost-related" substring match) claim neither the
name nor real software. `assay`, `dossier`, `flume`, `billet`, `outrider`,
`satchel` were considered and rejected as common enough English words that
unrelated software already sits on them. `shode` was a close second — clean
on every package registry — but a GitHub org already holds the exact
username. A costrel is a small portable vessel historically carried out to
someone working at a distance, filled in advance with what they'd need —
the actual mechanism this project is named for: a compact, prepared thing
carried out to sustain someone else's work.

**Trigger taxonomy.** Settled on two shapes — stakes (asymmetric or
hard-to-walk-back cost of a wrong verdict) and range (a task that doesn't
decompose, where a fan-out of cheap moves can't reconstruct what one mind
holding the whole span would catch). Manual trigger only, on purpose: firing
this on a vibe that something is "probably worth a consult" defeats the
point, and the taxonomy isn't trusted enough yet to hook automatically.

**Closed-book design.** No tool access for the consultant, by design — a
consultant call at this tier is expensive enough that budget spent fetching
instead of reasoning shows up as real cost on the bill. This
also makes brief completeness a hard requirement rather than a nice-to-have:
there's nothing to re-derive with, so the brief has to be complete by
construction.

**Dispatch mechanism, verified in three parts.** Three candidates were
compared: a non-fork `Agent()` call with a model override, a `Workflow`
script's backgrounded `agent()`, and cross-session peer messaging. The
`fork` subagent type was ruled out outright — it always inherits the
parent's own model, silently ignoring any override, so it can't be a
genuinely different, more expensive model. Of the three facts the first
pressure test flagged as unverified:
- **Async, confirmed by direct observation.** Every non-fork `Agent()` call
  made this session returned immediately and notified on completion; the
  orchestrating turn was never blocked.
- **Model selection, confirmed against real docs.** The per-invocation
  `model` parameter on the `Agent()` call — not the static `model:` field in
  the agent-type file — is what actually picks the consultant tier, and it
  wins first in the resolution order.
- **Zero tools, confirmed by direct test.** Created the `costrel-consultant`
  agent type (`tools: []`) and, once a fresh session picked it up (custom
  agent-type files are not hot-reloaded — confirmed by a failed attempt in
  the same session that created the file), spawned it and asked it to list
  every tool it had. It reported none, and correctly flagged an unrelated
  identity conflict on its own (the system prompt named one model while an
  injected attribution reminder named another) instead of silently picking
  one.

**First pressure test.** Ran the six-field brief for real against the
dispatch-mechanism question above — the nearest genuinely unresolved
decision, not a staged example — closed-book, told explicitly to flag
anything the brief didn't supply rather than guess past a gap. The
consultant recommended the mechanism above, correctly ruled out peer
messaging on cost grounds, and named three facts the brief had left
unverified instead of guessing past them. That gap is what produced the two
hard requirements now in the brief template (verify checkable State facts
before dispatch; require an `Answering as: <model>` opening line). The run
also didn't land in either the stakes or range bucket — a bounded technical
choice among enumerated candidates is a third shape with one data point
behind it (see README's Status section for what that means for the
taxonomy).

**Shipped:** the `costrel` skill, the `costrel-consultant` agent type, and
this repo.

## 2026-09-03 — first external consult, one brief-template gap found

A sibling project, `freshet`, ran a real consult against a stalled quality
metric ("close the Sonnet-weaver quality gap") and reported the outcome
back over the same agent-to-agent messaging rail this project already
watches. The consultant found a confound sitting right in the State
section — a sampling setting riding the same model knob as the metric
itself — and a second one behind it, a control-flow gate tied to that same
knob. No budget went toward re-deriving anything already on the page.

It also flagged the one thing the brief left out: whether the scoring
judge saw the final answer or the full pipeline trace, a fact that decides
between two different failure mechanisms. Added to the State checklist:
for any eval-shaped consult, state what the scorer sees, same tier as the
metric.

## 2026-09-03 — a meta-consult, a confirmed range example, a retired bucket

**The meta-consult.** Dispatched a real costrel brief to Fable asking what
should change in costrel's own design. Before writing it, fetched
Anthropic's current prompting guides for this model family directly
(`Prompting Claude Fable 5` and `Prompting Claude Fable 5.1`) rather than
reasoning from general priors about the model being dispatched to — since
those guides state facts about the exact thing the brief needed to get
right.

The answer's central finding: `costrel-consultant.md`'s three sentences
were never the whole prompt the consultant runs on. Every subagent also
inherits the parent session's global CLAUDE.md and this host's plugin
injections, several of which are direct
instructions — call a status tool, run a git audit before claiming
something doesn't exist — that a zero-tool agent has no way to satisfy.
Checked against Claude Code's own documentation afterward rather than
taken on trust: CLAUDE.md inheritance is documented with no agent-type
field to disable it, and the `Agent()` tool has no effort-tier parameter
at all. Closed-book turned out to mean "no tools," not "no inherited
instructions" — a narrower guarantee than the original design implied.

Rewrote `costrel-consultant.md` around that finding: lead with the
verdict, attach a knowledge gap to the specific claim it blocks rather
than a preamble, label recalled facts as memory instead of observation,
and state outright that inherited instructions assuming tools or a coding
task don't apply. That last line is doing real work — the platform offers
no structural way to isolate a subagent from the rest of its context, so
the instruction is the only enforcement available. Also added two
brief-template requirements: label State bullets verified, inferred, or
recalled; paste a consult's brief verbatim when reporting it back rather
than summarizing it.

Considered and declined: calling the Anthropic API directly instead of
`Agent()`, to get access to an explicit effort tier. No real consult so
far has failed on reasoning depth — the failures were a missing fact, an
unlabeled recollection, and this session's own overconstraint finding —
so the fix belongs in the prompt, not in a rebuilt dispatch path that
would also give up async delivery and the SHA-driven update model.

**A confirmed range example.** Asked `freshet` for the verbatim brief
behind their consult rather than trusting the earlier summary. It holds
up: a ten-stage pipeline that went from 10 wins to 2 losses against a
frozen baseline when only the synthesis stage's model tier changed, with
the open question spanning the whole pipeline rather than one claim. The
consultant's key move — noticing that two separate stages downstream of
the swap shared the same model setting as the thing being measured — was
an implication the brief's own State section supported but hadn't named.
That's the first real range example that ran through the actual template,
rather than predating it.

**A retired bucket.** The first pressure test's run — picking a dispatch
mechanism — was recorded as a third, unconfirmed trigger shape. Rereading
it during the meta-consult changed that: the run was fully checkable with
a handful of direct tests immediately afterward, which by the taxonomy's
own definition of range (a fan-out of cheap moves can't reconstruct what
one mind would catch) means it never needed a costrel dispatch in the
first place. The taxonomy correctly flagged a decomposable question; it
wasn't evidence of a shape the two-bucket split was missing. `SKILL.md`
and `README.md` no longer carry the third-bucket language.
