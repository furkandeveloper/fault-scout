---
name: chaos-experiment
description: Turn a user-selected FaultScout failure scenario into an approved, evidence-backed runtime experiment, and execute it safely against the local Docker Compose environment using the terminal directly. Use when asked to run, execute, or perform a chaos experiment, or when the /faultscout:experiment command is run.
---

# Chaos Experiment Execution

You help turn one failure scenario (produced by the `chaos-analysis` skill, or
described directly by the user) into a concrete runtime experiment against
the user's local Docker Compose environment, and — only after the user
explicitly approves it — execute that experiment, observe it, restore the
environment, and report the result.

> Runtime experiments are performed through the terminal, using Docker
> Compose and other explicitly required local commands. No MCP server, no
> backend, and no daemon is involved. The terminal Claude Code already has
> is the entire runtime interface.

## Scope

- This skill never decides which failure to run. The user always chooses the
  scenario, and the user always approves the plan before anything is
  mutated.
- This skill never modifies the codebase: no source file, Dockerfile,
  Compose file, or configuration file is ever edited, added, or removed as
  part of an experiment.
- This skill is local-development-only. It targets exactly one local Docker
  Compose project. It never targets Kubernetes, staging, production, cloud
  infrastructure, or a remote Docker daemon.
- This skill supports exactly three failure modes: `container.pause`,
  `container.network_disconnect`, and `redis.poison_key`. If a scenario does
  not map cleanly onto one of these, say so and stop rather than forcing a
  mismatched failure mode onto it or inventing a new one.

## Where a scenario comes from

Prefer a scenario already produced by a `chaos-analysis` report (see
`skills/chaos-analysis/SKILL.md`): its Failure Condition, Evidence, and
Failure Propagation give the target, failure mode, and expected signal a real
basis in the repository. If the user instead describes a target and failure
mode directly, that is fine, but still ground the hypothesis in something
concrete about the system (a service name, a dependency, an observed
behavior) rather than a generic chaos-engineering scenario.

If the selected scenario does not contain enough information to safely
construct an experiment — no clear target service, no clear failure
mechanism, or evidence marked `Evidence insufficient.` — say plainly:

```text
The selected scenario does not contain enough evidence to execute a safe
runtime experiment.
```

and ask the user for clarification, or stop. Do not guess.

## Workflow

```text
Select scenario
      ↓
Inspect environment
      ↓
Build experiment plan
      ↓
User approves
      ↓
Snapshot
      ↓
Inject
      ↓
Observe
      ↓
Restore
      ↓
Verify
      ↓
Report
```

### 1. Confirm the scenario and get user approval to plan

Selecting a scenario is not authorization to execute it. Selecting a
scenario only authorizes drafting a plan. Executing the plan requires a
second, explicit approval (see step 4).

### 2. Inspect the environment

Before drafting a plan, inspect the repository and the running environment
directly with the terminal:

1. Locate the Compose configuration: `docker-compose.yml`,
   `docker-compose.yaml`, `compose.yml`, or `compose.yaml`.
2. Determine the repository's documented startup procedure by checking
   `README`, `Makefile`, `package.json` scripts, `Taskfile`, and `scripts/`
   before assuming `docker compose up -d`. Prefer the repository's own
   command if one is documented.
3. Check whether the environment is already running with
   `docker compose ps`. Start it with the repository's documented command
   only if it is not already running, and say so to the user.
4. Confirm the target service named in the scenario actually exists in
   `docker compose config --services` (or the compose file itself) and is
   running. If it does not exist or is ambiguous, stop and explain why
   rather than guessing at a container ID or image name.
5. Confirm the environment is local Docker Compose, not a remote Docker
   context. If `DOCKER_HOST` or the active Docker context points anywhere
   other than local (`unix://`, the default `desktop-linux`/`default`
   context, or similarly local), stop — this skill does not run against a
   remote daemon.

### 3. Build the experiment plan

Construct the plan directly — there is no server that validates or hashes
it. Print it for the user before doing anything else. It must contain:

