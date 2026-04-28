# The trident workflow

This is the working method I use to build Atenea. It splits a single development task across three roles: a model that diagnoses and designs, a model that implements, and a human who tests against reality. Each role has different failure modes, so keeping them separate produces fewer regressions than asking any one of them to do all three.

## The three roles

**1. Diagnosis and design — Claude.**
Reads the relevant code, identifies the gap, and writes a concrete plan: which files to touch, what each change should do, what edge cases the implementation has to handle, and what could break. The output is a specification, not code.

**2. Implementation — Opus.**
Takes the specification and writes the code. No re-architecting, no scope expansion. If the spec is wrong, it surfaces the contradiction rather than papering over it.

**3. Testing — me.**
Runs the agent against real targets. Reads logs. Reports back what actually happened — not what should have happened. Failures here feed the next diagnosis cycle.

## Why split it this way

The split exists because the three jobs benefit from different things:

- Diagnosis benefits from breadth — reading more code, considering more failure modes, being explicit about assumptions. It is fine for it to be slow and verbose.
- Implementation benefits from constraint — a tight spec keeps the change minimal and reviewable. Letting the implementer also re-design tends to grow the diff.
- Testing benefits from being adversarial and stateful in a way a model is not. Real targets behave in ways logs never fully capture; a human notices "this finished too fast" or "this output looks wrong" before any assertion fires.

When the same actor does all three, the most common failure I have seen is the implementer quietly redefining the problem to match what was easy to write. The split makes that visible.

## Worked example: closing the Atenea pentest phase (April 2026)

The agent could already chain recon, exploitation, and post-exploitation on most boxes, but a class of Linux targets was failing: those requiring Tomcat WAR deployment followed by a cron-based privilege escalation. End-to-end runs stalled at the foothold step.

**Diagnosis (Claude).** Identified three forced actions that were missing from the planner's vocabulary: `tomcat_manager_auth`, `tomcat_war_deploy`, and `cron_privesc`. Pointed at the two files that needed to change (`agente_executor.py`, `agente_planner.py`). Flagged the non-obvious problem: the Windows-WSL boundary in the listener — the existing `msfconsole` invocation did not survive across the boundary cleanly, and the manager response parsing assumed HTML where some Tomcat versions return plain text. Specified a persistent listener with cookie reuse and a text/HTML fallback in the response parser.

**Implementation (Opus).** Added the three actions in `agente_executor.py` and the corresponding planning hooks in `agente_planner.py`. Implemented the persistent `msfconsole` listener, cookie handling, and the text/HTML fallback as specified. No changes outside the listed files.

**Testing (me).** Ran the agent against Thompson on TryHackMe. It reached root in 9 steps and 39 seconds on the first attempt, no manual intervention.

**Result.** 19 rooms validated end-to-end, 96% coverage of the target set for the phase.

A few things from this run are worth naming, because they are the reason the split paid off rather than being overhead:

- The Windows-WSL listener problem was not visible from the failing logs alone. It took reading the surrounding code to spot it. That is exactly the kind of thing the diagnosis step is for.
- The text/HTML fallback was a single line in the spec but would have been easy to skip if the same actor had been writing both spec and code under time pressure.
- The 39-second first-attempt run was the signal that the spec had been complete. A working-but-flaky run would have meant going back to diagnosis, not patching the implementation.

## When this is overkill

For trivial changes — renames, single-line fixes, obvious bugs — running the full tridente is friction without payoff. I use it when the change touches more than one file, or when the failure mode is unclear from the symptoms alone.
