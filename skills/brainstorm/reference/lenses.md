# Thinking lenses

Use these during the **diverge** step. **Rotate** lenses: one distinct lens per proposed direction, not the same one five times. Each block below is written as something a sharp professor might say, plus one concrete follow-up you can ask.

## First principles

Strip the problem down: what is irreducible? What are the true constraints (physics, math, economics, incentives) vs labels we invented? The goal is to see whether the difficulty is in the world or in our phrasing.

**Example prompt:** “If we had to bet the department budget on a single fact about this system, which fact is least negotiable, and what follows from that alone?”

## Inversion

Start from what would *guarantee* failure, then work backwards. Often the list of “must not happen” is shorter and more falsifiable than the list of “nice to have”.

**Example prompt:** “Suppose this project is a well-known type of bad outcome in six months—what is the one mistake that most predictably got us there?”

## Analogy

Who, in another field, faced a structurally similar setup (uncertainty, stakeholders, feedback loops, constraints)? The analogy is for structure, not branding—name what matches and what breaks when you import it.

**Example prompt:** “This reminds me of __ (different domain). The usual lesson there is __. Where does the analogy break for us?”

## Constraint flip

Change a resource or boundary: one day, no money, 100× the data, forced simplicity, or an adversary who can pick the worst case. Sometimes the “real” plan hides in the under-resourced version.

**Example prompt:** “If you had 24 hours and $0, what is the most informative thing you could still do—and would you trust its sign?”

## Stakeholder shift

Look from the user, the adversary, the operator, the regulator, or a future maintainer. Conflicts and edge cases show up at boundaries between incentives, not in the happy path.

**Example prompt:** “If you were the person with the most incentive to make this go wrong, what is your first move—and how would we notice it in data or behavior?”

## Time-horizon shift

Separate what must be true in a week (probe, learn, de-risk) from what must be true in a year (sustain, scale, compound). A direction can be a great short-horizon bet and a bad long-horizon bet or vice versa.

**Example prompt:** “What is the 1-week decision, and what is the 1-year invariant—do we need the same bet for both?”

## Scale shift

Ask what breaks at 10× (load, people, money, error rate) and what becomes trivial at 0.1×. Many “open” problems are scale confusion in disguise.

**Example prompt:** “At 10× our current [throughput/team/constraint], which part of the plan becomes the first bottleneck, and is that the kind of bottleneck you want to own?”

## Simplest thing that could work (STCTW)

What is the most embarrassing, minimal, end-to-end version that still teaches you something? Not the MVP slide—the experiment that is almost too small to admit in public, but that falsifies a key claim.

**Example prompt:** “If we weren’t allowed to build anything we’d be proud of on GitHub, what is the smallest end-to-end slice that would still let us be wrong in an interesting way?”

## Prior art / standard objection

Name the “textbook” reaction. Good ideas are often a variant of a known pattern; the work is in seeing why this time is *not* a dumb repeat.

**Example prompt:** “A referee’s first line would be: ‘This is the same as __, which fails when __.’ What is our answer that isn’t just optimism?”

## Pre-mortem

It’s six months out, and this failed. Write the obituary: the proximate cause, the ignored warning sign, the cheap check we didn’t run. Do not be polite.

**Example prompt:** “In the first paragraph of the post-mortem, what is the unforced error, and which Tuesday meeting could we have used to see it early?”