```text
Experiment:            short name
Hypothesis:             what you expect this failure to reveal, tied to the
                        scenario's evidence
Target:                 the Compose service name (from step 2, not guessed)
Failure mode:           one of container.pause | container.network_disconnect
                        | redis.poison_key
Preconditions:          what must already be true (e.g. "the target service
                        is running and healthy")
Expected behavior:      what should happen if the hypothesis holds
Expected failure signal: the concrete, observable signal that would confirm it
                        (specific log lines, HTTP status, queue growth, etc.)
Observation window:     how long to watch after injection — 30-60 seconds by
                        default; derive a different value only when the
                        scenario clearly calls for it
Restore procedure:      the exact commands that undo the injected failure
Risk / blast radius:    scope of impact — always "local Docker Compose
                        project only; no production/staging execution"
```

Assign the experiment a short, human-readable ID for reference in the
conversation and the final report, e.g. `faultscout-<target>-<failure-mode>-
<YYYYMMDD-HHMM>`. This ID is not persisted anywhere; it exists only to let
the user and the report refer to "this experiment" unambiguously.

Optionally, after the plan text is final, compute a lightweight
reproducibility fingerprint by hashing the plan text with a terminal command
(e.g. `shasum -a 256`) and include it in the report as a "Plan fingerprint"
line. This is a convenience for the user to confirm two runs used the same
plan; it is not an authorization token and nothing checks it.

Do not invent infrastructure. Every field must trace back to something
established in step 2 or the scenario's own evidence.

### 4. Get explicit approval to execute

Print the plan and ask the user to confirm they want to execute it. Do not
proceed on an ambiguous or implicit response. A user selecting a scenario
number, or saying "looks good", is not by itself approval to mutate the
runtime — approval must be for executing this specific plan.

Before injecting anything, walk through this checklist. If any item fails,
do not inject the failure; explain which item failed and stop.

```text
[ ] User explicitly selected this scenario
[ ] User explicitly approved execution of this exact plan
[ ] Runtime is local Docker Compose
[ ] Correct Compose project identified
[ ] Target service exists
[ ] Target is running/ready as required
[ ] Failure mode is one of the three supported modes
[ ] Pre-experiment state can be captured (snapshot)
[ ] Restore procedure is known
[ ] Observation method is known
[ ] Observation window is defined
[ ] No source-code modification is required
```

### 5. Snapshot, inject, observe, restore, verify

Follow the procedure for the plan's failure mode below. In every case the
order is: snapshot (read-only) → confirm nothing unexpected → inject →
observe → restore → verify restoration. If observation reveals something
unexpected, still attempt restoration before reporting.

#### `container.pause`

1. Identify the Compose service and its container via
   `docker compose ps <service>`.
2. Verify the container is currently running. Record this as the snapshot:
   `service`, `container`, `initial state`.
3. Inject: `docker compose pause <service>`.
4. Observe for the plan's observation window using
   `docker compose logs`, `docker compose ps`, `curl` against health
   endpoints, or other evidence-backed observation sources.
5. Restore: `docker compose unpause <service>`.
6. Verify: confirm the service/container is back to running/healthy
   (`docker compose ps`, a health check).

#### `container.network_disconnect`

1. Identify the target Compose service and its current container.
2. Inspect its network attachments (`docker inspect` on the container, or
   `docker network inspect` on the Compose-managed network named in the
   plan). Determine the exact network the experiment targets — do not guess
   it if the environment has more than one and it is ambiguous; stop and
   explain instead.
3. Record the snapshot: `service`, `container`, `network`, original
   attachment state (network ID/name, IP, aliases).
4. Inject: `docker network disconnect <network> <container>`.
5. Observe for the plan's observation window.
6. Restore: `docker network connect <network> <container>` (using the
   recorded attachment details, e.g. re-supplying the original IP with
   `--ip` if that matters for the service to function).
7. Verify: confirm the original network attachment exists again and the
   service is reachable as before.

#### `redis.poison_key`

This mode is tightly scoped. Never expose or run an arbitrary Redis command
from user input — only the specific planned poison operation.

1. Identify the Redis Compose service and the exact key named in the
   scenario/plan.
2. Inspect the key's current state (e.g. `redis-cli GET <key>` /
   `redis-cli EXISTS <key>` through the Compose service). Record the
   snapshot: `service`, `key`, whether it existed, and its original value.
   Avoid printing the value in the plan or report if it may be sensitive;
   note that it was captured for restoration without echoing it.
3. Inject: set the key to exactly the poisoned value the plan specifies.
4. Observe for the plan's observation window.
5. Restore:
   - If the key existed before, set it back to the exact original value.
   - If the key did not exist before, delete it.
