---
description: Plan and, with explicit approval, run a chaos experiment from a FaultScout failure scenario against the local Docker Compose environment
argument-hint: "[scenario-number]"
---

Turn a FaultScout failure scenario into a runtime experiment against the
current repository's local Docker Compose environment, and execute it only
after the user explicitly approves the plan.

Load and follow the `chaos-experiment` skill for the full rules. This command
uses the terminal directly (Docker Compose, `docker`, `redis-cli`, `curl`) —
no MCP server, no backend, no daemon. It never modifies application source
code, Dockerfiles, Compose files, or configuration, and it never targets
anything other than the local Docker Compose environment.

## Usage

```text
/faultscout:experiment
/faultscout:experiment 3
```

Arguments passed to this command: `$ARGUMENTS`

- A bare number (e.g. `3`): use failure scenario 3 from the most recent
  `/faultscout:chaos` report discussed in this conversation.
- No arguments: identify the failure scenarios available from the most
  recent `/faultscout:chaos` report in this conversation, list them
  concisely (number, title, failure type, one-line hypothesis), and ask the
  user which one to experiment with. Do not guess which one they want.
- If no `/faultscout:chaos` report exists yet in the conversation, say so
  and ask whether to run `/faultscout:chaos` first or to describe the
  target/failure mode directly.

## Steps

1. **Identify the scenario.** Resolve it from the argument or the user's
   answer to the list above. If the referenced scenario number does not
   exist, say so and show the available list again.
2. **Check the scenario has enough evidence** to build a safe plan (a clear
   target, a clear failure mechanism mapping to one of the three supported
   failure modes). If not, say `The selected scenario does not contain
   enough evidence to execute a safe runtime experiment.` and ask for
   clarification or stop.
3. **Inspect the environment**: locate the Compose configuration, determine
   the repository's documented startup command, check whether it is already
   running, and confirm the target service exists and is local Docker
   Compose (never remote/staging/production/Kubernetes).
4. **Build and print the experiment plan** (Experiment, Hypothesis, Target,
   Failure mode, Preconditions, Expected behavior, Expected failure signal,
   Observation window, Restore procedure, Risk/blast radius), following the
   `chaos-experiment` skill's field-by-field guidance. Do not execute
   anything yet.
5. **Ask the user to explicitly approve this exact plan.** Selecting a
   scenario number in step 1 is not approval to execute — treat it only as
   selecting what to plan. Do not proceed past this point without an
   unambiguous yes.
6. **Run the safety checklist** from the `chaos-experiment` skill. If any
   item fails, stop and explain which one, without injecting anything.
7. **Execute**: snapshot → inject → observe for the plan's observation
   window → restore → verify restoration, following the exact procedure for
   the plan's failure mode (`container.pause`,
   `container.network_disconnect`, or `redis.poison_key`).
8. **Report the result** using the `chaos-experiment` skill's report format,
   with a `Result` of exactly one of `CONFIRMED`, `PARTIALLY_CONFIRMED`,
   `NOT_REPRODUCED`, `INCONCLUSIVE`, or `ABORTED`. If restoration failed,
   report that plainly and give the exact manual command needed to finish
   restoring the environment.

## Report language

Same behavior as `/faultscout:chaos`: default `english`; `turkish` is also
supported if the user asks for the report in Turkish. Code identifiers,
service/container/network names, Redis keys, file paths, command names, and
log content are never translated.

## Rules that always apply

- Never treat scenario selection as authorization to execute. Execution
  requires a separate, explicit approval of the printed plan.
- Never use a failure mode other than `container.pause`,
  `container.network_disconnect`, or `redis.poison_key`.
- Never run `docker compose down`, `docker system prune`, `docker rm`,
  `docker kill`, `docker stop`, `docker volume rm`, or `docker network rm`.
- Never modify application source code, Dockerfiles, Compose files, or
  configuration to make an experiment work; report that it cannot be run
  instead.
- Never target Kubernetes, staging, production, cloud infrastructure, or a
  remote Docker daemon.
- Every experiment that is injected must reach a restore attempt, and the
  restored state must be verified, not assumed.
