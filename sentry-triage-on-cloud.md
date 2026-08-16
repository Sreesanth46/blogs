---
title: I Let the Triage Agent Run Unattended for a Week. It Broke.
date: 2026-08-17 00:50:11 +0530
tags:
  - sentry
  - bug-fix
  - agent
  - ai
  - fix
  - wip
---
Every automated pipeline eventually meets the environment it wasn't tested against.

Part one covered the workflow: rank by impact, dedupe, file a ticket that explains itself, explain the root cause before touching code, fix only the narrow stuff, and stop on everything else.

That version ran with me watching. I saw every step.

The natural next move was to stop watching — put it on a schedule, let it clear the backlog on its own, and come back to a stack of explained tickets and draft PRs.

That's what I did.

Then it broke.

Not catastrophically. Nothing got merged that shouldn't have. The safety boundaries from part one held.

The problem was more mundane.

**A lot of scheduled runs failed, and the tickets got stuck in** `In Progress`

The cron job kept running. More tickets entered the pipeline. Failed runs didn't always clean up after themselves. The result was a growing backlog of tickets that looked active but weren't actually being worked on.

Eventually, I had to turn off the cron job, go through the stuck tickets manually, figure out why each run had stopped, fix the underlying problems, and clear the backlog before turning the automation back on.

The triage agent needed triage.

The interesting part was that the diagnoses mostly held up. The reasoning was sound.

What broke wasn't the judgment.

It was everything that judgment depended on being true.

## 1. The agent couldn't reach private packages

The first real blocker wasn't the code.

It was the network.

Some of the repositories the project depended on used private npm packages. The agent could see the imports and understand what the code was trying to do, but when it tried to install dependencies or inspect the package, the request was blocked by Claude's network restrictions.

From the agent's perspective, the package simply wasn't reachable.

That created an interesting failure mode.

The diagnosis could be completely correct, but the agent couldn't get far enough to verify it or run the tests. An unattended run can't exactly ask someone to approve a URL in the middle of a scheduled job.

So the agent stopped.

That was the right behavior, but the scheduled task had no way to resolve the problem by itself. The ticket stayed in `In Progress`, and the next scheduled run moved on to other work.

The fix was simple once I found the problem: I manually added the package registry URL to the allowed network destinations.

After that, the same workflow could proceed normally.

What caught me off guard was that I hadn't considered network access part of the agent's operating environment.

When I was running the workflow manually, I already knew the project had private dependencies. If something failed during installation, I'd recognize the problem immediately and unblock it.

Once the agent was running unattended, **network access became another dependency that had to be made explicit**.

The lesson wasn't "give the agent unrestricted network access."

It was the opposite.

Define the network boundary deliberately, and make sure the agent has access to the resources required for the work it's actually allowed to perform.

A scheduled agent doesn't just need code, credentials, and a repository.

It needs an environment capable of reproducing the work a human would have done interactively.

## 2. A stack trace tells you where an error surfaced, not necessarily where it went unhandled

Part one already covered the limits of root-cause tools and the fallback of reading the stack trace and source manually.

The problem that showed up at scale was more subtle.

With minified traces, especially for unhandled promise rejections, the stack can point into a shared bundle — an interceptor, a vendor SDK, or some other common layer.

That's often where the `Error` object was constructed.

It isn't necessarily where the promise was left unhandled.

For one ticket, a person can look at that trace, recognize the dead end, and go somewhere else.

For many tickets running unattended, that ambiguity creates two possible outcomes.

The bad one is a confident but incorrect diagnosis.

The good one is an honest:

> Can't pin this down from the available evidence.

Mine mostly did the second.

That's the design working.

The agent stopped instead of inventing an answer.

But it exposed another problem: **a backlog of correctly deferred work still grows when nobody is there to pick it up.**

Failing closed prevents bad changes.

It doesn't make the unresolved work disappear.

And when the workflow is scheduled, every stopped run can leave another ticket sitting in `In Progress`.

Someone still has to clear that queue.

## 3. Ticket-level dedup misses root-cause-level duplicates

The original deduplication logic asked:

> Does a ticket already exist for this error?