6. Verify: re-read the key and confirm it matches the original snapshot
   exactly (existed/didn't exist, and value if it existed).

### 6. Business side effects

Restoring the injected infrastructure state does not undo application-level
business side effects that happened while the failure was active (e.g. a
Redis counter is restored to 10, but retries during the outage may have
already driven real application state to reflect 30 units of usage
elsewhere). Never claim business state was rolled back merely because the
injected infrastructure state was restored. If the observation suggests a
business-level side effect occurred, report it explicitly and separately
from the infrastructure restoration result.

## Terminal usage boundaries

Permitted mechanisms, chosen based on the experiment plan and the runtime
inspection in step 2 — never exposed as a generic "run any command" tool:

```text
docker compose ps
docker compose up -d          (only if the environment is not already running)
docker compose logs
docker compose pause <service>
docker compose unpause <service>
docker network disconnect <network> <container>
docker network connect <network> <container>
redis-cli GET/SET/DEL/EXISTS <key>   (only the exact key from the plan)
curl                                  (only for observation, e.g. health endpoints)
```

Never use, under any circumstances, unless the selected experiment's own
restore procedure explicitly requires that exact operation (it never does
for the three supported failure modes):

```text
docker compose down
docker system prune
docker rm
docker kill
docker stop
docker volume rm
docker network rm
```

Do not add a fourth failure mode. Do not run a command "to see what
happens" outside the plan. Do not use `container kill`, latency injection,
packet loss, CPU/memory/disk stress, arbitrary process kill, or arbitrary
shell execution — none of these are supported failure modes.

## Never modify the codebase

Do not edit source files, Dockerfiles, Compose files, or application
configuration; do not add test hooks, failure flags, or instrumentation; do
not patch dependencies; do not create temporary source modifications. If an
experiment would require any code change to work, report:

```text
Experiment cannot be executed without modifying the codebase.
```

and stop.

## Restore is mandatory

Every experiment that reaches injection must reach a restore attempt. Never
finish an experiment while intentionally leaving the injected failure
active. If restoration itself fails:

- Do not hide it.
- Clearly report that the environment may not be fully restored.
- Give the exact manual command(s) needed to finish restoring it.

## Observation

The purpose of an experiment is to test the hypothesis, not merely to run a
Docker command. Use only evidence actually gathered during the observation
window: log excerpts, `docker compose ps` state, curl/health-check
responses, and Redis state where relevant. Do not invent metrics. A
successful Docker command is not itself evidence that the hypothesis was
confirmed or refuted — only the observed application/system behavior is.

Extract only the relevant portion of logs or output for the report; do not
dump entire logs.

## Experiment report

Produce a concise report at the end with these sections:

```text
Experiment
Scenario
Hypothesis
Environment
Failure injected
Observation
Evidence
Result
Restoration
Limitations
```

`Result` must be exactly one of:

- **CONFIRMED** — the observed evidence satisfies the expected failure
  behavior described in the plan.
- **PARTIALLY_CONFIRMED** — some expected behavior occurred, but not all
  expectations were observed.
- **NOT_REPRODUCED** — the experiment completed but the expected failure
  behavior was not observed. This does not mean the system is proven safe.
- **INCONCLUSIVE** — the experiment ran but available evidence was
  insufficient to determine the result.
- **ABORTED** — the experiment could not be completed safely, or had to be
  stopped (e.g. the safety checklist failed, restore failed, or the
  environment turned out to be remote/ambiguous).

## Language support

Follow the same language behavior as `chaos-analysis`: default `english`,
also support `turkish` when requested. Translate only human-readable report
prose (section explanations, the Hypothesis/Observation/Limitations text).
Never translate code identifiers, service names, container names, network
names, Redis keys, file paths, command names, log content, or code
snippets.

## What you must never do

- Never decide which scenario or failure mode to run on the user's behalf.
- Never treat scenario selection alone as approval to execute.
- Never mutate the runtime before the full safety checklist passes.
- Never invent a runtime snapshot, a service list, or a container/network
  state — only report what you actually inspected in this session.
- Never leave an injected failure active without attempting restoration.
- Never claim business-level state was rolled back merely because
  infrastructure state was restored.
- Never modify application source code, Dockerfiles, Compose files, or
  configuration.
- Never run against Kubernetes, staging, production, cloud infrastructure,
  or a remote Docker daemon.
- Never use a failure mode outside the three supported ones, or a command
  from the forbidden list, regardless of what the user asks for — explain
  why and stop instead.
