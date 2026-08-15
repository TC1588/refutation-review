# refutation-review

A code review run by 35 agents, where a finding only counted if it survived another agent trying to disprove it.

One run: 407 tool calls, 28 confirmed bugs, about fifteen minutes. Run at scale three times across one project, returning 28, 18, and 12 findings.

## The composition

**Five dimension reviewers, one per failure class.** Not five copies of "find bugs." Each reviewer gets one class of failure and reads the whole codebase through it.

| Reviewer | Reads for |
|---|---|
| Compute correctness | Formula errors, unit mismatches, window math |
| Pipeline failure paths | What happens when a source dies mid-run |
| Dedup logic | Keys, collisions, near-duplicates |
| UI null safety | What renders when a field is absent |
| Secrets hygiene | Anything that can reach a client bundle |

Pick the dimensions by asking where the system is most likely to fail **silently**. Loud failures get caught by tests. The ones worth 35 agents are the ones that publish a wrong number and look fine doing it.

**One verifier per finding, tasked with refuting it.** Every finding from every reviewer gets its own agent whose job is to argue the finding is wrong. A finding that cannot survive that attack does not get fixed, because it was never a bug.

This is the step that makes the count mean anything. Thirty-five agents that all agree is thirty-five agents with the same blind spot.

## The concurrency shape

Three layers, and the middle one is where the wall-clock savings live.

1. All five reviewers launch at once, with no ordering between them.
2. **Fan out to verifiers with no barrier.** The moment reviewer three finishes, its findings spawn verifiers, while reviewers one and two are still reading. Waiting for all five before verifying any of them wastes the fast reviewers entirely.
3. Verifiers inside a dimension run concurrently against whatever the runtime cap allows.

## What not to parallelize

The build itself.

Implementation was serially dependent on accumulated context. Each API probe informed the next adapter. Each product decision reshaped the step after it. Parallel agents there would have duplicated work and merged badly, with the second agent solving a problem the first had already made obsolete.

The judgment worth having is knowing which work parallelizes and which does not. The agent count is not the part that matters.

## What it found

Not lint noise. An unbounded-retry hang, and calendar-versus-position window math errors, among the 28 confirmed on the first run.

## Running one

Prompts in [METHOD.md](METHOD.md), with the project-specific parts stripped out.