That's useful, but it isn't enough.

I eventually found two different error-tracking issues with two different stack traces that were actually the same bug underneath.

Both came from a truthiness check against the wrong object, just surfacing in two unrelated components.

Two issues.

Two tickets.

One root cause.

Checking the issue ID can't catch that.

You need something that can reason about the **diagnosis**, not just the identifier attached to the error.

I fixed the shared cause once, left a comment on the duplicate tickets pointing to the fix, and skipped the second redundant PR.

The original pipeline had no place for that step.

That was a gap in the workflow, not a failure of the model.

## 4. "Already fixed" isn't a state the ticket tracker knows about

A few tickets described bugs that were real when the error last fired but had quietly been fixed on `main` since.

The auto-generated ticket was frozen at file time.

The codebase wasn't.

That distinction matters.

Before spending time diagnosing or fixing an issue, the agent needs to ask a simpler question:

**Does this bug still exist?**

That means checking the current source before doing more work.

It's cheap per ticket.

It's also very easy to skip when the workflow is optimized around moving quickly through a backlog.

An unattended system can't assume that the state captured by one tool is still true just because the ticket says it is.

The ticket is evidence.

It isn't reality.

## The bigger problem wasn't the failures

Looking back, the individual failures weren't particularly scary.

The agent couldn't reach a private package.

A stack trace wasn't enough to establish the root cause.

Two different errors turned out to be the same underlying bug.

A ticket described something that had already been fixed.

None of these caused a bad production change.

The more interesting failure was what happened **because the system was scheduled**.

A supervised workflow has a natural feedback loop:

> Run → fail → human notices → fix → continue.

A scheduled workflow can instead become:

> Run → fail → ticket stuck → schedule runs again → more tickets → more stuck tickets.

The failure itself isn't necessarily dangerous.

**The accumulation is.**

That's why I eventually had to turn off the cron job.

Not because the agent was making unsafe changes, but because it was generating unfinished work faster than I could understand why it was unfinished.

Once the scheduler was off, I could work through the backlog, fix the environmental issues, clean up the stuck tickets, and only then turn the automation back on.

That was the part I hadn't designed for.

I had designed the agent to fail safely.

I hadn't designed the **scheduler** to fail safely.

## Why this shape breaks down unattended

None of this touched the core judgment from part one.

The rules still held:

- Only attempt narrow fixes.
- Stop when the change is ambiguous.
- Explain the root cause before acting.
- Never merge automatically.

What broke was the assumption underneath those rules:

**that the environment and the evidence could be trusted at face value, and that a failed run would remain an isolated failure.**

A supervised agent has a person acting as an error-correction layer.

Someone sees a dependency installation fail.

Someone reads a suspicious stack trace and says, "This isn't actually the source of the problem."

Someone realizes that two apparently different errors are actually the same bug.

Someone notices that the ticket describes something that's already been fixed.

And importantly, someone notices when the system itself starts producing more unfinished work than it can handle.

Those checks feel almost free when a human is sitting there.

Take the human away and they don't disappear.

They become requirements.

That's the part I underestimated.

I thought I was removing a spectator.

I was actually removing the cheapest verification layer the system had.

## The takeaway

Unattended doesn't mean hands-off.

It means every check a human used to perform almost unconsciously now has to become explicit.

- Verify that the environment can actually reach the dependencies the project needs.
- Treat uncertainty as a valid outcome instead of forcing a diagnosis.
- Deduplicate on root cause, not just issue ID.
- Revalidate that the bug still exists before spending effort fixing it.
- Make failed runs visible and prevent the scheduler from endlessly accumulating stuck work.
- Have a way to pause the system, clear the backlog, and resume safely.

The first version of the system was designed around a simple question:

**Can the agent safely fix this?**

The unattended version needs another question before that:

**Can the agent trust the environment and evidence it is working with — and what happens when it can't?**

That's a harder problem than I expected.

Automate the reconnaissance. Automate the obvious fixes. Stop on real judgment calls.

But when nobody is watching in real time, rebuild — as explicit checks, failure states, and recovery paths — the verification that a live session used to give you for free.

<p style="text-align: center;">fin.</p>